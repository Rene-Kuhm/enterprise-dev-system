# SPEC.md Template

## Nombre del Proyecto

Breve descripción del proyecto.

## Overview

Resumen ejecutivo de qué es y por qué existe.

## Goals

### Goals del Negocio
- Goal 1
- Goal 2

### Goals Técnicos
- Goal 1
- Goal 2

## Non-Goals

Qué NO es este proyecto (delimitar scope).

## Background

Contexto y razón de ser del proyecto.

## Requirements

### Funcional Requirements

| ID | Requirement | Priority | Notes |
|----|-------------|----------|-------|
| FR-01 | Descripción | Must/Should/Could | Notas |

### Non-Functional Requirements

| ID | Requirement | Target | Notes |
|----|-------------|--------|-------|
| NFR-01 | Performance | < 200ms p95 | |
| NFR-02 | Availability | 99.9% | |
| NFR-03 | Security | OWASP Top 10 | |

## User Stories

```
Como [usuario]
Quiero [acción]
Para [beneficio]
```

### Critical User Flows

1. Flow 1
2. Flow 2

## Architecture

### System Context

```
┌─────────────────────────────────────┐
│           External Systems          │
├─────────────────────────────────────┤
│                                     │
│         [This System]               │
│                                     │
└─────────────────────────────────────┘
```

### Component Diagram

```
┌──────────┐     ┌──────────┐
│  Client  │────▶│   API    │
└──────────┘     └────┬─────┘
                      │
              ┌───────┴───────┐
              ▼               ▼
        ┌──────────┐    ┌──────────┐
        │   DB     │    │  Cache   │
        └──────────┘    └──────────┘
```

### Tech Stack

| Component | Technology | Justification |
|-----------|------------|---------------|
| Frontend | React 19 | Required by client |
| API | Node.js | Team expertise |
| DB | PostgreSQL 15 | ACID compliance |

## API Design

### Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | /api/v1/users | List users |
| POST | /api/v1/users | Create user |

### Request/Response Examples

```json
// POST /api/v1/users
Request:
{
  "email": "user@example.com",
  "name": "John Doe"
}

Response (201):
{
  "id": "uuid",
  "email": "user@example.com",
  "name": "John Doe",
  "createdAt": "2026-06-04T10:00:00Z"
}
```

## Data Model

### Users

| Field | Type | Constraints |
|-------|------|-------------|
| id | UUID | PK |
| email | VARCHAR(255) | UNIQUE, NOT NULL |
| name | VARCHAR(100) | NOT NULL |
| createdAt | TIMESTAMP | NOT NULL |

## Security

### Authentication
- JWT tokens con refresh
- OAuth 2.0 para terceros

### Authorization
- RBAC con 3 roles: admin, user, guest

### Data Protection
- Encryption at rest (AES-256)
- TLS 1.3 en tránsito

## Monitoring & Observability

### Metrics
- Request rate
- Error rate
- Latency p50, p95, p99

### Alerts
- Error rate > 1%
- Latency p95 > 500ms

### Logging
- Structured JSON logs
- Log levels: debug, info, warn, error

## Timeline

| Phase | Dates | Deliverables |
|-------|-------|--------------|
| Planning | Jun 1-7 | Spec, Architecture |
| Development | Jun 8-30 | Working features |
| Testing | Jul 1-15 | QA, bug fixes |
| Staging | Jul 16-22 | UAT |
| Production | Jul 23 | Go live |

## Open Questions

1. Pregunta 1
2. Pregunta 2

## Dependencies

| Dependency | Version | Notes |
|------------|---------|-------|
| React | ^19.0 | Latest stable |
| Node.js | >=20 | LTS required |