# Freight Rate API

A Spring Boot service for managing logistics contract rates: create a rate, file it, amend it with full version history, and quote shipments against it.

Inspired by the rate filing and amendment workflows I worked on in enterprise pricing systems. This is an independent, simplified re-implementation with made-up data.

## Features

- Create contract rates per customer, lane (origin → destination) and container type
- Filing workflow: `DRAFT → FILED`
- Amendments never overwrite. The old version becomes `SUPERSEDED` and a new version links back to it, so every change is auditable
- Quote endpoint finds the filed rate valid on the ship date and prices the shipment
- Bean Validation on requests, RFC 7807 error responses
- Flyway migrations, PostgreSQL constraints backing up the business rules
- Unit tests with JUnit 5 + Mockito, GitHub Actions CI

## Tech stack

Java 17 · Spring Boot 3 · Spring Data JPA · PostgreSQL · Flyway · JUnit 5 · Docker · GitHub Actions

## Run it

```bash
docker compose up -d       # starts PostgreSQL
mvn spring-boot:run
```

Swagger UI: http://localhost:8080/swagger-ui.html

## Try it

```bash
# 1. Create a draft rate
curl -X POST localhost:8080/api/v1/rates -H "Content-Type: application/json" -d '{
  "customerCode":"ACME","origin":"INBLR","destination":"NLRTM","containerType":"40HC",
  "baseRate":1800,"currency":"USD","validFrom":"2026-01-01","validTo":"2026-12-31"}'

# 2. File it
curl -X POST localhost:8080/api/v1/rates/1/file

# 3. Amend the price (creates version 2)
curl -X POST localhost:8080/api/v1/rates/1/amendments -H "Content-Type: application/json" -d '{"baseRate":1950}'

# 4. Get a quote
curl -X POST localhost:8080/api/v1/quotes -H "Content-Type: application/json" -d '{
  "customerCode":"ACME","origin":"INBLR","destination":"NLRTM","containerType":"40HC",
  "quantity":3,"shipDate":"2026-05-10"}'
```

## Endpoints

| Method | Path | Purpose |
|---|---|---|
| POST | `/api/v1/rates` | Create a draft rate |
| GET | `/api/v1/rates/{id}` | Get a rate |
| GET | `/api/v1/customers/{code}/rates` | List a customer's filed rates |
| POST | `/api/v1/rates/{id}/file` | File a draft rate |
| POST | `/api/v1/rates/{id}/amendments` | Amend a filed rate (new version) |
| POST | `/api/v1/quotes` | Quote a shipment |

## Roadmap (ideas to extend)

- [ ] Surcharges (fuel/BAF, peak season) as separate line items
- [ ] Prevent overlapping validity windows for the same lane
- [ ] Integration tests with Testcontainers
- [ ] Bulk rate upload from Excel/CSV
- [ ] Role-based access (pricing manager vs. sales) with Spring Security
- [ ] Dockerfile for the app itself
