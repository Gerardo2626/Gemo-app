# GemoEats 🍔

App móvil en **Flutter** para pedir comida en las tiendas suscritas a
**Gemo-Store**. El usuario ve las tiendas cercanas, las busca, abre el menú,
arma su carrito, hace el pedido, guarda tiendas como **favoritas** y las
**puntúa** con estrellas.

## Funciona al instante (modo DEMO)
La app trae datos de ejemplo. Puedes compilarla y probar todo el flujo sin
tener aún el backend conectado. Cuando configures tu servidor, usa la API real.

## Cómo ejecutarla

> Este paquete contiene el código de la app (`lib/` + `pubspec.yaml`). Las
> carpetas nativas (`android/`, `ios/`) se generan con un comando.

```bash
# 1) Dentro de la carpeta del proyecto, genera las plataformas nativas
#    (NO borra tu lib/ ni tu pubspec.yaml)
flutter create . --org com.gemostore --project-name gemo_eats

# 2) Descarga dependencias
flutter pub get

# 3) Conecta un teléfono o abre un emulador y corre
flutter run
```

## Permisos que debes añadir (ubicación + internet)

**Android** — `android/app/src/main/AndroidManifest.xml`, dentro de `<manifest>`:
```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
```

**iOS** — `ios/Runner/Info.plist`:
```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>Usamos tu ubicación para mostrarte las tiendas más cercanas.</string>
```

## Conectar con tu Gemo-Store real

Abre `lib/services/app_config.dart` y pon la URL de tu servidor:
```dart
static const String baseUrl = 'https://tudominio.com';
```
Con eso, la app deja el modo demo y llama a los endpoints `/api/...`
descritos en **BACKEND_API.md**. Si la red falla, cae a los datos demo para
no romperse.

## Qué incluye

- **Inicio**: cabecera con tu ubicación, buscador, filtros por categoría y
  lista de tiendas ordenadas por cercanía (distancia real por GPS).
- **Mapa**: tiendas sobre OpenStreetMap (sin API key), con tu posición.
- **Tienda**: portada, rating, botón de favorito, menú por categorías y
  carrito con cantidades.
- **Carrito y checkout**: datos del cliente, a domicilio / retiro, método de
  pago y envío del pedido.
- **Favoritos**: tus tiendas guardadas (local).
- **Pedidos**: historial de lo que has pedido desde la app (local).
- **Puntuar**: estrellas + comentario, se guarda local y se envía a la API.

## Arquitectura

- Estado con `provider` (`Cart`, `FavoritesService`, `RatingsService`,
  `OrdersService`, `CatalogProvider`).
- Capa de red en `lib/services/api_service.dart` con respaldo a
  `lib/data/demo_data.dart`.
- UI con Material 3, tipografía Poppins (google_fonts) y tema cálido.

## Nota
No pude compilar Flutter en el entorno donde se generó; el código está
revisado estáticamente. Corre `flutter analyze` y `flutter run` en tu equipo.
