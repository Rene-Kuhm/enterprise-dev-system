# Security Standards

## Principios de Seguridad

1. **Zero Trust** - Nunca confiar, siempre verificar
2. **Defense in Depth** - Múltiples capas de seguridad
3. **Least Privilege** - Solo los permisos necesarios
4. **Secure by Default** - Seguridad integrada desde el diseño

## OWASP Top 10 (2021)

### 1. Broken Access Control
- Verificar permisos en cada request
- No confiar en IDs de URL
- Deny by default

### 2. Cryptographic Failures
- No store passwords en texto claro
- Usar hashing (bcrypt, argon2)
- TLS para todo

### 3. Injection
- Sanitizar inputs
- Usar prepared statements
- No concatenar SQL

### 4. Insecure Design
- Threat modeling
- Secure design patterns
- Banned虐宠 analysis

### 5. Security Misconfiguration
- Hardening default configs
- Automatizar config validation
- Minimal permissions

### 6. Vulnerable Components
- Update dependencies
- Remove unused packages
- Monitor CVEs

### 7. Auth Failures
- Strong password policies
- MFA enabled
- Session timeout

### 8. Data Integrity Failures
- Validate all data
- Sign/verify messages
- Audit trails

### 9. Logging Failures
- Log security events
- Maintain audit trail
- Alert on anomalies

### 10. SSRF (Server-Side Request Forgery)
- Validate URLs
- Whitelist allowed domains
- Disable unnecessary protocols

## Checklist de Seguridad

```markdown
## Autenticación

- [ ] Passwords hashed con bcrypt/argon2
- [ ] MFA disponible
- [ ] Rate limiting en login
- [ ] Session timeout
- [ ] No tokens en URLs

## Autorización

- [ ] Todos los endpoints protegidos
- [ ] Roles y permisos definidos
- [ ] Resource-level permissions
- [ ] Principle of least privilege

## Input Validation

- [ ] Todos los inputs validados
- [ ] Whitelist cuando sea posible
- [ ] Sanitización en output
- [ ] No eval()
- [ ] Prepared statements para SQL

## Data Protection

- [ ] Encryption at rest
- [ ] TLS en tránsito
- [ ] Secrets en env vars
- [ ] No secrets en código
- [ ] PII handling compliant

## Dependencies

- [ ] Deps actualizadas
- [ ] No known CVEs
- [ ] Minimal dependencies
- [ ] License compliance
```

## Secure Coding Guidelines

### Input Validation

```typescript
// ❌ Never
const query = `SELECT * FROM users WHERE id = ${userId}`;

// ✅ Always
const sanitizedId = parseInt(userId, 10);
if (isNaN(sanitizedId)) throw new ValidationError('Invalid ID');
const query = `SELECT * FROM users WHERE id = ?`;
```

### Password Handling

```typescript
// ✅ Using bcrypt
import bcrypt from 'bcrypt';

async function hashPassword(password: string): Promise<string> {
  const saltRounds = 12;
  return bcrypt.hash(password, saltRounds);
}

async function verifyPassword(password: string, hash: string): Promise<boolean> {
  return bcrypt.compare(password, hash);
}
```

### Environment Variables

```typescript
// ❌ Never in code
const API_KEY = 'sk-1234567890abcdef';

// ✅ From environment
const API_KEY = process.env.API_KEY;
if (!API_KEY) throw new Error('API_KEY is required');
```

## Security Review Process

### Pre-commit

- [ ] Scan de secretos (git-secrets, gitleaks)
- [ ] Dependabot actualizado
- [ ] SAST corriendo (ESLint security plugin)

### Pre-merge

- [ ] Security review aprobado
- [ ] Dependency audit passed
- [ ] Vulnerability scan limpio

### Post-deploy

- [ ] DAST corriendo en staging
- [ ] Monitoring de anomalías
- [ ] Logs de seguridad centralizados

## Tools Recomendados

| Category | Tool | Purpose |
|----------|------|---------|
| SAST | ESLint security, SonarQube | Static analysis |
| DAST | OWASP ZAP, Burp Suite | Dynamic testing |
| Dependency | npm audit, Snyk, Dependabot | Vulnerabilities |
| Secrets | Gitleaks, GitGuardian | Secret detection |
| Container | Trivy, Grype | Container scanning |

## Threat Modeling Template

```markdown
## Asset
¿Qué estamos protegiendo?

## Threat Actor
¿Quién podría atacar?
- External hackers
- Malicious insiders
- Competitors

## Attack Surface
¿Cómo pueden atacar?
- API endpoints
- User input
- Third-party integrations

## Vulnerabilities
¿Qué debilidades explotan?

## Impact
¿Qué pasa si tienen éxito?
- Data breach
- Service disruption
- Financial loss

## Mitigation
¿Cómo prevenimos?
- Security controls
- Monitoring
- Response plan
```