# Lab 6: SQL injection UNION attack, retrieving data from other tables

**Nivel:** Practitioner
**Estado:** ✅ Solved

## Descripción

Laboratorio que combina todo lo aprendido en los anteriores: la aplicación tiene una tabla llamada `users` con columnas `username` y `password` (nombres conocidos de antemano, a diferencia de laboratorios posteriores donde hay que descubrirlos). El objetivo es iniciar sesión como `administrator`.

## Metodología

1. Se determinó el número de columnas con `ORDER BY` incremental (2 columnas).
2. Se confirmó qué columna aceptaba texto con `UNION SELECT 'a',NULL--`.
3. Con esa información, se construyó el ataque UNION final apuntando directamente a la tabla `users`:
   ```
   Gifts' UNION SELECT username,password FROM users--
   ```

## Payload final

```
GET /filter?category=Gifts'+UNION+SELECT+username,password+FROM+users--
```

## Resultado

La respuesta mostró la lista completa de usuarios junto con sus contraseñas (en texto plano), incluyendo al usuario `administrator`. Con esas credenciales se inició sesión en **My account**, resolviendo el laboratorio.

## Conclusión

Este laboratorio consolida el flujo completo de un ataque UNION: determinar columnas → encontrar columna de texto → extraer datos de una tabla específica. Es la base conceptual para los laboratorios siguientes, donde ni el nombre de la tabla ni el de las columnas se conocen de antemano y deben descubrirse mediante el catálogo del sistema de la base de datos.
