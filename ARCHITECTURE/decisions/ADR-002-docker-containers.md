# ADR-002: Contenerización con Docker

## Status
Aceptado

## Context
Necesitamos estandarizar cómo desplegamos aplicaciones para:
- Consistencia entre ambientes (dev, staging, prod)
- Reproducibilidad de builds
- Aislamiento de dependencias
- Escalabilidad horizontal

## Decision

Usaremos **Docker** como tecnología de contenedorización.

### Stack de Docker

```dockerfile
# Multi-stage build para Node.js
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

### Image Security Hardening

- Usar imagenes oficiales con alpine
- No correr como root (USER node)
- Minimal packages
- Scan con Trivy en CI

## Consequences

### Positive
- Same environment everywhere
- Fast scaling with orchestration
- Resource isolation
- Easy rollback

### Negative
- Docker learning curve
- Additional complexity
- Registry costs

### Neutral
- Kubernetes future-ready
- Cloud-agnostic templates