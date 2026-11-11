# API que espera GemoEats (para conectar con tu Gemo-Store Laravel)

La app funciona en modo demo sin backend. Para datos reales, añade estos
endpoints JSON a tu proyecto Laravel (p. ej. en `routes/api.php` con un
`ApiTiendaController`). Todos son públicos (sin login), como la tienda pública
actual `/t/{slug}`.

## 1. Listado de tiendas cercanas / búsqueda
`GET /api/tiendas?lat=10.49&lng=-66.90&q=pizza`

Debe devolver solo negocios **activos y suscritos** que tengan tienda pública.
```json
{
  "data": [
    {
      "id": 1,
      "slug": "burger-town",
      "nombre": "Burger Town",
      "descripcion": "Hamburguesas artesanales",
      "logo_url": "https://.../logo.png",
      "portada_url": "https://.../portada.jpg",
      "lat": 10.4825,
      "lng": -66.9010,
      "categoria": "Hamburguesas",
      "abierto": true,
      "rating_prom": 4.7,
      "rating_count": 213,
      "tiempo_entrega_min": 25,
      "costo_envio": 1.5,
      "direccion": "Av. Principal 4"
    }
  ]
}
```
> La app igual reordena por distancia usando `lat`/`lng`. Puedes ignorar
> `lat`/`lng`/`q` en el servidor y filtrar/orden en el cliente, pero lo ideal
> es filtrar en SQL por radio y por `nombre LIKE q`.
>
> `lat`/`lng` por tienda pueden salir de `tienda_config` (hay campos GPS en el
> modelo de pedidos/tienda). `rating_prom`/`rating_count` requieren una tabla
> nueva `tienda_ratings` (ver punto 4).

## 2. Menú de una tienda
`GET /api/tiendas/{slug}`
```json
{
  "tienda": { "id": 1, "slug": "burger-town", "nombre": "Burger Town", ... },
  "productos": [
    { "id": 101, "nombre": "Cheeseburger", "descripcion": "...",
      "precio": 6.5, "stock": 50, "imagen_url": "https://...",
      "categoria": "Hamburguesas" }
  ]
}
```
Usa los `productos` activos del negocio (`negocio_id`), con su `categoria`.

## 3. Crear pedido
`POST /api/tiendas/{slug}/pedido`  (JSON)
```json
{
  "nombre": "Juan", "cedula": "V123", "telefono": "0412...",
  "direccion": "Calle 1", "lat": 10.49, "lng": -66.90,
  "tipo_entrega": "domicilio", "notas": "sin cebolla",
  "metodo_pago": "Efectivo", "total": 15.5,
  "items": [ { "producto_id": 101, "nombre": "Cheeseburger", "cantidad": 2, "precio": 6.5 } ]
}
```
Respuesta:
```json
{ "ok": true, "mensaje": "Pedido recibido", "pedido_id": 123,
  "seguimiento_url": "https://tudominio.com/t/burger-town/track/123" }
```
Reutiliza tu lógica actual de `TiendaPublicaController@pedido` (crea el
`pedido` + `pedido_detalle` + campos GPS). Protege con `throttle` como ya haces.

## 4. Puntuar una tienda
`POST /api/tiendas/{slug}/rating`  (JSON)
```json
{ "estrellas": 5, "comentario": "Excelente" }
```
Respuesta `{ "ok": true }`. Sugerencia de tabla nueva:
```sql
CREATE TABLE IF NOT EXISTS tienda_ratings (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  negocio_id BIGINT UNSIGNED NOT NULL,
  estrellas TINYINT NOT NULL,
  comentario VARCHAR(500) NULL,
  cliente_cedula VARCHAR(40) NULL,
  creado_en TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_rating_negocio (negocio_id)
);
```
Y `rating_prom`/`rating_count` en el punto 1 salen de
`AVG(estrellas)` / `COUNT(*)` por `negocio_id`.

## Seguridad
- Devuelve solo negocios activos/suscritos; nunca datos de negocios ajenos.
- Valida `slug` y limita `items` a productos de ese negocio al crear el pedido.
- Mantén `throttle` en `pedido` y `rating`.
- Escapa/valida `comentario` y `notas` antes de guardarlos.
