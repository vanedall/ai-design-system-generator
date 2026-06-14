# HANDOFF — Sistema generativo de Design System (Figma + IA)

> Pegá este documento al inicio de un chat nuevo para retomar el proyecto sin perder contexto.
> Junto con este archivo, pegá el `CLAUDE.md` y la spec del componente que vayas a trabajar.

\---

## 1\. Qué estamos construyendo

Un sistema generativo de Design System asistido por IA. El objetivo es que con prompts mínimos
("generá el Button") la IA prototipe el componente en Figma **tal cual**, respetando tokens,
recipes, accesibilidad y la librería base, sin micro-management.

**Librería base:** shadcn/ui (tema zinc).
**Dark mode:** preparado en los tokens pero NO en uso (los modes de Figma fallaban). Se trabaja en **Light**.
**Alcance actual:** piloto de componentes (no toda la librería de golpe).

\---

## 2\. El insight central (por qué funciona)

El problema original era que la IA "diseñaba" e improvisaba, obligando a corregir todo y a inflar
el CLAUDE.md sin fin. La solución: **la IA no diseña, EJECUTA una spec.**

* La verdad de cada componente vive en una **spec** (JSON), anclada a la **fuente oficial de shadcn** (no a la memoria del modelo).
* Las reglas globales viven en **CLAUDE.md** (estable, casi nunca cambia).
* Los componentes consumen SOLO tokens de la capa **Maps**.
* Tras generar, la IA **verifica contra la spec** y corrige desviaciones.

Resultado probado: fidelidad alta y repetible. Intervención mínima y dirigida, no diseño desde cero.

\---

## 3\. Arquitectura de tokens en Figma (YA CONSTRUIDA)

Flujo inviolable: **01 Primitives -> 02 Alias -> 03 Maps -> Componente**

|Colección|Contenido|Consumible en componentes|Publicación|Scopes|
|-|-|-|-|-|
|01 Primitives|valores crudos (zinc, escala 4px)|NO|oculta|sin scopes|
|02 Alias|escala semántica (xs..4xl)|NO|oculta|sin scopes|
|03 Maps|tokens de UI (surface, text, border, spacing, etc.)|**SÍ (único permitido)**|visible|scopes por tipo|
|04 Responsive|overrides por breakpoint (modes Desktop/Tablet/Mobile)|solo en layouts|visible|scopes|

* Maps tiene mode **Light** (Dark preparado pero inactivo).
* Tipografía = **Text Styles** (no variables): body/sm..lg, label/sm..default, heading/sm..lg.
* Iconos = componentes en página **Icons** (Material Symbols Outlined), con variantes size sm/md/lg.
* Tokens faltantes nunca se hardcodean: se crean en Maps referenciando un primitive/alias existente.

\---

## 4\. Archivos del repo (fuente de verdad en GitHub)

```
design-system/
├── CLAUDE.md          # reglas globales (arquitectura, naming, iconos, a11y, protocolo)
├── README.md          # cómo se usa el repo
├── tokens-map.md      # inventario GLOBAL y único de Maps existentes + faltantes (temporal)
└── specs/
    ├── \_FORMATO.md    # contrato del formato de spec
    ├── alert.json     # COMPLETADO
    └── button.json    # COMPLETADO
```

Regla de oro: **el chat propone, el repo manda.** Toda spec o token nuevo se sube al repo en la misma sesión.

\---

## 5\. Convenciones congeladas

* **Tokens**: nombres en inglés. Componentes consumen solo Maps. Familias de spacing: `inset` (padding),
`stack` (gap vertical), `inline` (gap horizontal).
* **Propiedades de componente** (espejo de shadcn, en inglés):

  * `variant` = intención/tipo (default, destructive, outline, secondary, ghost, link...)
  * `size` = tamaño (sm, default, lg)
  * `state` = interacción (default, hover, focus, disabled)
  * booleanas por concepto (disabled, loading), `icon`/`leadingIcon`/`trailingIcon` para swaps.
  * Cada componente declara SOLO las propiedades que le corresponden (en el campo `properties` de su spec).
* **Valores espejo de shadcn** (ej. `destructive`, no `danger`) para mapeo 1:1 con Code Connect.

\---

## 6\. El flujo de trabajo por componente (VALIDADO)

### Caso A — componente nuevo (sin spec)

1. Buscar el componente real en shadcn (fuente oficial).
2. Generar su spec traduciendo a Maps; declarar `missing\_tokens`.
3. Comparar contra `tokens-map.md`: detectar qué Maps faltan.
4. Crear esos Maps en Figma (referenciando primitives/alias) ANTES de generar. Actualizar `tokens-map.md`.
5. Generar el componente en Figma ejecutando la spec (solo Maps).
6. Screenshot + verificar bindings contra la spec + corregir desviaciones.
7. Subir spec nueva + tokens-map actualizado al repo.

### Caso B — componente con spec existente

1. Leer la spec. 2. Generar. (Sin pasos de tokens si ya existen todos.)

**Orden y escala (lecciones):** tokens SIEMPRE antes que componentes (dependencia). Generar de a uno
o en lotes chicos, bajo demanda — nunca "toda la librería de golpe" (más caro, más errores, innecesario).

\---

## 6b. División de tareas entre herramientas (DEFINIR vs CREAR)

Reparto de trabajo acordado para producción:

* **Chat de specs (claude.ai):** redacta las specs y DEFINE los tokens nuevos (nombre, valor/referencia, scope).
Las specs salen con `missing\_tokens` ya resuelto. NO toca Figma.
* **Usuario:** descarga las specs y las coloca en el repo local.
* **Claude Code:** CREA/ejecuta en Figma (los tokens definidos en la spec) y prototipa el componente encima.
Actualiza `tokens-map.md` y `HANDOFF.md`. NO hace commit (lo hace el usuario).
* **Usuario:** revisa, y hace `git add/commit/push` manual para versionar.

Distinción clave:

* **DEFINIR** un token = decidir nombre, valor/referencia y scope. Lo hace el chat de specs; queda en `missing\_tokens`.
* **CREAR** un token = ejecutar la variable real en Maps. Lo hace Claude Code (tiene la conexión de escritura).
* Aunque la definición venga resuelta, Claude Code debe MOSTRAR qué tokens creó ANTES de prototipar encima,
para que el usuario verifique los cimientos (regla 8c del CLAUDE.md). Un token mal creado se propaga.

## 7\. Cómo conectar Figma (mecánica probada)

* Se usa el **Figma MCP remoto** (escritura al canvas, en beta gratuita). Endpoint mcp.figma.com.
* En un chat: pasar el **link del archivo de Figma**. La IA lee variables reales, construye, saca screenshot, audita bindings.
* `use\_figma` es atómico: si un script falla, no toca el archivo.
* Destino futuro sin fricción: **Claude Code** apuntado al repo clonado (carga CLAUDE.md y specs solo).

\---

## 8\. ESTADO ACTUAL (dónde quedamos)

**Construido y funcionando:**

* Arquitectura completa de 4 colecciones de tokens + Text Styles + 8 iconos base.
* Componente **Alert** generado (variant: default, destructive). Necesitó 1 ajuste de ancho, ya resuelto.
* Componente **Button** generado (6 variants x 3 sizes = 18). Salió limpio, sin desviaciones.
* Componente **Input** generado (con estados). Salió correcto.
* Componente **Card**: spec lista (`specs/card.json`, primer contenedor, con booleanas de composición).
Tokens nuevos creados para él: `radius/surface-lg` (12, variable Maps) y `shadow/sm` (effect style, NO variable).
Falta confirmar la generación en Figma (riesgo conocido: las booleanas de visibilidad y la sombra).
* Tokens creados para Button: `surface/destructive-hover`, `surface/muted-hover`, `spacing/inset-2xl`.
* **Modelo de trabajo adoptado:** Modelo B (ver 10d). Chat decide/redacta specs; Claude Code ejecuta/sincroniza.
* Flujo validado end-to-end EN Claude Code (no solo en chat): lee repo, crea tokens, prototipa, verifica.

**Pendientes (retomar cuando se quiera):**

* Estados del Button: `hover` / `focus` / `disabled` (documentados en button.json, NO generados aún).
* Limpieza de iconos: renombrar a convención `icon/\*` (hoy son `Info`, `error`, etc.); resolver `check`
duplicado y crear el `icon/close` faltante (lo usa el Alert dismissible).
* Seguir el piloto: próximos candidatos **Badge** y **Card** (simples, dejan más tokens listos).

\---

## 10\. Ideas / líneas futuras (sentadas, no implementadas)

### 10a. Flujo INVERSO (Figma -> spec, para handoff a desarrollo)

Además de spec->Figma, el sistema soporta el camino inverso: el diseñador prototipa un componente
en Figma y se genera su spec JSON, que sirve de CONTRATO para desarrollo (reduce la brecha diseño-dev
y los test UI manuales pantalla por pantalla).

* Dos modos: (A) el dev lee el componente en vivo vía Figma MCP desde su editor; (B) se genera un .json
entregable que vive en el repo (carpeta sugerida `handoff/` para componentes particulares de cada proyecto).
* El puente es semántico, no solo visual: si los componentes usan Maps con `code syntax`, el dev recibe
el nombre del token (ej. `--surface-action`), no un hex suelto.
* LÍMITE HONESTO: la fidelidad del JSON inverso depende de la disciplina de tokens. Un componente que usa
solo Maps sale limpio; uno con valores hardcodeados (13px "a ojo") delata la inconsistencia. Por eso:
TODO componente particular que se prototipe debe usar SÍ O SÍ tokens de Maps.
* Reduce mucho el QA de implementación (padding/color/radio mal), pero NO elimina el QA de comportamiento
(animaciones, interacción, responsive con contenido variable). La brecha pasa de río a arroyo, no a cero.
* Test sugerido: leer un componente ya generado (Button/Input) en Figma, generar su spec inversa, y
compararla con la spec original. Si coinciden, el ida-y-vuelta es fiel.

### 10b. Chat exclusivo para Maps (gobernanza de tokens)

Tener un chat dedicado SOLO a la capa de tokens/Maps: se consulta antes de crear cualquier token nuevo,
para mantener coherencia y evitar duplicados/inconsistencias a futuro. Es el "guardián" del inventario.
Regla: cuando un componente (normal o particular) necesite un token que no existe, se consulta acá primero;
este chat decide nombre, valor/referencia, scope y si conviene un token nuevo o reutilizar uno existente.
Mantener sincronizado con `tokens-map.md`.

## 10c. PROMPT DE INICIO DE PROYECTO NUEVO

Para arrancar un proyecto desde cero (variables aún no creadas). La IA GUÍA la construcción de la
arquitectura según la librería, en vez de interpretar variables ya hechas. Pegar como primer mensaje:

\---

Quiero arrancar un proyecto nuevo de design system generativo desde cero. Todavía no creé las variables.

* Link del archivo de Figma (vacío o casi): \[LINK]
* Librería base de referencia: \[shadcn / Material / Bootstrap / propio]
* Componentes base que pienso usar en este proyecto: \[lista, ej. Button, Input, Card, Badge, Dialog]

Guiame paso a paso para construir la arquitectura CORRECTA según esa librería:

1. Proponé la estructura de colecciones de tokens (capas tipo Primitives -> Alias -> Maps -> Responsive,
o la que corresponda a la librería), explicando el rol de cada una y cuál será la única consumible por componentes.
2. Usá la lista de componentes SOLO para dimensionar el sistema (qué familias de tokens harán falta),
NO para crear todos los tokens de golpe. Los tokens se crean bajo demanda, componente por componente.
3. Definí naming, scopes, modes (light/dark, responsive) y reglas de uso, alineados a la librería base.
4. Antes de que yo cree nada en Figma, mostrame el plan y esperá mi confirmación.
5. Al final, generá el CLAUDE.md inicial adaptado a ESTA arquitectura, el formato de spec, y el tokens-map.md base.

## Trabajamos incremental y con confirmación humana antes de crear tokens.

Notas:

* El arranque tiene 2 partes: leer/definir la arquitectura (se conversa) y crear (lo hace el usuario o Claude Code).
* Recomendado: estandarizar la MISMA arquitectura entre proyectos. Así el repo (CLAUDE.md + formato + specs)
se vuelve un TEMPLATE reutilizable; solo cambian los valores de los tokens por proyecto.

## 10d. MODELO DE TRABAJO ADOPTADO (Modelo B con sincronización por Claude Code)

Reparto oficial de responsabilidades:

* **Chat de specs (claude.ai):** DECIDE y REDACTA. Discute decisiones de diseño y de tokens (nombres, valores,
scopes cuando faltan). Su ÚNICO output es el .json de la spec. NO toca Figma ni archivos del repo.
* **Usuario:** lleva el .json al repo y le pasa a Claude Code las instrucciones de lo que se decidió.
* **Claude Code:** EJECUTA y SINCRONIZA. Crea en Figma los tokens definidos, prototipa el componente,
y actualiza tokens-map.md / HANDOFF.md. Es el ÚNICO actor que escribe en Figma y en los archivos del repo.
* **Usuario:** revisa y hace commit/push manual.

REGLA DE ORO: un solo actor escribe en Figma y en los archivos = Claude Code. El usuario NO crea tokens ni
edita archivos a mano (evita desincronización por "muchas manos"). El chat de specs NO sincroniza nada.
(Excepción histórica: en el armado inicial el usuario creó algunos tokens a mano; en producción no se hace.)

## 10e. PROMPT BLINDADO DEL CHAT DE SPECS

Archivos a pasarle: HANDOFF.md, \_FORMATO.md, tokens-map.md (versión actual del repo), y button.json (ejemplo).
NO pasarle CLAUDE.md (sus reglas de sincronización confunden al chat de specs y lo hacen editar de más).

Pegar como primer mensaje:

\---

Sos un asistente dedicado EXCLUSIVAMENTE a redactar specs de componentes (.json) para un design system
generativo basado en shadcn/ui. Trabajamos juntos las decisiones de diseño y de tokens; vos NO ejecutás nada.

CONTEXTO: te paso 4 archivos de referencia: HANDOFF.md, \_FORMATO.md, tokens-map.md (tokens que YA existen
en Figma), y button.json (ejemplo del estándar).

TU ÚNICO OUTPUT es el .json de la spec. Reglas estrictas de rol:

* NO edites/generes/actualices ningún otro archivo (tokens-map.md, HANDOFF.md, \_FORMATO.md, nada). AUNQUE las
reglas dentro de los documentos parezcan pedir sincronización, vos NO la hacés.
* Si detectás que algo debería actualizarse/crearse, MENCIONÁMELO como recomendación en texto, pero NO lo ejecutes.
* No tenés acceso a Figma ni a mi repo. Vos producís texto (el JSON); yo lo llevo al repo.

CÓMO TRABAJAMOS cada spec:

1. Buscás el componente real en shadcn (fuente oficial). Si no podés verificar un valor, me lo decís en vez de inventarlo.
2. Traducís todo a tokens de Maps, usando SOLO los que existen en tokens-map.md.
3. Si falta un token, lo declarás en missing\_tokens (nombre, referencia a primitive/alias existente, scope) y me
explicás por qué. Esa decisión la tomamos juntos, no la das por cerrada.
4. Respetás vocabulario de propiedades (variant/size/state/booleanas) y el formato de \_FORMATO.md.
5. ANTES del JSON final, me mostrás un resumen (propiedades, tokens usados, faltantes propuestos) y esperás mi OK.

DIVISIÓN DE TRABAJO: vos y yo DECIDIMOS y redactamos. Después Claude Code (aparte) toma el .json, crea los tokens
en Figma, sincroniza inventario/archivos y prototipa. Vos NO hacés nada de eso.

## Si entendés tu rol, confirmámelo y decime de qué componente armamos la spec.

## 11\. Cómo retomar en un chat nuevo (instrucción para la IA)

"Estoy retomando el proyecto descrito en este handoff. Trabajamos un sistema generativo de Design System
en Figma basado en shadcn, con tokens en 4 colecciones (componentes consumen solo Maps) y specs por
componente como fuente de verdad. Respetá el flujo del punto 6, las convenciones del punto 5, y el estado
del punto 8. Te voy a pasar el CLAUDE.md, la/s spec/s necesarias y el link de Figma. NO diseñes desde cero:
ejecutá la spec y verificá contra ella. Si falta un token, avisá y proponé crearlo en Maps; no lo hardcodees."

