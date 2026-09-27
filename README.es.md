# APEXlang — Customers (proyecto demo)

🌐 También disponible en: [Português (Brasil)](README.md) | [English](README.en.md)

## Qué es este proyecto

Un proyecto demo/de prueba para explorar el desarrollo de aplicaciones
Oracle APEX usando **APEXlang** (un DSL textual para APEX) junto con
**Claude Code** como agente de IA, cerrando el ciclo con **SQLcl** y la
**extensión Oracle SQL Developer para VS Code** (que importa/publica la
app directamente en el workspace de APEX desde el editor).

No es un proyecto de producción — es un laboratorio para validar el flujo
de extremo a extremo: describir una funcionalidad en lenguaje natural → el
agente genera `.apx` a partir de contratos de gramática y del esquema real
de la base de datos → lo importa en APEX desde VS Code → lo prueba en el
navegador → corrige lo que falle → registra lo aprendido para no repetir el
mismo error en la próxima página.

## Qué se ha hecho hasta ahora

- App Oracle APEX `CUSTOMERS` creada y publicada (workspace `ZONA`, app id
  `100`).
- Esquema inicial (`Database/DDL_v001.sql`): 5 tablas — clientes, tipos de
  teléfono, representantes de ventas, teléfonos del cliente y pedidos —
  con clave primaria subrogada `RAW(16)`/`SYS_GUID()`, columnas de
  auditoría mantenidas por trigger, claves foráneas y restricciones de
  unicidad.
- Primera funcionalidad solicitada mediante un prompt e implementada de
  extremo a extremo: mantenimiento de **Tipos de Teléfono** — página con
  Interactive Grid (solo se puede editar Nombre/Descripción, Código es de
  solo lectura, sin insert/delete en línea), creación de registros a
  través de una página modal separada, más una entrada de menú y de
  breadcrumb.
- Convenciones del proyecto y varios problemas reales de runtime
  encontrados en el camino (con sus causas) documentados en los archivos
  `CLAUDE.md`, para que cada página nueva ya nazca sin repetir los mismos
  errores.

## Cómo está organizado el repositorio

```
APEXlang/
├── CLAUDE.md                        # convenciones generales del repo para el agente de IA
├── README.md / README.en.md / README.es.md
├── customers/                       # app Oracle APEX (raíz de la app — ver abajo)
├── Database/                        # evolución del esquema (DDL) + diccionario de datos
└── Prompts/                         # historial de solicitudes de funcionalidades, por app
```

### `customers/` — app Oracle APEX

Contiene los artefactos `.apex`/`.apx` que genera el skill APEXlang y que
la extensión Oracle SQL Developer importa/exporta directamente en el
workspace de APEX. A diferencia de la convención más habitual del skill
(`applications/<app>/`), aquí `customers/` ya **es** la raíz de la app.

- `application.apx` — definición de la aplicación (nombre, tema,
  autenticación, navegación).
- `pages/` — un archivo `.apx` por página (`p00001-home.apx`,
  `p00002-phone-types.apx`, `p00003-phone-types-create.apx`, ...).
- `shared-components/` — breadcrumbs, listas de menú de navegación,
  autenticación y autorización, tema, LOVs, archivos estáticos.
- `supporting-objects/` — scripts de instalación/desinstalación de la app.
- `deployments/default.json` — workspace y app id de destino (`ZONA` /
  `100`).
- `apex-exports/` — copias de seguridad automáticas en `.sql` que la
  extensión genera antes de cada Import; es una salida de seguridad, no
  una fuente de verdad.

### `Database/` — esquema y diccionario de datos

Es la fuente de verdad del modelo de datos, alimentada por el proyecto
Database de Oracle JDeveloper.

- `DDL_vNNN.sql` — un archivo por versión, generado por JDeveloper. **Cada
  archivo es un diff incremental** (solo lo que cambió en esa versión), no
  una instantánea completa del esquema — la verdad actual es la suma de
  todos los archivos aplicados en orden.
- `Tables/<TABLA>.md` — un diccionario de datos por tabla: nombre del
  campo (igual al nombre de la columna en la base de datos), una etiqueta
  de visualización en portugués y una descripción. Complementa la DDL con
  los datos que esta no aporta (etiqueta, descripción, indicación de campo
  oculto/no editable) y se trata como metadato autoritativo, al mismo
  nivel que la DDL — no como un simple comentario suelto.

### `Prompts/` — historial de solicitudes

Un archivo por funcionalidad solicitada, que nunca se sobrescribe:
`Prompts/<app>/NNN_NombreDeLaFuncionalidad.md`. Cada archivo registra la
solicitud original y, cuando corresponde, los problemas encontrados
durante la implementación y cómo se resolvieron — funciona como un
changelog vivo por funcionalidad.

## Herramientas involucradas

- **Claude Code** con el skill `apexlang`, que genera y valida `.apx` a
  partir de contratos de gramática y de metadatos reales del compilador de
  Oracle APEX (no "inventa" sintaxis).
- **SQLcl** (`D:\Bin\sqlcl`) — usado por el skill para consultar metadatos
  del compilador y por la extensión de VS Code para conectarse a la base
  de datos.
- **Extensión Oracle SQL Developer para VS Code** — reconoce archivos
  `.apx`/`.apex` y publica (Import) la app en el workspace `ZONA`
  directamente desde el editor.

## Convenciones y detalles para quien trabaje en el proyecto

Las reglas sobre cómo generar código, cómo nombrar las cosas (inglés para
el ámbito técnico/identificadores, portugués para todo lo que ve el
usuario final) y la lista de problemas de runtime ya descubiertos a lo
largo del proyecto están documentadas en:

- [`CLAUDE.md`](CLAUDE.md) — convenciones generales del repositorio.
- [`customers/CLAUDE.md`](customers/CLAUDE.md) — convenciones específicas
  de la app `CUSTOMERS`.
