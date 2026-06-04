# AI Agent Configuration for Enterprise Development

Este archivo configura cómo los AI coding agents deben trabajar en proyectos enterprise.

## Principios de Trabajo

1. **Nunca escribir código sin tests** - TDD primero
2. **Code review obligatorio** - Todo pasa por review
3. **Documentación actualizada** - Si no está docu, no existe
4. **Seguridad por defecto** - Security first, siempre
5. **Performance desde el día 1** - No optimizar después

## Configuración por Tipo de Proyecto

### Web (Next.js, Astro, React)

```
Skills: vercel-react-best-practices, web-design-guidelines, seo-audit
Testing: playwright-testing
Quality: security-review, performance
```

### Backend (NestJS, Spring Boot, Go)

```
Skills: springboot-patterns, golang-patterns, postgres-patterns
Testing: golang-testing, playwright-testing
Quality: security-review
```

### Full-stack

```
Combinar ambas configuraciones + arquitectura hexagonal
```

## Workflow de Desarrollo

```markdown
1. SPEC.md primero → Escribir especificación completa
2. ARCHITECTURE.md → Definir arquitectura y patrones
3. Código + Tests → TDD/BDD
4. Security Review → Verificar vulnerabilidades
5. Code Review → Peer review
6. Performance Audit → Validar métricas
7. Documentación → Actualizar README/ADRs
```

## Checklists de Calidad

### Antes de Commit

- [ ] Tests pasando
- [ ] Linting sin errores
- [ ] Security scan limpio
- [ ] Types correctos
- [ ] Documentación actualizada

### Antes de Merge

- [ ] Code review aprobado
- [ ] Coverage > 80%
- [ ] Performance benchmarks verdes
- [ ] No secretos en código
- [ ] ADRs actualizados si hay cambios de arquitectura

## Nombres de Commits (Conventional Commits)

```
feat: nueva funcionalidad
fix: corrección de bug
docs: documentación
style: formateo, estilos
refactor: refactorización
test: tests
chore: mantenimiento
perf: optimización
security: seguridad
```

## Estructura de Branching

```
main (production)
├── develop (integración)
│   ├── feature/* (nuevas features)
│   ├── bugfix/* (correcciones)
│   ├── hotfix/* (urgentes production)
│   └── release/* (preparación de release)
```

## Configuración de Revisión

### Lo que el reviewer debe verificar

1. **Diseño** - ¿Está bien estructurado?
2. **Funcionalidad** - ¿Hace lo que debe?
3. **Complejidad** - ¿Se puede simplificar?
4. **Tests** - ¿Hay tests correctos?
5. **Nombres** - ¿Son claros?
6. **Comentarios** - ¿Están bien documentados?
7. **Estilo** - ¿Sigue el style guide?
8. **Seguridad** - ¿Hay vulnerabilidades?

## Integración con Skills

Los skills instalados proporcionan:

| Categoría | Skills | Propósito |
|-----------|--------|-----------|
| Testing | tdd, bdd-*, indexion-sdd | Metodologías de testing |
| Patterns | golang-*, springboot-*, postgres-* | Patrones de código |
| Frontend | vercel-*, web-design-guidelines | UI/UX y perf |
| Quality | playwright-testing, security-review | Validación |
| SEO | seo-audit, ai-seo, performance | Optimización |

## Configuración de Logging

```yaml
# Config para diferentes entornos
development:
  level: debug
  format: detailed

production:
  level: info
  format: json
  redacted_fields: [password, token, secret]
```

## Métricas de Éxito

- **Code Coverage**: > 80%
- **Cyclomatic Complexity**: < 10
- **Technical Debt**: < 5% del codebase
- **Response Time**: p95 < 200ms
- **Error Rate**: < 0.1%
- **Uptime**: > 99.9%