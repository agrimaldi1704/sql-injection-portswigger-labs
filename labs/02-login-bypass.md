# Lab 2: SQL injection vulnerability allowing login bypass

**Nivel:** Apprentice
**Estado:** ✅ Solved

## Descripción

La aplicación tiene un formulario de login que valida usuario y contraseña contra la base de datos. El objetivo es iniciar sesión como `administrator` **sin conocer su contraseña**.

## Vulnerabilidad

La consulta de autenticación concatena directamente los valores del formulario:

```sql
SELECT * FROM users WHERE username = 'INPUT_USER' AND password = 'INPUT_PASS'
```

## Metodología

1. Se interceptó la petición `POST /login` con Burp Proxy y se envió a Repeater.
2. En el campo **username** se inyectó una comilla simple seguida de un comentario SQL, para cerrar la cadena del nombre de usuario y **comentar el resto de la consulta** (incluida la verificación de contraseña):
   ```
   administrator'--
   ```
3. El campo **password** se dejó con cualquier valor (o vacío), ya que nunca se evalúa.

## Payload final

```
username: administrator'--
password: (cualquier valor)
```

## Resultado

La consulta resultante queda así:

```sql
SELECT * FROM users WHERE username = 'administrator'--' AND password = 'cualquiercosa'
```

Todo lo que sigue a `--` se interpreta como comentario, por lo que la verificación de contraseña nunca se ejecuta. La base de datos valida al usuario `administrator` solo por su nombre, y la aplicación responde con un `302 Found` redirigiendo a `/my-account?id=administrator`, confirmando el inicio de sesión exitoso.

## Conclusión

Este laboratorio demuestra cómo un comentario SQL (`--`) puede usarse para **anular condiciones completas** de una consulta de autenticación, logrando un bypass de login sin necesidad de conocer ninguna credencial válida.
