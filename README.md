# SQL Injection Labs — PortSwigger Web Security Academy

Documentación de 10 laboratorios prácticos de **inyección SQL** resueltos en el [PortSwigger Web Security Academy](https://portswigger.net/web-security/sql-injection), usando **Burp Suite Community Edition** como herramienta principal de interceptación y manipulación de peticiones HTTP.

> ⚠️ Todos los laboratorios se realizaron en entornos de práctica aislados y autorizados por PortSwigger, diseñados específicamente para el aprendizaje ético de seguridad web. Ninguna técnica aquí descrita fue aplicada contra sistemas sin autorización.

## Objetivo

Este repositorio forma parte de mi portafolio de ciberseguridad, enfocado en demostrar comprensión práctica de vulnerabilidades web del OWASP Top 10 (A03:2021 — Injection), como preparación para roles de **analista junior de ciberseguridad / SOC**.

## Herramientas utilizadas

- **Burp Suite Community Edition** (Proxy, Repeater)
- **Hackvertor** (extensión de Burp para codificación/evasión de filtros WAF)
- Navegador del Burp Suite embebido

## Metodología general

Para cada laboratorio se siguió el mismo flujo de trabajo:

1. Interceptar la petición vulnerable con **Proxy → HTTP history**
2. Enviarla a **Repeater** para manipular el parámetro inyectable
3. Determinar el número de columnas de la consulta (`ORDER BY` incremental)
4. Identificar qué columna(s) aceptan texto (`UNION SELECT 'a',NULL--`)
5. Extraer información del esquema (`information_schema` / `all_tables`) cuando el nombre de tabla/columnas no se conoce
6. Construir el payload `UNION SELECT` final para extraer los datos objetivo
7. Confirmar la resolución del laboratorio (badge **Solved**)

## Laboratorios resueltos

| # | Laboratorio | Motor de BD | Técnica principal |
|---|---|---|---|
| 1 | [Retrieving hidden data](labs/01-retrieving-hidden-data.md) | — | Manipulación de parámetro de filtro |
| 2 | [SQL injection login bypass](labs/02-login-bypass.md) | — | Comentario SQL (`'--`) para evadir validación de password |
| 3 | [Filter bypass vía codificación XML (WAF bypass)](labs/03-waf-bypass-xml-encoding.md) | — | Evasión de WAF con Hackvertor (hex entities) |
| 4 | [UNION attack — determinar número de columnas](labs/04-union-determine-columns.md) | — | `ORDER BY` incremental |
| 5 | [UNION attack — encontrar columna con texto](labs/05-union-find-string-column.md) | — | `UNION SELECT 'a',NULL--` |
| 6 | [UNION attack — extraer datos de otra tabla](labs/06-union-retrieve-data-other-tables.md) | — | `UNION SELECT username,password FROM users--` |
| 7 | [Consultar tipo y versión de BD (Oracle)](labs/07-querying-db-version-oracle.md) | Oracle | `v$version`, tabla `dual` |
| 8 | [Consultar tipo y versión de BD (MySQL/MSSQL)](labs/08-querying-db-version-mysql-mssql.md) | MySQL / SQL Server | `@@version` |
| 9 | [Listar contenido de BD (no-Oracle)](labs/09-listing-db-contents-non-oracle.md) | MySQL / SQL Server | `information_schema.tables` / `.columns` |
| 10 | [Listar contenido de BD (Oracle)](labs/10-listing-db-contents-oracle.md) | Oracle | `all_tables` / `all_tab_columns` |

## Conceptos clave demostrados

- **UNION-based SQL injection** para extraer datos arbitrarios de la base de datos
- **Autenticación bypass** mediante comentarios SQL
- **Enumeración de esquema** (tablas y columnas) sin conocimiento previo del modelo de datos
- **Diferencias de sintaxis entre motores de BD**: Oracle (requiere `FROM dual`, usa `all_tables`) vs. MySQL/SQL Server (`information_schema`, `@@version`)
- **Evasión de WAF** mediante codificación de payloads (hex entities XML vía Hackvertor)
- Uso profesional de **Burp Suite** (Proxy → Repeater) como flujo estándar de pentesting web

## Autor

Alfredo Grimaldi García — [LinkedIn](https://www.linkedin.com/) · Portafolio de ciberseguridad en [GitHub](https://github.com/agrimaldi1704)
