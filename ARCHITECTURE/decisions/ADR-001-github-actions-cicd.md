# ADR-001: Usar GitHub Actions para CI/CD

## Status
Aceptado

## Context
Necesitamos establecer un pipeline de CI/CD automatizado para todos los proyectos de TecnoDespegue. El objetivo es:
- Automatizar testing y deployment
- Reducir errores humanos
- Asegurar calidad consistente
- Acceder rápido a producción

## Decision

Usaremos **GitHub Actions** como sistema de CI/CD por las siguientes razones:

### Ventajas
1. **Integración nativa con GitHub** - Usamos GitHub para todos los repos
2. ** marketplace amplio** - Miles de actions disponibles
3. **Gratuito para públicos** - Open source gratis, privado con minutos limitados
4. **YAML simple** - Fácil de mantener y entender
5. **Secretos seguros** - Gestión integrada de variables de entorno

### Alternativas consideradas

| Alternativa | Por qué no |
|-------------|-----------|
| Jenkins | Requiere servidor propio, mantenimiento adicional |
| GitLab CI | No tenemos GitLab, múltiples plataformas |
| CircleCI | Similar a GitHub Actions pero sin integración directa |
| Travis | Menos features, UI menos intuitiva |

## Consequences

### Positive
- Pipeline centralizado en GitHub
- Automatización sin infraestructura propia
- Testing automático en cada PR
- Deployment automatizado a múltiples ambientes

### Negative
- Vendor lock-in con GitHub
- Minutos de Actions limitados en planes free
- Complejidad para pipelines muy custom

### Neutral
- Necesita curva de aprendizaje para equipo
- YAML puede crecer con el tiempo

## Implementation

El pipeline implementa:

```yaml
# Stages
1. lint          → ESLint, Prettier check
2. type-check    → TypeScript, types validation
3. test          → Unit & Integration tests
4. build         → Docker image build
5. security      → Dependency audit, SAST
6. deploy        → Staging/Production
```

## Related

- ADR-002: Contenerización con Docker
- ADR-003: Monitoreo con CloudWatch