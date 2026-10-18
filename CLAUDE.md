# StarkApps

Base para aplicaciones locales, ligeras y con poco consumo de RAM. Responde en español, de forma directa y práctica.

## Contexto y alcance

- Estas son las instrucciones comunes. Consulta `docs/README.md` para procedimientos y referencias; antes de trabajar en una aplicación, lee también su `CLAUDE.md` y la documentación relevante.
- Cada aplicación vive en `apps/<nombre>/`, con propósito, alcance, modalidad y excepciones en su `CLAUDE.md`. Trabaja desde su carpeta y conserva cambios ajenos.
- Crea aplicaciones, código, dependencias o skills solo cuando formen parte del trabajo solicitado. Evita funcionalidades e infraestructura anticipadas.
- Las skills compartidas están en `.claude/skills/`; consulta las disponibles cuando sean relevantes.

## Stack predeterminado

- Frontend: Svelte 5 + TypeScript + Vite; componentes shadcn-svelte con Tailwind CSS 4, añadidos según necesidad.
- Backend: Go. Escritorio con Wails o navegador con servidor HTTP Go local, según lo solicitado; no implementes ambas modalidades por anticipado.
- SQLite: `modernc.org/sqlite` mediante `database/sql`, SQL directo, consultas parametrizadas, transacciones y límite explícito de conexiones. No añadas un ORM por defecto.
- Migraciones: Goose (`github.com/pressly/goose/v3`) como biblioteca, SQL versionado en `internal/database/migrations/` e incorporado con `go:embed`. Aplica las pendientes antes de atender operaciones; si fallan, detén el arranque. No distribuyas su CLI ni ejecutes reversiones automáticas.
- Usa versiones estables y fija dependencias. Las betas, excepciones al stack y servicios externos requieren acuerdo explícito y documentado. La aplicación debe funcionar sin internet.

## Organización del código

- Cada aplicación mantiene su módulo Go y dependencias frontend. Comparte código entre aplicaciones solo ante una necesidad concreta.
- Conserva el punto de entrada convencional de la versión elegida de Wails: arranque y conexión de dependencias separados de la lógica. Los métodos expuestos al frontend y los handlers HTTP delegan en servicios Go independientes del transporte.
- Agrupa tipos, operaciones y consultas por funcionalidad en `internal/`, con nombres claros como `documents`. Reserva `internal/database/` para inicialización y migraciones, e `internal/storage/` para adjuntos cuando hagan falta.
- Organiza el frontend por funcionalidades y componentes reutilizados. Empieza con pocos archivos; evita capas, interfaces, carpetas vacías y paquetes genéricos como `utils` o `common` sin responsabilidad clara.
- Coloca pruebas Go junto al código en `_test.go`. Modifica las fuentes de resultados regenerables, como los bindings de Wails. Los componentes incorporados por shadcn-svelte sí son código propio editable.

## Diseño e interfaz

- Geist, de Vercel, es la referencia visual. Antes de diseñar pantallas o cambiar patrones, revisa las decisiones de la aplicación y consulta la documentación oficial de Geist, Web Interface Guidelines y shadcn-svelte enlazada en el README general. Si no puedes consultarla, usa lo documentado y declara la limitación.
- Implementa con Svelte y shadcn-svelte, adaptando los ejemplos React/Next.js sin introducir esos frameworks. Parte de sus estilos y comportamiento nativos; usa tokens semánticos y variantes compartidas.
- Prioriza superficies neutras, jerarquía clara, bordes sutiles, espaciado consistente y acentos funcionales. Prefiere Geist Sans y Geist Mono donde corresponda, empaquetadas localmente y con alternativa de sistema.
- Contempla carga, vacío, error y éxito. Conserva accesibilidad, teclado, foco visible y movimiento reducido; comprueba distintos tamaños de ventana y zoom.

## Datos y adjuntos

- Cada aplicación tiene una raíz configurable mediante `APP_DATA_DIR`, con `app.db` y `storage/` como equivalente local de un bucket. Las ubicaciones y su resolución están en el README general.
- Excluye datos reales de Git y del empaquetado. La instalación, actualización, compilación o limpieza del proyecto no deben eliminarlos.
- Importar un adjunto copia su contenido a `storage/` y conserva el original. Usa un identificador único para el nombre interno; guarda nombre original, ruta relativa a la raíz de datos, tipo y tamaño en SQLite. Enlazar un archivo externo debe ser una opción explícita.
- Maneja archivos desde Go y la raíz configurada, sin depender del directorio de ejecución. Comprueba que las rutas administradas permanezcan dentro de `storage/` y no publiques la carpeta completa.
- Implementa respaldos y restauración coherentes de la base y los adjuntos; contempla fallos entre operaciones de SQLite y archivos.

## Verificación y documentación

- Mantén recarga automática durante desarrollo. Verifica el recorrido relevante hasta la persistencia y el resultado visible; ajusta las pruebas al riesgo. Comprueba el ejecutable si cambian empaquetado o recursos locales.
- Mide recursos contando todos los procesos, incluido el WebView. Al entregar, distingue resultados comprobados, estimaciones y limitaciones.
- Mantén instrucciones duraderas en `CLAUDE.md` y detalles cambiantes en `docs/README.md`: el raíz para información compartida, el de cada aplicación para la particular. Enlaza en lugar de duplicar.
- Actualiza las secciones afectadas en el mismo trabajo cuando cambien comportamiento, arquitectura, diseño, datos o comandos. Describe el estado actual, elimina lo obsoleto y evita registros cronológicos o secciones de relleno.
- Crea otro documento solo si el tema es independiente y dificulta consultar el existente; enlázalo desde el README. Los cambios menores sin impacto documentado no requieren actualizarlo.

## Seguimiento

- Usa el gestor de trabajo configurado para cada aplicación; no dupliques su backlog en documentación. Mantén las cuentas, organizaciones, credenciales y rutas personales en configuración local no versionada.
- Antes de publicar, revisa archivos y metadatos de los commits. Usa un alias público y el correo `noreply` de GitHub; nunca incluyas datos personales, secretos ni configuración privada.

## Mantenimiento de esta base

- El repositorio fuente del scaffolding es [pablorxz/stark-apps](https://github.com/pablorxz/stark-apps). Consúltalo para revisar la versión y los cambios de las instrucciones compartidas.
- Cuando se solicite actualizar esta base, compara el contenido local con el repositorio y adapta las secciones correspondientes de `CLAUDE.md` y `docs/README.md`, preservando las decisiones particulares de cada aplicación. Si la tarea incluye publicar, sincroniza los cambios con ese repositorio tras revisar el diff.
