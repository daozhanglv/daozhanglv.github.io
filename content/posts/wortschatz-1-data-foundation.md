---
title: 'WortSchatz (1): building the data foundation with Spring Boot and PostgreSQL'
date: 2026-11-24T12:20:40+08:00
draft: false
description: 'Setting up the Spring Boot 4 backend, PostgreSQL schema, input validation, Testcontainers integration test, and GitHub Actions CI for my German vocabulary trainer.'
tags: ['Java', 'Spring Boot', 'PostgreSQL', 'Projects']
---
[WortSchatz](https://github.com/daozhanglv/wortschatz) is a German vocabulary trainer I am building for my own studies. The goal is to paste a German article I actually read, let an LLM extract candidate words, track which ones I already know, and review the rest with spaced repetition.

I am building it step by step in small commits, using AI tools as a tutor and reviewer while making the design decisions and writing the final code myself. This post covers the first milestone in [ROADMAP.md](https://github.com/daozhanglv/wortschatz/blob/main/ROADMAP.md): the data foundation.

## The problem

Before calling an LLM or writing a review scheduler, I needed a backend that can store German words and source articles reliably, reject bad input with clear errors, and run automated tests against a real database on every push.

German vocabulary also has a few domain details worth modeling from the start: nouns carry a grammatical gender (`DER`, `DIE`, `DAS`) and a plural form, while verbs and adjectives do not, and a word is much easier to remember together with the sentence where I first saw it.

## What I built

- A Java 21 / Spring Boot 4.1.2 service using Maven Wrapper (`./mvnw`) and PostgreSQL 17 running locally via [`compose.yaml`](https://github.com/daozhanglv/wortschatz/blob/main/compose.yaml).
- Two JPA entities and repositories: [`Word`](https://github.com/daozhanglv/wortschatz/blob/main/src/main/java/io/github/daozhanglv/wortschatz/word/Word.java) and [`Article`](https://github.com/daozhanglv/wortschatz/blob/main/src/main/java/io/github/daozhanglv/wortschatz/article/Article.java), where a word can reference its source article and example sentence.
- REST endpoints to create and list words (`/api/words`) and articles (`/api/articles`), structured as controller -> service -> repository with request and response DTOs.
- Bean Validation on request records and a [`GlobalExceptionHandler`](https://github.com/daozhanglv/wortschatz/blob/main/src/main/java/io/github/daozhanglv/wortschatz/common/GlobalExceptionHandler.java) returning Spring `ProblemDetail` responses.
- A PostgreSQL integration test using Testcontainers ([`WortschatzApplicationTests`](https://github.com/daozhanglv/wortschatz/blob/main/src/test/java/io/github/daozhanglv/wortschatz/WortschatzApplicationTests.java)) and a GitHub Actions workflow ([`ci.yml`](https://github.com/daozhanglv/wortschatz/blob/main/.github/workflows/ci.yml)) that runs `./mvnw test` on every push.

## Design decisions

### Records as API boundaries, entities for persistence

Controllers never accept or return JPA entities directly. Requests and responses are Java `record` types such as [`CreateWordRequest`](https://github.com/daozhanglv/wortschatz/blob/main/src/main/java/io/github/daozhanglv/wortschatz/word/CreateWordRequest.java) and [`WordResponse`](https://github.com/daozhanglv/wortschatz/blob/main/src/main/java/io/github/daozhanglv/wortschatz/word/WordResponse.java).

This keeps the HTTP contract decoupled from the database schema and avoids accidentally serializing lazy JPA relationships: `Word.article` is mapped as `@ManyToOne(fetch = FetchType.LAZY)`, and `WordResponse.from(word)` exposes only `articleId`.

### Storing enums by name, not ordinal

In [`Word.java`](https://github.com/daozhanglv/wortschatz/blob/main/src/main/java/io/github/daozhanglv/wortschatz/word/Word.java), both `partOfSpeech` and `gender` use `@Enumerated(EnumType.STRING)`:

```java
@Column(nullable = false)
@Enumerated(EnumType.STRING)
private PartOfSpeech partOfSpeech;

@Enumerated(EnumType.STRING)
private Gender gender;
```

JPA defaults to ordinal integers, which silently corrupt existing rows if an enum constant is ever inserted or reordered. Storing the enum names makes the table readable and safe to evolve.

### Consistent error responses with `ProblemDetail`

Instead of letting Spring return inconsistent default error payloads, [`GlobalExceptionHandler`](https://github.com/daozhanglv/wortschatz/blob/main/src/main/java/io/github/daozhanglv/wortschatz/common/GlobalExceptionHandler.java) handles `MethodArgumentNotValidException` and `HttpMessageNotReadableException` with `ProblemDetail`. Validation failures return `400 Bad Request` with a field-to-message `errors` map; malformed or missing JSON bodies return a plain `400` with a clear title and detail.

### Real PostgreSQL in tests and CI early

Rather than testing against an in-memory H2 database that behaves differently from PostgreSQL, [`WortschatzApplicationTests`](https://github.com/daozhanglv/wortschatz/blob/main/src/test/java/io/github/daozhanglv/wortschatz/WortschatzApplicationTests.java) starts a real `postgres:17` container with Testcontainers and Spring Boot's `@ServiceConnection`:

```java
@Container
@ServiceConnection
private static final PostgreSQLContainer postgres = new PostgreSQLContainer("postgres:17");
```

I originally planned GitHub Actions CI for v0.2, but moved it forward right after adding this test. Having `./mvnw test` run automatically on every push is much more useful while the core schema and APIs are still changing.

## Issues encountered

- **Validation limits out of sync with database columns:** My first version of [`CreateWordRequest`](https://github.com/daozhanglv/wortschatz/blob/main/src/main/java/io/github/daozhanglv/wortschatz/word/CreateWordRequest.java) checked `@NotBlank` on `meaning` but left `plural` and `meaning` without `@Size(max = 255)`. Because PostgreSQL columns default to `VARCHAR(255)`, longer input would have passed validation and failed later with a database exception instead of a clean `400` response. I aligned the `@Size` bounds with the column limits and cleaned up a duplicate handler method name in `GlobalExceptionHandler`.
- **Single example per word is temporary:** Right now `Word` stores one `exampleSentence` and one `article_id`, and Hibernate manages the schema with `spring.jpa.hibernate.ddl-auto=update`. That kept the first milestone small, but it will break once the same word appears in two different articles.

## What is next

Next is the LLM extraction milestone: refactoring the initial DeepSeek client to use typed DTOs and externalized configuration, defining a JSON schema for candidate words, introducing Flyway migrations to replace `ddl-auto=update`, and moving example sentences into their own table linked to both `Word` and `Article`.

The code is on [GitHub](https://github.com/daozhanglv/wortschatz).
