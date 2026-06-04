# Code Review Standards - Google Style

Basado en [Google Engineering Practices](https://google.github.io/eng-practices/)

## Principios Fundamentales

1. **El objetivo principal** es mejorar la salud del codebase a largo plazo
2. **No buscar perfección**, buscar mejoras significativas
3. **Respeto y profesionalismo** siempre
4. **Comentar con propósito** - enseñar, no menospreciar

## Qué Revisar

### 1. Diseño
- ¿El código está bien diseñado para el sistema?
- ¿Es apropiado para el sistema?
- ¿Las interacciones entre componentes tienen sentido?

### 2. Funcionalidad
- ¿Hace lo que el autor pretende?
- ¿Es bueno para los usuarios?
- ¿Maneja edge cases?
- ¿La UI se ve bien?

### 3. Complejidad
- ¿Se puede simplificar?
- ¿Otros devs podrán entenderlo fácilmente?
- ¿Hay sobreingeniería?

### 4. Tests
- ¿Tiene tests correctos?
- ¿Los tests son buenos (cubren casos importantes)?
- ¿Son mantenibles?
- ¿Prueban lo correcto (no solo cobertura alta)?

### 5. Nombres
- ¿Son claros y descriptivos?
- ¿No hay confusiones posibles?
- ¿Están en el idioma correcto (español/inglés según proyecto)?

### 6. Comentarios
- ¿Son útiles?
- ¿Explican el "por qué", no el "qué"?
- ¿Están actualizados?

### 7. Estilo
- ¿Sigue el style guide?
- ¿Es consistente con el resto del codebase?

### 8. Documentación
- ¿Se actualizó README/ADRs/Specs?
- ¿El código está documentado donde corresponde?

## Cómo Escribir Comentarios

### Buenos comentarios ✨

```
"Considerá usar un Map aquí en lugar de un array para O(1) lookup 
en lugar de O(n). Esto mejora significativamente el performance 
para grandes datasets."

"Esta validación es importante porque sin ella un usuario podría 
acceder a datos de otro usuario (IDOR vulnerability)."
```

### Evitar ❌

```
"Nombre de variable malo." → ¿Cuál? ¿Por qué?

"Esto está mal." → ¿Qué está mal exactamente?

"¿Por qué hiciste esto así?" → ¿Qué sugiere el reviewer?
```

## Estándares de Aprobación

### LGTM (Looks Good to Me)
- El código puede mergearse
- Puede tener sugerencias menores opcionales

### NACK (Negative Acknowledgement)
- Problemas críticos que deben resolverse
- Razones claras de por qué

###blocking vs no-blocking
- **Blocking**: Debe resolverse antes de merge
- **Suggestion**: Opcional, mejora la calidad

## Velocidad de Review

- **Ideal**: < 24 horas para la primera respuesta
- ** Máximo**: No dejar un review pendiente más de 48 horas
- ** Para cambios pequeños ( < 100 líneas): mismo día

## Size de CL (Change List)

### Recomendado: < 400 líneas

Cambios grandes son:
- Difíciles de revisar bien
- Más propensos a bugs
- Más estresantes para el autor

### Si es necesario un cambio grande

Dividir en múltiples CLs:
1. Refactor primero (sin cambios funcionales)
2. Agregar funcionalidad
3. Fixes y polish

## Checklist de Review

```markdown
## Para el Autor

- [ ] El CL tiene un objetivo claro
- [ ] Está dividido en commits lógicos
- [ ] Tests incluidos
- [ ] Documentación actualizada
- [ ] No hay secretos hardcodeados
- [ ] Se probó localmente

## Para el Revisor

- [ ] Entiendo qué hace el código
- [ ] El diseño tiene sentido
- [ ] No hay security issues
- [ ] Tests son correctos
- [ ] Nombres son claros
- [ ] Documentación adecuada
- [ ] Performance aceptable
```

## Template de PR/CL Description

```markdown
## Summary
Breve descripción del cambio

## Motivation
Por qué se necesita este cambio

## Changes
- Cambio específico 1
- Cambio específico 2

## Testing
Cómo se probó
- Test unitarios: ✓
- Test integration: ✓
- Manual testing: ✓

## Screenshots (si aplica)
Antes / Después

## Related Issues
Closes #XX
```

## Recursos

- [Google Engineering Practices](https://google.github.io/eng-practices/)
- [Code Review Best Practices](https://www.atlassian.com/agile/code-reviews)
- [How to do a code review - Google](https://google.github.io/eng-practices/review/reviewer/)
```