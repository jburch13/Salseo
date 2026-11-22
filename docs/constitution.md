# Constitución — Salseo

1. **Stack mínimo**: Flutter + Firebase (Auth anónima y Firestore), `provider` y `go_router`. No se añade ninguna dependencia sin justificarla en la spec activa.
2. **Spec antes que código**: cada cambio funcional parte de una spec en `specs/`. El código solo implementa lo que la spec describe; si no coinciden, se actualiza primero la spec.
3. **MVVM estricto**: las Views no tienen lógica de negocio ni importan nada de `data/` ni de Firebase. La lógica vive en los ViewModels y los repositorios.
4. **Tests obligatorios**: cada ViewModel y cada repositorio nuevo tiene tests unitarios con fakes. Una tarea no se cierra hasta que `flutter test` y `flutter analyze` pasan sin errores.
5. **Persistencia solo vía repositorios**: los datos se leen y escriben en Firestore únicamente a través de repositorios. Ningún dato que identifique al usuario (nombre, email, teléfono) se guarda junto a sus publicaciones.
6. **Idioma**: identificadores y comentarios en inglés. Los textos visibles para el usuario van en español y centralizados para i18n, nunca escritos directamente en los widgets.