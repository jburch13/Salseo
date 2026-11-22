# Salseo

Red social anónima para adolescentes y estudiantes universitarios. Los usuarios se suscriben a sus institutos o universidades para compartir cotilleos, interactuar con las publicaciones de otros y participar en conversaciones sin revelar su identidad.

> Estado: en desarrollo inicial.

## Funcionalidades previstas
- Registro e inicio de sesión anónimos.
- Suscripción a institutos y universidades.
- Feed de publicaciones por centro.
- Publicar cotilleos, comentar e interactuar con publicaciones.
- Conversaciones sin revelar la identidad.

## Stack
- [Flutter](https://flutter.dev) (Dart SDK ^3.12.2), Material 3
- Firebase: Authentication (anónima) y Cloud Firestore
- Estado: `ChangeNotifier` + `ListenableBuilder`
- Inyección de dependencias: [`provider`](https://pub.dev/packages/provider)
- Navegación: [`go_router`](https://pub.dev/packages/go_router)

## Arquitectura
La app sigue el patrón **MVVM** recomendado por Flutter, tomando como referencia el
[caso de estudio oficial (Compass app)](https://docs.flutter.dev/app-architecture/case-study).

```
View  ──>  ViewModel  ──>  Repository  ──>  Service  ──>  Firebase
```

- **View**: widgets sin lógica de negocio que pintan el estado del ViewModel.
- **ViewModel**: `ChangeNotifier` que expone el estado de la UI y los comandos.
- **Repository**: fuente de verdad de cada tipo de dato. Convierte los modelos de API en modelos de dominio.
- **Service**: envuelve Firebase u otras APIs externas.

```
lib/
├── config/        # Inyección de dependencias
├── data/          # models (API), repositories, services
├── domain/        # Modelos de dominio
├── routing/       # Rutas (go_router)
├── ui/            # core (tema y widgets compartidos) + una carpeta por feature
├── utils/         # Result, Command, helpers
└── main.dart
test/              # Tests, misma estructura que lib/
testing/           # Fakes y datos de prueba
```

El detalle completo de convenciones está en [AGENTS.md](AGENTS.md).

## Primeros pasos

### Requisitos
- Flutter SDK compatible con Dart ^3.12.2
- Un proyecto de Firebase configurado (cuando se integre el backend)

### Comandos
```bash
flutter pub get      # Instalar dependencias
flutter run          # Ejecutar la app
flutter test         # Ejecutar los tests
flutter analyze      # Análisis estático
dart format .        # Formatear el código
flutter build apk    # Build Android
flutter build ios    # Build iOS
```

## Documentación
- [AGENTS.md](AGENTS.md): guía de arquitectura y convenciones para desarrolladores y agentes de IA.
- [docs/constitution.md](docs/constitution.md): principios del proyecto.
