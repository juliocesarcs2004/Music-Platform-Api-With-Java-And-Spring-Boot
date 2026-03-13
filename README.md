# ScreenSound (Java + Spring Boot)

ScreenSound is a Spring Boot application for managing artists and songs, with persistence in PostgreSQL and an OpenAI-powered lookup for artist information.

Important: despite this repository name containing "API", the current implementation is a **console (CLI) application**, not a REST API.

## Current Features

- Register artists (`SOLO`, `DUPLA`, or `BANDA`)
- Register songs linked to existing artists
- List all songs from all artists
- Search songs by artist name
- Query OpenAI to get information about an artist
- Persist data with Spring Data JPA + PostgreSQL

## Tech Stack

- Java 17
- Spring Boot 3.5.11
- Spring Data JPA
- PostgreSQL JDBC Driver
- OpenAI Java SDK (`com.openai:openai-java:4.23.0`)
- Maven Wrapper (`./mvnw`)

## Project Structure

```text
src/main/java/br/com/jccs/screensound
├── ScreenSoundApplication.java          # Spring Boot entry point + CommandLineRunner
├── Principal/Principal.java             # CLI menu and user interactions
├── model/
│   ├── Artista.java                     # Artist entity
│   ├── Musica.java                      # Song entity
│   └── TipoArtista.java                 # SOLO, DUPLA, BANDA enum
├── repository/
│   └── ArtistaRepository.java           # JPA repository + custom query
└── service/
    └── ConsultaChatGPT.java             # OpenAI integration for artist lookup
```

## Data Model

### `Artista` (`artistas`)

- `id` (`Long`, generated)
- `nome` (`String`, unique)
- `tipo` (`TipoArtista`, stored as string)
- `musicas` (`List<Musica>`, one-to-many, eager fetch, cascade all)

### `Musica` (`musicas`)

- `id` (`Long`, generated)
- `titulo` (`String`)
- `artista` (`Artista`, many-to-one)

## CLI Menu (Current Flow)

When the app starts, it opens this interactive menu:

1. Register artists
2. Register songs
3. List songs
4. Search songs by artist
5. Query artist information (OpenAI)
9. Exit

## Configuration

The app uses environment-variable placeholders in `application.properties`.

### Required Environment Variables

- `OPENAI_API_KEY`: OpenAI API key used by `ConsultaChatGPT`
- `CONNECTION_URL`: JDBC URL for PostgreSQL (for example: `jdbc:postgresql://localhost:5432/screensound`)
- `USERNAME`: PostgreSQL username
- `PASSWORD`: PostgreSQL password

### Current `application.properties`

```properties
spring.application.name=screensound
openai.api.key=${OPENAI_API_KEY}

spring.datasource.url=${CONNECTION_URL}
spring.datasource.username=${USERNAME}
spring.datasource.password=${PASSWORD}
spring.datasource.driver-class-name=org.postgresql.Driver
hibernate.dialect=org.hibernate.dialect.HSQLDialect

spring.jpa.hibernate.ddl-auto=update
```

Note: there is a probable mismatch in dialect configuration (`HSQLDialect` while using PostgreSQL). For PostgreSQL, use `org.hibernate.dialect.PostgreSQLDialect` if you need an explicit dialect.

## Running the Project

### 1. Set environment variables

```bash
export OPENAI_API_KEY="your_openai_key"
export CONNECTION_URL="jdbc:postgresql://localhost:5432/screensound"
export USERNAME="your_db_user"
export PASSWORD="your_db_password"
```

### 2. Run tests

```bash
./mvnw test
```

### 3. Start the application

```bash
./mvnw spring-boot:run
```

The program starts in terminal and waits for menu input.

## Repository Query Details

`ArtistaRepository` includes:

- `findByNomeContainingIgnoreCase(String nome)`
- `buscaMusicasPorArtista(String nome)` using:
  `SELECT m FROM Artista a JOIN a.musicas m WHERE a.nome ILIKE %:nome%`

`ILIKE` is PostgreSQL-specific and enables case-insensitive matching.

## Test Coverage (Current State)

Current automated test:

- `ScreensoundApplicationTests.contextLoads()`

This verifies Spring context startup, but there are no business-logic or repository behavior tests yet.

## Limitations / Next Evolution

- No REST controllers yet
- CLI text is currently in Portuguese
- Minimal test coverage
- OpenAI request prompt is fixed and not configurable

## License

No license file is currently defined in this repository.
