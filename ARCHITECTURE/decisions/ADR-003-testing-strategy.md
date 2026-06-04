# ADR-003: Testing Strategy con Jest y Playwright

## Status
Aceptado

## Context
Necesitamos una estrategia de testing unificada para:
- Garantizar calidad del código
- Prevenir regressions
- Testing rápido en CI
- E2E coverage para flows críticos

## Decision

Usaremos **Jest** para tests unitarios/integración y **Playwright** para E2E.

### Stack

| Tipo | Herramienta | Propósito |
|------|------------|-----------|
| Unit | Jest | Fast, good DX, built-in coverage |
| Integration | Jest + Supertest | API testing |
| E2E | Playwright | Cross-browser, reliable |

### Coverage Goals

- Unit: > 80%
- Integration: 100% de endpoints
- E2E: Login, Checkout, APIs críticas

## Consequences

### Positive
- Jest tiene excelente DX y speed
- Playwright es más confiable que Cypress
- Coverage automático
- CI integration straightforward

### Negative
- Mantener dos frameworks de testing
- Playwright slower que unit tests
- Flaky tests possible sin buenas prácticas

## Related

- ADR-001: CI/CD con GitHub Actions
- ADR-004: BDD con Gherkin