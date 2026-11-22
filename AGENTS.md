# AGENTS.md — salseo

## Proyecto
App en Flutter dirigida a adolescentes y estudiantes universitarios. Red social anónima para compartir cotilleos (gossip) sobre universidades/institutos a los que están suscritos, interactuar en publicaciones de otros usuarios y participar en conversaciones sin revelar su identidad.

Estado actual: proyecto recién creado (plantilla de Flutter). La arquitectura descrita abajo es la que debe seguir todo el código nuevo.

## Comandos
- Instalar dependencias: `flutter pub get`
- Ejecutar app: `flutter run`
- Ejecutar tests: `flutter test`
- Analizar código: `flutter analyze`
- Formatear código: `dart format .`
- Generar build Android: `flutter build apk`
- Generar build iOS: `flutter build ios`

## Stack técnico
- Flutter + Dart SDK ^3.12.2, Material 3.
- Backend: Firebase (Firebase Auth con autenticación anónima, Cloud Firestore para datos).
- Estado: `ChangeNotifier` + `ListenableBuilder` (sin librerías externas de estado).
- Inyección de dependencias: `provider`.
- Navegación: `go_router`.
- Tests: `flutter_test` + fakes escritos a mano; `mocktail` solo cuando un fake no sea práctico.

## Arquitectura (MVVM)
Se sigue la arquitectura recomendada por Flutter, tomando como referencia el caso de estudio oficial (Compass app):
https://docs.flutter.dev/app-architecture/case-study

### Capas y flujo de datos
```
View  ──>  ViewModel  ──>  Repository  ──>  Service  ──>  Firebase
 (UI)   <── estado ──    (fuente de verdad)  (API externa)
```
El flujo es unidireccional: la View emite eventos, el ViewModel llama al Repository, el Repository llama al Service, y el resultado vuelve hacia arriba hasta que el ViewModel actualiza su estado y llama a `notifyListeners()`.

### Capa UI
- **View**: widgets de presentación. Normalmente un `StatelessWidget` que recibe solo `key` y `viewModel`. Se reconstruye con `ListenableBuilder`. Sin lógica de negocio: solo pinta estado y conecta eventos con comandos del ViewModel.
- **ViewModel**: extiende `ChangeNotifier`. Recibe repositorios por constructor y los guarda en campos privados. Expone estado inmutable mediante getters y acciones mediante comandos (`Command0`, `Command1`). Nunca accede directamente a un Service ni a Firebase.
- Relación 1:1 entre View y ViewModel. Un ViewModel puede usar varios repositorios.
- Los ViewModels de pantalla se crean en la configuración de rutas (`routing/router.dart`), leyendo los repositorios con `context.read()`.

### Capa de datos
- **Service**: clase sin estado que envuelve una única fuente externa (p. ej. `AuthService` sobre Firebase Auth, `FirestoreService` sobre Firestore). Devuelve modelos de API envueltos en `Result`. Solo los usan los repositorios.
- **Repository**: fuente de verdad de un tipo de dato y único lugar donde se modifica. Se encarga de caché, reintentos y de mapear modelos de API a modelos de dominio. Se define como clase abstracta (`PostRepository`) con implementaciones concretas (`PostRepositoryFirebase`) para poder sustituirlas en tests.
- **Modelos**:
  - Modelos de API/persistencia: sufijo `ApiModel` (p. ej. `PostApiModel`), en `data/models/`. Nunca llegan a la UI.
  - Modelos de dominio: nombre limpio (p. ej. `Post`, `Comment`, `School`), en `domain/models/`. Inmutables.

### Capa de dominio
- Solo `domain/models/`. No se usan casos de uso (use-cases) por defecto; los ViewModels llaman directamente a los repositorios. Añadir use-cases únicamente si una lógica se comparte entre varios ViewModels.

### Utilidades comunes
- `Result<T>` (`Ok` / `Error`) para devolver éxito o error en operaciones asíncronas en lugar de lanzar excepciones entre capas.
- `Command` (`Command0`, `Command1`) para encapsular acciones asíncronas del ViewModel con estados `running`, `error` y `completed`.

### Estructura de carpetas
```
lib/
├── config/                  # Configuración e inyección de dependencias (providers)
├── data/
│   ├── models/              # Modelos de API/Firestore (<entity>_api_model.dart)
│   ├── repositories/        # <entity>_repository.dart + implementaciones
│   └── services/            # Wrappers de Firebase y otras APIs
├── domain/
│   └── models/              # Modelos de dominio inmutables
├── routing/                 # router.dart y routes.dart (go_router)
├── ui/
│   ├── core/
│   │   ├── themes/          # Tema Material 3 personalizado
│   │   └── ui/              # Widgets compartidos entre features
│   └── <feature>/
│       ├── view_models/     # <feature>_view_model.dart
│       └── widgets/         # <feature>_screen.dart y widgets propios
├── utils/                   # Result, Command, helpers
└── main.dart

test/                        # Refleja la estructura de lib/ (data, domain, ui, utils)
testing/
├── fakes/                   # Fake<Nombre> (repositorios y servicios falsos)
└── models/                  # Datos de prueba compartidos
```
- `ui/` se organiza por feature (p. ej. `feed`, `post_detail`, `compose_post`, `schools`, `auth`).
- `data/` se organiza por tipo, porque repositorios y servicios se comparten entre features.

## Estilo y convenciones
- Seguir las convenciones oficiales de Dart y Flutter.
- UI Material 3 con personalización visual propia.
- Iconografía Material Icons o iconografía personalizada.
- Utilizar snake_case para nombres de archivos.
- Utilizar PascalCase para clases, enums y tipos.
- Utilizar camelCase para variables, métodos y propiedades.
- Sufijos de clase: `...Screen`, `...ViewModel`, `...Repository`, `...Service`, `...ApiModel`, `Fake...`.
- Inyección de dependencias por constructor con parámetros `required` nombrados y campos privados `final`.
- Preferir widgets pequeños, reutilizables y especializados.
- Evitar widgets excesivamente grandes y métodos con demasiadas responsabilidades.
- Utilizar const siempre que sea posible.
- Preferir composición de widgets frente a herencia.
- Identificadores en inglés, mensajes de usuario en español.
- Mantener los textos de la interfaz preparados para una futura internacionalización.

## Tests
- ViewModels: tests unitarios usando repositorios fake (sin Firebase).
- Repositories: tests unitarios usando services fake.
- Views: widget tests usando los mismos fakes, inyectados con `Provider.value` / `ChangeNotifierProvider.value`.
- Ningún test debe depender de Firebase real ni de red.

## Reglas
- Lee docs/constitution.md y la spec activa antes de tocar código.
- Priorizar soluciones sencillas, mantenibles y escalables.
- Respetar las capas: la UI nunca importa nada de `data/services/` ni de Firebase.
- No modifiques archivos dentro de `specs/` salvo petición explícita.

## Al terminar cualquier tarea
- Ejecuta `flutter test` y confirma en tu respuesta que todo pasa.
