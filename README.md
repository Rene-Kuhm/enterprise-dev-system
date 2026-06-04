# Enterprise Development System (EDS)

Sistema integral de desarrollo enterprise para crear software de nivel profesional.

## Philosophy

> "El código enterprise no es sobre complejidad, es sobre **claridad, mantenibilidad y escalabilidad**."

Este sistema integra las mejores prácticas de:
- **Google Engineering Practices** - Code review, calidad
- **Spotify Engineering Culture** - Autonomía, equipos pequeños
- **AWS Well-Architected Framework** - 6 pilares de excelencia
- **Microsoft Cloud Design Patterns** - Arquitectura cloud-native

## Pilares del Sistema

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ENTERPRISE DEVELOPMENT SYSTEM                     │
├─────────────┬─────────────┬─────────────┬─────────────┬─────────────┤
│   Design    │   Quality   │  Security   │   DevOps    │  Document   │
│             │             │             │             │             │
│ Architecture│ Code Review │ Threat      │ CI/CD       │ ADRs        │
│ Patterns    │ Testing     │ Modeling    │ Monitoring  │ RFCs        │
│ Clean Code  │ TDD/BDD     │ Compliance  │ Observab.   │ Wikis       │
│ DDD         │ Performance │ SAST/DAST   │ IaC         │ Specs       │
└─────────────┴─────────────┴─────────────┴─────────────┴─────────────┘
```

## Quick Start

```bash
# Clonar y usar el sistema
git clone https://github.com/Rene-Kuhm/enterprise-dev-system.git
cd enterprise-dev-system

# Inicializar en tu proyecto
./scripts/init-project.sh
```

## Estructura del Repo

```
enterprise-dev-system/
├── README.md
├── AGENTS.md                    # Config para AI agents (Mavis, Claude, etc.)
├── ARCHITECTURE/
│   ├── patterns/               # Patrones de diseño
│   ├── decisions/              # ADRs - Architecture Decision Records
│   └── templates/              # Templates de arquitectura
├── QUALITY/
│   ├── code-review/           # Guía de code review (Google style)
│   ├── testing/               # Estrategia de testing
│   └── standards/             # Coding standards
├── SECURITY/
│   ├── threat-modeling/       # Análisis de amenazas
│   ├── secure-coding/          # Guidelines de seguridad
│   └── compliance/             # Checklists de compliance
├── DEVOPS/
│   ├── cicd/                   # Pipelines CI/CD
│   ├── monitoring/            # Observabilidad
│   └── iac/                   # Infrastructure as Code
├── SKILLS/                     # Skills instalados para agents
├── TEMPLATES/                  # Templates reutilizables
└── SCRIPTS/                    # Scripts de automatización
```

## Componentes Principales

### 1. AGENTS.md - Configuración de AI Agents

```markdown
# Este archivo configura cómo los AI coding agents
# (Mavis, Claude Code, Cursor, etc.) deben trabajar
# en proyectos enterprise.
```

### 2. Architecture Patterns

| Pattern | Descripción | Caso de Uso |
|---------|-------------|-------------|
| Hexagonal | Ports & Adapters | Sistemas con múltiples orígenes de datos |
| CQRS | Command Query Responsibility Segregation | Alta escalabilidad read/write |
| Event Sourcing | Estado como secuencia de eventos | Auditoría completa, replay |
| Clean Architecture | Capas independentes | Proyectos complejos mantenibles |
| Microservices | Servicios independientes | Escalabilidad horizontal |

### 3. Code Review (Google Style)

**Checklist de Review:**

- [ ] **Diseño**: ¿Está bien diseñado para el sistema?
- [ ] **Funcionalidad**: ¿Hace lo que el autor pretende?
- [ ] **Complejidad**: ¿Se puede simplificar?
- [ ] **Tests**: ¿Tiene tests correctos y bien diseñados?
- [ ] **Nombres**: ¿Son claros los nombres de variables/clases?
- [ ] **Comentarios**: ¿Son útiles y claros?
- [ ] **Estilo**: ¿Sigue el style guide?
- [ ] **Documentación**: ¿Se actualizó la docs relevante?

### 4. AWS Well-Architected 6 Pilares

1. **Operational Excellence** - Automatización, recovery
2. **Security** - Zero trust, encrypt everything
3. **Reliability** - HA, fault tolerance, DR
4. **Performance Efficiency** - Right-sizing, caching
5. **Cost Optimization** - Reserved instances, spot
6. **Sustainability** - Carbon footprint, efficiency

## Integración con Skills

Skills instalados para este sistema:

```bash
# Development
tdd                          # Test-driven development
bdd-patterns, bdd-principles # Behavior-driven development
indexion-sdd                 # Specification-driven development
golang-patterns              # Go patterns
springboot-patterns          # Spring Boot patterns
postgres-patterns            # PostgreSQL patterns

# Frontend
vercel-react-best-practices  # React/Next.js optimization
web-design-guidelines        # UI/UX guidelines

# Quality
playwright-testing          # E2E testing
security-review             # Security audit

# SEO & Performance
seo-audit, ai-seo           # SEO optimization
performance, convex-perf    # Web vitals optimization
```

## Uso con Gentle-AI

Este sistema está diseñado para integrarse con Gentle-AI:

```yaml
# gentle-ai.config.yaml
integration:
  code_review:
    enabled: true
    tools:
      - security-review
      - playwright-testing
      - golang-patterns

  architecture:
    enabled: true
    patterns:
      - hexagonal
      - cqrs
      - clean-architecture

  quality_gates:
    - security-review
    - performance
    - accessibility
```

## Contribuir

Este es un sistema vivo. Contribuí con:
1. Fork el repo
2. Creá una rama feature
3. seguí los coding standards
4. hacé PR con documentación

## Licencia

MIT - TecnoDespegue © 2026