# Lab 1: SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

**Nivel:** Apprentice
**Estado:** ✅ Solved

## Descripción

La aplicación es una tienda en línea que filtra productos por categoría usando un parámetro `category` en la URL. La consulta SQL subyacente solo muestra productos **released** (publicados), pero es posible manipular el filtro para que también devuelva productos no publicados.

## Vulnerabilidad

El parámetro `category` se concatena directamente en una consulta SQL sin sanitización:

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

## Metodología

1. Con **Burp Proxy** interceptando el tráfico, se identificó la petición:
   ```
   GET /filter?category=Gifts
   ```
2. Se envió a **Repeater** y se probó cerrar la cadena de texto e inyectar una condición siempre verdadera, comentando el resto de la consulta:
   ```
   Gifts'--
   ```
3. Al no mostrar productos (la condición `released = 1` seguía aplicando porque el comentario eliminó esa parte, pero el filtro de categoría quedó roto), se ajustó el payload para neutralizar ambas condiciones con `OR`:
   ```
   Gifts' OR 1=1--
   ```

## Payload final

```
GET /filter?category=Gifts'+OR+1=1--
```

## Resultado

La consulta se convierte en:

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```

Como `1=1` siempre es verdadero, la condición `OR` hace que se devuelvan **todos los productos**, incluyendo los no publicados (ocultos). El laboratorio se marcó como resuelto al mostrarse un producto fuera de catálogo.

## Conclusión

Este laboratorio introduce el concepto base de la inyección SQL: cuando la entrada del usuario se concatena sin sanitizar dentro de una consulta, es posible alterar la lógica booleana de la cláusula `WHERE` para eludir filtros de negocio.
