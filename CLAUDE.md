# Design System — Reglas Globales

> Fuente de verdad para generación de componentes en Figma.
> Base library: **shadcn/ui** (tema zinc). NO copiar shadcn literal: traducir a estos tokens.
> Este archivo contiene SOLO reglas globales. Cada componente tiene su spec en `/specs/<componente>.json`.

---

## 1. Flujo de tokens (INVIOLABLE)

```
01 Primitives  →  02 Alias  →  03 Maps  →  Componente
```

- **Los componentes consumen ÚNICAMENTE tokens de `03 Maps`.**
- NUNCA usar Primitives, Alias, ni valores hardcodeados (hex, px) en un componente.
- Si un valor que necesitás no existe como Map, NO lo hardcodees: avisá que falta el token y proponé crearlo en Maps.
- Primitives y Alias están ocultos de publicación y sin scopes: no deberían aparecer en los pickers. Si aparecen, es un error.

## 2. Colecciones disponibles

| Colección | Qué contiene | ¿Consumible en componentes? |
|---|---|---|
| 01 Primitives | valores crudos (zinc, escala 4px) | NO |
| 02 Alias | escala semántica (xs…4xl) | NO |
| 03 Maps | tokens de UI (surface, text, border, spacing, etc.) | **SÍ — único permitido** |
| 04 Responsive | overrides por breakpoint (page-padding, etc.) | solo en layouts, no en componentes |

## 3. Tokens de Maps disponibles

**Color** (cambian con mode Light/Dark automáticamente):
- `surface/*` → page, card, muted, action, action-hover, destructive
- `text/*` → default, muted, on-action, on-destructive, destructive
- `border/*` → default, subtle, focus

**Numéricos** (iguales en Light/Dark):
- `spacing/inset-*` → padding interno (xs=4, sm=8, md=12, lg=16, xl=24)
- `spacing/stack-*` → gap VERTICAL (xs=4, sm=8, md=12, lg=16)
- `spacing/inline-*` → gap HORIZONTAL (xs=4, sm=8, md=12)
- `control-height/*` → sm=32, md=36, lg=40
- `icon-size/*` → sm=16, md=20, lg=24
- `radius/control` (6), `radius/surface` (8), `radius/pill` (9999)

**Regla de spacing:** inset = padding, stack = gap en columna, inline = gap en fila. No mezclar.

## 4. Tipografía — usar Text Styles (no variables)

`body/sm`, `body/default`, `body/lg`, `label/sm`, `label/default`, `heading/sm`, `heading/md`, `heading/lg`.
Default de UI es 14px (body/default). Botones/badges/inputs usan `label/*` (weight 500).

## 5. Iconos

- Fuente única: componentes de la página **Icons** (Google Material Symbols Outlined).
- Consumir SIEMPRE `icon/*` como instancia. NUNCA dibujar vectores propios, importar otras librerías, ni usar emojis.
- Tamaño vía propiedad `size` (sm/md/lg) de la instancia, ligada a `icon-size/*`.
- Si falta un icono, avisá y proponé agregarlo a la página Icons; no lo inventes.

## 5b. Naming de propiedades de componente (variant properties)

Los nombres de propiedad y sus valores se ajustan SIEMPRE a la librería base (shadcn/ui), igual que los tokens. En inglés, minúscula. NO inventar nombres propios ni traducir.

Vocabulario fijo:
- `variant` → intención/tipo del componente (ej. default, destructive, outline, ghost, secondary). Es la palabra que shadcn usa en código.
- `size` → tamaño (sm, md, lg).
- `state` → estado de interacción (default, hover, focus, disabled). NO confundir con variant.
- Booleanas por concepto cuando aplique: `disabled`, `loading`, `dismissible` (true/false).
- Swap de icono: `icon`, o `leadingIcon` / `trailingIcon`.

Reglas:
- `variant` (tipo) y `state` (interacción) son ejes DISTINTOS. Un componente puede tener ambos a la vez (ej. Button: variant + size + state).
- Cada componente declara SOLO las propiedades que le corresponden (lo define su spec, campo `properties`). No todos los componentes tienen todas: un Alert no lleva `state` ni `disabled`; un Button sí.
- Los valores son espejo de shadcn (ej. `destructive`, no `danger`). Esto permite mapeo 1:1 con Code Connect sin tabla de traducción.

## 6. Construcción en Figma

- Todo componente usa **Auto Layout**.
- Padding del auto layout → `spacing/inset-*`. Gap → `spacing/stack-*` o `inline-*` según dirección.
- Width/height de controles → `control-height/*`. Nunca altura fija hardcodeada.
- Radius → `radius/*`.
- Fills → `surface/*`. Texto → `text/*`. Strokes → `border/*`.
- Las variantes (variant properties) y sus nombres vienen definidos en la spec de cada componente.

## 7. Accesibilidad (aplica a todos)

- Contraste mínimo AA (4.5:1 texto normal) en Light y Dark. Los pares text/surface de Maps ya lo cumplen; respetarlos.
- Estado de foco visible usando `border/focus`.
- Touch target mínimo 32px de alto en controles interactivos.
- Texto nunca por debajo de 12px.

## 8. Estados (convención global)

Salvo que la spec diga otra cosa, los componentes interactivos definen estos estados como variant property `state`:
- `default`, `hover`, `focus`, `disabled`.
- `disabled` → opacidad 50% (layer opacity), sin cambiar tokens.
- `hover` en superficies de acción → usar `surface/action-hover`.

## 8b. Configuración del proyecto (para Claude Code)

- **Figma fileKey:** el usuario provee el link/fileKey de su archivo de Figma al inicio de la sesión. Usar ese.
- **Antes de inspeccionar Figma:** leer `tokens-map.md` para saber qué Maps ya existen. Solo consultar Figma para confirmar o para datos que el inventario no tenga.
- **Modelo:** Sonnet alcanza para ejecutar specs ya definidas. Opus se reserva para traducir componentes nuevos/complejos de shadcn a spec, o resolver problemas no triviales.
- **Sincronización:** cuando se crean tokens o specs, actualizar `tokens-map.md` y/o `HANDOFF.md` en la misma corrida. El repo manda; toda mejora se commitea.

## 8c. Regla de creación de tokens (CONTROL HUMANO)

Antes de crear cualquier token nuevo en Maps:
1. Detectar los faltantes comparando la spec contra `tokens-map.md`.
2. MOSTRAR al usuario la lista (nombre, valor/referencia propuesta, scope) y ESPERAR su confirmación.
3. Recién con el OK, crear los tokens (referenciando primitives/alias existentes), aplicar scopes, y actualizar `tokens-map.md`.

NUNCA crear tokens en automático sin confirmación: un token mal definido se propaga a todos los componentes que lo usen.

## 9. Cómo generar un componente (protocolo)

1. Leer `/specs/<componente>.json` completo.
2. Construir respetando EXACTAMENTE cada binding de token de la spec.
3. No tomar decisiones de diseño no especificadas: si la spec no lo dice y no hay regla global, PREGUNTAR antes de inventar.
4. Al terminar, verificar el componente contra la spec y listar cualquier desviación.
