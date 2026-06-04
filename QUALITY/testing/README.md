# Testing Strategy Enterprise

## Testing Pyramid

```
                    ┌───────────┐
                    │    E2E    │  ← Pocas, costosas
                   ┌┴───────────┴┐
                  │  Integration  │  ← Algunas, más rápidas
                 ┌┴─────────────┴┐
                │    Unit Tests   │  ← Muchas, baratas
               ┌┴────────────────┴┐
```

## Niveles de Testing

### 1. Unit Tests (70%)

**Qué probar:**
- Funciones puras
- Clases/Componentes aislados
- Lógica de negocio

**Herramientas:**
- Jest (JS/TS)
- Pytest (Python)
- Go testing (Go)

**Ejemplo:**
```typescript
describe('calculateDiscount', () => {
  it('applies 10% discount for orders over $100', () => {
    const result = calculateDiscount(150);
    expect(result).toBe(135);
  });
  
  it('does not apply discount for orders under $100', () => {
    const result = calculateDiscount(50);
    expect(result).toBe(50);
  });
});
```

### 2. Integration Tests (20%)

**Qué probar:**
- Comunicación entre módulos
- Bases de datos
- APIs internas

**Herramientas:**
- Jest + Supertest
- Playwright (API testing)

**Ejemplo:**
```typescript
describe('POST /api/users', () => {
  it('creates user and returns 201', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ email: 'test@example.com', name: 'Test' })
      .expect(201);
      
    expect(response.body.id).toBeDefined();
    
    // Cleanup
    await User.destroy({ where: { email: 'test@example.com' }});
  });
});
```

### 3. E2E Tests (10%)

**Qué probar:**
- Flows críticos de usuario
- Login/Auth
- Checkout flows
- Funcionalidad core

**Herramientas:**
- Playwright
- Cypress

**Ejemplo Playwright:**
```typescript
import { test, expect } from '@playwright/test';

test('user can complete purchase flow', async ({ page }) => {
  await page.goto('/products');
  
  await page.click('[data-testid="product-1"]');
  await page.click('Add to Cart');
  
  await page.click('[data-testid="checkout-btn"]');
  await page.fill('[data-testid="card-number"]', '4242424242424242');
  
  await page.click('Pay Now');
  
  await expect(page.locator('.success-message')).toBeVisible();
});
```

## TDD Workflow (Test-Driven Development)

```
1. RED    → Escribir test que falla
2. GREEN  → Escribir código mínimo para pasar
3. REFACTOR → Mejorar código manteniendo tests verdes
```

```typescript
// 1. RED - Test que falla
describe('ShoppingCart', () => {
  it('should calculate total price', () => {
    const cart = new ShoppingCart();
    // Test aún no existe - falla
  });
});

// 2. GREEN - Implementación mínima
class ShoppingCart {
  calculateTotal() {
    return 0; // Implementación mínima
  }
}

// 3. REFACTOR - Mejorar
class ShoppingCart {
  private items: CartItem[] = [];
  
  addItem(item: CartItem) {
    this.items.push(item);
  }
  
  calculateTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
}
```

## BDD (Behavior-Driven Development)

### Gherkin Syntax

```gherkin
Feature: User Login

  Scenario: Successful login with valid credentials
    Given I am on the login page
    When I enter "user@example.com" in the email field
    And I enter "password123" in the password field
    And I click the "Login" button
    Then I should be redirected to the dashboard
    And I should see a welcome message
  
  Scenario: Failed login with invalid credentials
    Given I am on the login page
    When I enter "invalid@example.com" in the email field
    And I enter "wrongpassword" in the password field
    And I click the "Login" button
    Then I should see an error message "Invalid credentials"
    And I should remain on the login page
```

### Implementación con Jest

```typescript
import { defineFeature, loadFeature } from 'jest-cucumber';
import { shallow } from 'enzyme';
import LoginPage from './LoginPage';

const feature = loadFeature('./features/login.feature');

defineFeature(feature, test => {
  test('Successful login', ({ given, when, then }) => {
    let wrapper;
    
    given('I am on the login page', () => {
      wrapper = shallow(<LoginPage />);
    });
    
    when('I enter valid credentials', () => {
      wrapper.find('#email').simulate('change', { target: { value: 'user@test.com' }});
      wrapper.find('#password').simulate('change', { target: { value: 'password123' }});
    });
    
    then('I should see the dashboard', () => {
      expect(wrapper.state('redirectTo')).toBe('/dashboard');
    });
  });
});
```

## Coverage Goals

| Tier | Target | Notes |
|------|--------|-------|
| Overall | > 80% | Mínimo required |
| Critical Paths | 100% | Login, payment, auth |
| New Code | 90% | Todo código nuevo |
| Business Logic | 95% | Reglas de negocio |

## Testing Checklist

```markdown
## Antes de PR

- [ ] Unit tests para toda lógica nueva
- [ ] Integration tests para APIs
- [ ] E2E para flows críticos
- [ ] Coverage no bajó
- [ ] Tests corren rápido (< 5min total)
- [ ] Tests son determinísticos (no flaky)

## Checklist de Calidad

- [ ] Tests tienen nombres descriptivos
- [ ] Tests follow Arrange-Act-Assert
- [ ] Tests son independientes (no step-dependency)
- [ ] Tests清理an datos (setup/teardown)
- [ ] Mocks están bien calibrados (no sobremockear)
- [ ] Edge cases cubiertos
```

## Performance Testing

```typescript
describe('Performance', () => {
  it('should load products in under 200ms', async () => {
    const start = performance.now();
    
    await fetchProducts({ page: 1, limit: 20 });
    
    const duration = performance.now() - start;
    expect(duration).toBeLessThan(200);
  });
});
```

## Security Testing

```typescript
describe('Security', () => {
  it('should sanitize SQL input', () => {
    const maliciousInput = "'; DROP TABLE users; --";
    const sanitized = sanitizeInput(maliciousInput);
    
    expect(sanitized).not.toContain("DROP");
  });
  
  it('should validate authorization', async () => {
    const response = await request(app)
      .get('/api/admin/users')
      .set('Authorization', 'Bearer invalid-token');
      
    expect(response.status).toBe(401);
  });
});
```

## Herramientas Recomendadas

| Propósito | Herramienta | Lenguaje |
|-----------|------------|----------|
| Unit + Integration | Jest | JS/TS |
| E2E | Playwright | Multi |
| Coverage | Istanbul / Coverage.py | JS/Python |
| BDD | jest-cucumber | JS/TS |
| API Testing | Supertest | JS |
| Load Testing | k6 | Multi |
```