# Lab 7: SQL injection attack, querying the database type and version on Oracle

**Nivel:** Practitioner
**Estado:** ✅ Solved

## Descripción

Laboratorio enfocado en el reconocimiento (fingerprinting) de la base de datos: el objetivo es extraer, mediante un ataque UNION, el tipo y la versión exacta del motor de base de datos **Oracle** que utiliza la aplicación.

## Particularidad de Oracle

A diferencia de MySQL o SQL Server, **Oracle exige que todo `SELECT` incluya una cláusula `FROM`**, incluso cuando no se está consultando ninguna tabla real. Para estos casos, Oracle provee una tabla ficticia llamada `dual`.

## Metodología

1. Se determinó el número de columnas con `ORDER BY` incremental (2 columnas).
2. Se identificó la columna que aceptaba texto con:
   ```
   Gifts' UNION SELECT 'a','a' FROM dual--
   ```
3. Para obtener la versión del motor, se usó la tabla de sistema `v$version`, que en Oracle almacena el string de versión en la columna `banner` (y puede devolver varias filas):
   ```
   Gifts' UNION SELECT banner,NULL FROM v$version--
   ```

## Payload final

```
GET /filter?category=Gifts'+UNION+SELECT+banner,NULL+FROM+v$version--
```

## Resultado

La respuesta mostró el string completo de versión de Oracle (`11.2.0.2.0`), cumpliendo el objetivo del laboratorio.

## Conclusión

Este laboratorio introduce dos particularidades propias de Oracle que lo diferencian de otros motores: la obligatoriedad de `FROM` en todo `SELECT` (resuelta con la tabla `dual`) y el uso de la tabla de sistema `v$version` para obtener la versión del motor, en contraste con la variable `@@version` disponible directamente en MySQL y SQL Server.
