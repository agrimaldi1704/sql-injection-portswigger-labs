# Lab 3: SQL injection with filter bypass via XML encoding

**Nivel:** Practitioner
**Estado:** ✅ Solved

## Descripción

Este laboratorio contiene una vulnerabilidad de inyección SQL idéntica a la del filtro de categoría de productos, pero protegida por un **Web Application Firewall (WAF)** que bloquea payloads que contengan palabras clave SQL reconocibles en texto plano (como `UNION`, `SELECT`, `OR`, etc.).

## Vulnerabilidad

El parámetro `category` sigue siendo inyectable, pero cualquier payload "obvio" es detectado y bloqueado por el WAF antes de llegar a la aplicación.

## Metodología

1. Se confirmó que un payload estándar (`' OR 1=1--`) era bloqueado por el WAF.
2. Se instaló la extensión **Hackvertor** en Burp Suite (Extensions → BApp Store), que permite codificar fragmentos de un payload dentro de etiquetas especiales.
3. Se usó la etiqueta `<@hex_entities>...<@/hex_entities>` de Hackvertor para envolver las palabras clave SQL del payload, convirtiéndolas en **entidades hexadecimales XML**. De esta forma, el WAF —que hace coincidencia de texto plano— no reconoce las palabras SQL, pero el parser XML del servidor las decodifica antes de que lleguen al motor de base de datos.
4. Esto permitió reconstruir el ataque UNION completo sin ser bloqueado por el WAF.

## Payload final (conceptual)

```
' UNION SELECT username, password FROM users--
```
codificado parcialmente mediante Hackvertor con `<@hex_entities>` sobre las palabras clave sensibles, de modo que el WAF solo ve entidades hexadecimales en lugar de texto SQL reconocible.

## Resultado

El servidor decodificó el payload antes de ejecutarlo, el WAF no lo bloqueó, y se logró extraer usuario y contraseña del administrador, iniciando sesión exitosamente y resolviendo el laboratorio.

## Conclusión

Este laboratorio ilustra una limitación común de los WAF basados en coincidencia de patrones (pattern matching) de texto plano: no evalúan el payload **después** de que la aplicación lo decodifica/transforma. Codificar el payload en un formato que el WAF no reconoce, pero que el servidor sí interpreta correctamente, es una técnica clásica de evasión de WAF.
