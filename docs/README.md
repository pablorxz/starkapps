# Documentación general de StarkApps

Las reglas y el stack predeterminado están en [AGENTS.md](../AGENTS.md). Aquí se mantienen procedimientos, estructura y referencias compartidas.

Repositorio fuente: [pablorxz/starkapps](https://github.com/pablorxz/starkapps). El procedimiento para actualizar las instrucciones está en la sección **Mantenimiento de esta base** del `AGENTS.md`.

## Estado actual

Solo hay instrucciones y documentación. `apps/` y `.agents/skills/` contienen únicamente un `.gitkeep` vacío para conservarlas en Git; esta preparación no instala dependencias ni configura aplicaciones. Cuando se cree una, actualiza esta sección con enlaces a sus instrucciones y documentación.

## Claude Code y Codex

`AGENTS.md` es la única fuente de instrucciones compartidas. Codex lo carga de forma nativa; `CLAUDE.md` contiene solo `@AGENTS.md`, una importación que Claude Code admite sin enlaces simbólicos ni configuración personal. Edita siempre el archivo canónico.

Inicia el agente desde la raíz para tareas generales o desde `apps/<nombre>/` para una aplicación. Cada futura app tendrá su propio `AGENTS.md` y un `CLAUDE.md` con la misma importación local; no copies las reglas comunes. Para comprobar la carga, consulta `/context` en Claude y pide a Codex que identifique las instrucciones aplicables. Esta base no modifica ajustes globales de los agentes.

## Entorno y ejecución

Al iniciar una aplicación, comprueba Go, Node.js y un gestor de paquetes. Para escritorio, verifica además el CLI de Wails, WebView2 y los requisitos de plataforma de la versión estable elegida. Registra aquí únicamente el entorno común comprobado; fija las versiones particulares en cada proyecto.

Dentro de un proyecto Wails, `wails doctor` verifica el entorno, `wails dev` inicia desarrollo y `wails build` genera el ejecutable. Son referencias para futuras aplicaciones, no comandos para esta base vacía. Documenta los comandos definitivos de cada app; en modalidad navegador, configura Vite y la recarga del servidor Go.

### Notas de integración

- **SQLite:** modernc evita CGO para el controlador, no los requisitos nativos de Wails ni garantiza menor RAM o más velocidad. Mide las operaciones reales antes de cambiar la capa de acceso.
- **Goose:** reutiliza la conexión `*sql.DB` y un proveedor con `embed.FS`. Los archivos SQL admiten `Up` y `Down`; los recursos de migración se incluyen en el ejecutable, sin archivos externos para el usuario.
- **UI:** incorpora componentes en `frontend/src/lib/components/ui/`. Actualizar paquetes no actualiza automáticamente ese código local. Geist es una referencia visual; sus clases y variables no existen automáticamente en shadcn-svelte: tradúcelas a los tokens del tema.

## Estructura de aplicaciones

Ejemplo orientativo para Wails, no una lista de carpetas que deban crearse vacías. Conserva los archivos adicionales del generador de la versión elegida.

```text
apps/<nombre>/
├── AGENTS.md               # Propósito, alcance y reglas particulares
├── CLAUDE.md               # Solo @AGENTS.md
├── docs/
│   └── README.md
├── main.go
├── app.go
├── go.mod
├── go.sum
├── wails.json
├── internal/
│   ├── documents/          # Ejemplo de funcionalidad
│   ├── database/
│   │   └── migrations/
│   └── storage/            # Código de adjuntos, si se necesita
├── frontend/
│   └── src/lib/
│       ├── components/
│       └── features/
├── build/                  # Recursos y resultados de empaquetado Wails
└── data/                   # Datos de desarrollo, excluidos de Git y del paquete
    ├── app.db
    └── storage/            # Contenido de los adjuntos
```

En Go, separa primero archivos dentro de un paquete; extrae otro paquete cuando tenga responsabilidad propia. En una app de navegador se omiten los archivos de Wails. `cmd/` puede servir para varios ejecutables, pero no es obligatorio.

El `AGENTS.md` de la aplicación explica propósito, alcance, modalidad, restricciones y excepciones, con referencia a `../../AGENTS.md`. Las reglas particulares se aplican dentro de esa app; las comunes se heredan de la raíz. Su `docs/README.md` reúne secciones de desarrollo/verificación, implementación, diseño, datos y decisiones/limitaciones, conforme exista contenido útil. No repitas el propósito ni copies esquemas o listas de dependencias que ya mantiene el código.

Las decisiones de diseño particulares incluyen tema, tipografías, ajustes compartidos y enlaces a los patrones oficiales consultados. El conocimiento útil para varias aplicaciones se integra aquí; el específico permanece en su app.

## Ubicación y conservación de datos

La raíz de datos se resuelve una vez al arrancar:

1. Si existe `APP_DATA_DIR`, usa esa ruta absoluta. El comando de desarrollo proporciona por defecto `apps/<nombre>/data/`, sin sobrescribir una ubicación elegida por el usuario.
2. Si no existe, usa `%LOCALAPPDATA%\StarkApps\<id-app>\data\` en Windows, obteniendo la carpeta del usuario del sistema.

El comando de desarrollo debe calcular su ruta desde el proyecto. La ejecución normal no presupone dónde están los fuentes. Mantén un identificador estable por aplicación, independiente de su nombre visible; cambiar la configuración no mueve datos automáticamente.

El backend crea `app.db` y `storage/` cuando se necesitan. Una ruta guardada en SQLite, como `storage/<id>.pdf`, sigue siendo válida al trasladar la raíz completa. `internal/storage/` contiene código; `data/storage/` contiene archivos y nunca se incorpora al frontend o al ejecutable.

SQLite y el sistema de archivos no comparten una transacción. Al importar o eliminar, define cómo resolver operaciones incompletas y limpiar temporales u huérfanos.

Para respaldar, obtén una copia consistente de SQLite mediante su API de respaldo o un cierre controlado, y coordínala con los adjuntos. No copies solo `app.db` mientras está abierta. La restauración debe recuperar ambos; los archivos externos enlazados no están incluidos.

## Skills

Usa `.agents/skills/<nombre>/SKILL.md` para una futura skill compartida, o la misma estructura dentro de una app si es exclusiva. Por ahora solo contiene el marcador `.gitkeep`. Cada skill tendrá metadatos `name` y `description`, instrucciones y, solo si hacen falta, recursos relativos a su carpeta. Una skill aporta un procedimiento reutilizable, sin duplicar reglas ni documentación.

Codex descubre `.agents/skills/` automáticamente desde la carpeta de trabajo hasta la raíz del repositorio. Claude Code descubre nativamente `.claude/skills/`, no `.agents/skills/`: aquí consulta las skills compartidas mediante la instrucción de `AGENTS.md`, sin prometer comandos `/nombre` automáticos. Si una futura skill necesita ese descubrimiento nativo en Claude, añade entonces un adaptador mínimo en `.claude/skills/<nombre>/SKILL.md` con metadatos y una referencia al archivo canónico, sin copiar su procedimiento. No crees adaptadores ni carpetas exclusivas de un agente por anticipado.

## Referencias oficiales

- Go: [organización](https://go.dev/doc/modules/layout) y [paquetes](https://go.dev/blog/package-names).
- Wails: [estructura](https://wails.io/docs/gettingstarted/firstproject/) y [desarrollo](https://wails.io/docs/gettingstarted/development/).
- Datos: [modernc.org/sqlite](https://pkg.go.dev/modernc.org/sqlite), [proveedor Goose](https://pressly.github.io/goose/documentation/provider/), [respaldo SQLite](https://www.sqlite.org/backup.html) y [carpetas Windows](https://learn.microsoft.com/en-us/windows/win32/shell/knownfolderid).
- UI: [shadcn-svelte](https://www.shadcn-svelte.com/docs) y [Vite](https://www.shadcn-svelte.com/docs/installation/vite).
- Vercel: [Geist](https://vercel.com/geist/introduction), [colores](https://vercel.com/geist/colors), [tipografía](https://vercel.com/geist/typography), [fuentes](https://vercel.com/font) y [Web Interface Guidelines](https://vercel.com/design/guidelines).
- Codex: [AGENTS.md](https://learn.chatgpt.com/docs/agent-configuration/agents-md) y [skills](https://learn.chatgpt.com/docs/build-skills).
- Claude Code: [importar AGENTS.md](https://code.claude.com/docs/en/memory#share-one-file-with-other-coding-tools) y [skills](https://code.claude.com/docs/en/skills).
