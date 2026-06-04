# Architecture Patterns

Directorio para patrones de arquitectura documentados.

## Patrones Disponibles

### Domain-Driven Design (DDD)
- Entities, Value Objects
- Aggregates, Repositories
- Domain Events
- Bounded Contexts

### Hexagonal Architecture (Ports & Adapters)
- Core Domain (sin dependencias externas)
- Ports (interfaces de entrada/salida)
- Adapters (implementaciones concretas)

### CQRS (Command Query Responsibility Segregation)
- Separate read/write models
- Event sourcing para escalabilidad
- Eventually consistent

### Event-Driven Architecture
- Event sourcing
- Message queues (Kafka, RabbitMQ)
- CQRS con eventos

### Clean Architecture
- Entities (reglas de negocio)
- Use Cases (aplicación)
- Interface Adapters (presentación)
- Frameworks & Drivers (infraestructura)

## Cuándo Usar Cada Patrón

| Patrón | Cuándo Usar | Cuándo NO |
|--------|-------------|-----------|
| DDD | Domain complejo, múltiples bounded contexts | Dominio simple |
| Hexagonal | Múltiples fuentes de datos, testing difícil | App simple monolith |
| CQRS | Alta write concurrency, reporting complejo | Load balanceado simple |
| Event-Driven | Tiempo real, auditoría completa | Sin necesidad de replay |

## Template para ADR

```markdown
# ADR-XXX: Título descriptivo

## Status
Aceptado | Propuesto | Deprecado

## Context
Descripción del problema o situación

## Decision
Descripción de la decisión tomada

## Consequences
### Positive
- Beneficios de esta decisión

### Negative
- Desventajas o trade-offs

### Neutral
- Elementos a considerar
```

## Template para RFC

```markdown
# RFC-XXX: Título

## Summary
Resumen ejecutivo

## Motivation
Por qué necesitamos este cambio

## Detailed Design
### API Design
### Data Model
### Architecture

## Alternatives Considered
Otras opciones evaluadas

## Adoption Plan
Cómo implementamos esto

## Unresolved Questions
Preguntas abiertas
```