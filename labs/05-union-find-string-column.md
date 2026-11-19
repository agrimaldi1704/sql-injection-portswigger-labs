# Lab 5: SQL injection UNION attack, finding a column containing text

**Nivel:** Practitioner
**Estado:** ✅ Solved

## Descripción

Una vez conocido el número de columnas de la consulta vulnerable, el siguiente paso de un ataque UNION es identificar **cuál(es) de esas columnas acepta(n) datos de tipo texto (string)**, ya que la información que se quiere extraer (usuarios, contraseñas, etc.) normalmente es texto.

## Metodología

Con el número de columnas ya confirmado (2, mediante `ORDER BY`), se probó sustituir cada posición de columna por un valor de texto fijo, dejando las demás como `NULL`:

```
Gifts' UNION SELECT 'a',NULL--
Gifts' UNION SELECT NULL,'a'--
```

Si el motor de base de datos acepta texto en esa posición, la consulta se ejecuta sin error y el valor `'a'` aparece reflejado en algún punto de la respuesta HTML (al final de la lista de productos). Si la columna es de un tipo incompatible (por ejemplo, numérico), la base de datos devuelve un error.

## Payload final

```
GET /filter?category=Gifts'+UNION+SELECT+'a',NULL--
```

## Resultado

El payload con `'a'` en la primera posición devolvió `200 OK` y mostró el carácter `a` en la página, confirmando que **la primera columna acepta texto**. Con esta información se resolvió el laboratorio.

## Conclusión

Antes de poder extraer datos reales con un ataque UNION, es indispensable confirmar qué columna(s) son compatibles con texto. Probar con un valor distintivo (`'a'`, o un string aleatorio proporcionado por el propio laboratorio) y observar si se refleja en la respuesta es la forma estándar de hacerlo.
