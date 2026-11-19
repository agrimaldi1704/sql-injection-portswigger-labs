# Lab 4: SQL injection UNION attack, determining the number of columns returned by the query

**Nivel:** Practitioner
**Estado:** ✅ Solved

## Descripción

Primer laboratorio de la serie de ataques **UNION**. Antes de poder combinar una consulta propia con la consulta original mediante `UNION SELECT`, es necesario que ambas consultas devuelvan **el mismo número de columnas** — es un requisito del propio operador `UNION` en SQL.

## Metodología

Se usó la técnica de `ORDER BY` incremental: se ordena el resultado por un número de columna que va aumentando (1, 2, 3...) hasta que la base de datos responde con un error, lo que indica que esa columna no existe.

```
Gifts' ORDER BY 1--
Gifts' ORDER BY 2--
Gifts' ORDER BY 3--   ← Error: la columna 3 no existe
```

## Payload final

```
GET /filter?category=Gifts'+ORDER+BY+2--
```

## Resultado

`ORDER BY 2--` respondió `200 OK`, mientras que `ORDER BY 3--` devolvió un error, confirmando que la consulta original tiene **2 columnas**. Con este dato confirmado, el laboratorio se dio por resuelto.

## Conclusión

`ORDER BY` incremental es la forma más confiable de determinar el número de columnas de una consulta vulnerable antes de construir un ataque `UNION SELECT`, ya que no depende de que las columnas acepten texto (a diferencia de probar directamente con `UNION SELECT NULL,NULL--`).
