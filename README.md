# Design System Generativo asistido por IA

> Un sistema para generar componentes de UI en Figma a partir de "recetas" versionadas,
> manteniendo consistencia total con una librería base (shadcn/ui) y con los design tokens.
> La IA no improvisa diseño: **ejecuta una especificación**.

---

## El problema que resuelve

Generar componentes con IA "a mano" (prompt suelto → la IA diseña) falla de forma predecible:
la IA improvisa medidas y colores distintos cada vez, hay que corregir todo, y se termina
acumulando instrucciones sin fin. El resultado es inconsistente y más lento que prototipar a mano.

Este proyecto invierte el enfoque: **la verdad de cada componente vive en una especificación
(spec) versionada**, anclada a la fuente oficial de la librería y a los design tokens del sistema.
La IA lee esa spec y la construye en Figma de forma fiel y repetible. El resultado es consistente
porque no depende del "criterio" del modelo en cada corrida, sino de un contrato escrito.

## Cómo funciona (en una imagen)

```
Librería base (shadcn/ui)
        |  traducción a tokens semánticos
        v
Arquitectura de tokens en Figma:  Primitives -> Alias -> Maps -> Componente
        |  (los componentes consumen SOLO la capa Maps)
        v
Spec por componente (.json)  -->  IA ejecuta en Figma  -->  Verificación contra la spec
        |
        v
Repositorio (fuente de verdad versionada)
```

## Conceptos clave

- **Tokens en capas.** Los valores crudos (Primitives) se vuelven semánticos (Alias) y luego
  contextuales (Maps). Los componentes consumen únicamente Maps, nunca valores hardcodeados.
  Un cambio en un token se propaga a todo el sistema.
- **Spec como contrato.** Cada componente es un `.json` que declara su anatomía, propiedades,
  bindings de tokens, accesibilidad y, si necesita tokens que aún no existen, los declara en
  `missing_tokens` en vez de inventar valores.
- **La IA ejecuta, no diseña.** Lee la spec, construye en Figma, saca un screenshot y verifica
  que cada propiedad esté ligada al token correcto. La intervención humana es mínima y dirigida.
- **Fuente de verdad fuera del chat.** Todo vive en este repositorio. Las conversaciones con la IA
  son desechables; el repo es lo que perdura y versiona.

## Estructura del repositorio

```
.
|- README.md          Este archivo.
|- CLAUDE.md          Reglas globales del sistema (arquitectura, naming, accesibilidad, protocolo).
|                     Lo carga automáticamente Claude Code.
|- HANDOFF.md         Documento de continuidad: estado del proyecto, flujos de trabajo y prompts
|                     reutilizables. Permite retomar el proyecto en cualquier sesión nueva.
|- tokens-map.md      Inventario vivo de los tokens (Maps) y estilos que existen hoy en Figma.
\- specs/
   |- _FORMATO.md     Contrato del formato de spec (el molde de todo .json de componente).
   |- alert.json      Spec: mensaje semántico (variantes default/destructive).
   |- button.json     Spec: control interactivo (6 variantes x 3 tamaños).
   |- input.json      Spec: campo de formulario (con estados).
   \- card.json       Spec: superficie contenedora (primer componente compuesto).
```

## Cómo probarlo

### Requisitos
- Una cuenta de **Figma** con un archivo donde tengas (o vayas a crear) la arquitectura de tokens.
- **Claude Code** (requiere un plan pago de Claude) con el **MCP de Figma** conectado.
  Alternativamente, cualquier herramienta con acceso de escritura al MCP de Figma.

### Opción A — Generar un componente que ya tiene spec (camino corto)
1. Cloná este repositorio y abrí una terminal dentro de la carpeta.
2. Lanzá `claude` (Claude Code carga el `CLAUDE.md` automáticamente).
3. Verificá que Figma responde con `/mcp`.
4. Pedí, por ejemplo:
   > Generá el componente Button en Figma siguiendo `specs/button.json`.
   > Consumí solo tokens de la colección Maps. Al terminar, verificá los bindings contra la spec.
5. Revisá el resultado en Figma: el componente debe construirse consumiendo los tokens declarados.

> Nota: tus variables de Figma deben llamarse igual que las que esperan las specs
> (ver `tokens-map.md`). Los nombres son lo que hace portable el sistema.

### Opción B — Crear un componente nuevo (camino completo)
1. Se redacta su spec traduciendo el componente desde shadcn a los tokens (ver `HANDOFF.md`).
2. Si la spec declara `missing_tokens`, se crean esos tokens en Figma (con revisión humana) antes
   de generar. El inventario `tokens-map.md` se actualiza.
3. Se genera el componente y se verifica contra la spec.

El flujo de trabajo completo, los roles de cada herramienta y los prompts reutilizables están
documentados en `HANDOFF.md`.

## Alcance y límites (honestidad)

- Esto es un **piloto** que valida la mecánica, no una librería completa. Cubre componentes base.
- La fidelidad es alta y repetible, pero **no es 100% sin intervención**: componentes complejos
  (contenedores con composición condicional, estados interactivos) pueden requerir ajustes puntuales.
- La calidad del resultado depende de la **disciplina de tokens**: un componente que usa solo Maps
  sale limpio; uno con valores hardcodeados rompe la consistencia.
- La librería base de referencia es **shadcn/ui**. El sistema no copia shadcn literal: lo **traduce**
  a reglas y tokens semánticos reutilizables. El mismo enfoque sirve para otras librerías.

## Stack

- **Figma** (variables/tokens en 4 colecciones + Text Styles + Effect Styles + componentes).
- **shadcn/ui** como librería base de referencia.
- **Claude Code + MCP de Figma** como motor de generación.
- **Git/GitHub** como fuente de verdad versionada.
