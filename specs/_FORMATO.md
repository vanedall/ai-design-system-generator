# Formato de Spec de Componente

Una spec es la **fuente de verdad** de UN componente. Claude NO diseña: ejecuta esto.
Un archivo `.json` por componente. Reglas globales viven en `/CLAUDE.md`, no acá.

## Principios

1. **Todo binding de color/spacing/radius/size apunta a un token de `03 Maps`.** Nunca un hex, nunca un px, nunca un Primitive.
2. Tipografía se referencia por **Text Style** (ej. `body/default`), no por tamaño.
3. Iconos se referencian por componente `icon/*`.
4. Si el componente necesita un valor que NO existe como Map, va en `missing_tokens` con una propuesta. NO se inventa ni se hardcodea.
5. Traducción FIEL de la librería base: si shadcn solo tiene 2 variantes, la spec tiene 2. No agregamos variantes que la librería no tiene (eso sería una extensión consciente, no una traducción).

## Esqueleto

```json
{
  "component": "NombreComponente",
  "base": "shadcn/ui",
  "archetype": "una frase: qué ES este componente",
  "properties": {
    "<nombre>": { "type": "variant | size | state | boolean | icon-swap", "values": ["...", "..."] }
  },
  "anatomy": {
    "<parte>": {
      "element": "frame | text | icon-instance",
      "visibleWhen": "<propiedad> == <valor>   (opcional: muestra/oculta la parte segun una propiedad booleana)",
      "layout": { "direction": "...", "gap": "<token>", "padding": {...}, "align": "...", "width": "fill|hug" },
      "fill": "<surface token>",
      "stroke": { "color": "<border token>", "width": 1 },
      "radius": "<radius token>",
      "effect": "<effect style, ej. shadow/sm>",
      "textStyle": "<text style>",
      "textColor": "<text token>",
      "iconComponent": "icon/...",
      "iconSize": "<icon-size token>"
    }
  },
  "composition": {
    "notes": "solo para componentes contenedores con partes opcionales",
    "rules": [
      "regla de presencia/oculto gatillada por las propiedades booleanas, ej: header=false oculta el grupo header y su gap"
    ]
  },
  "variants": {
    "<propertyName>": ["valor1", "valor2"],
    "overrides": {
      "<valor>": { "<parte>.<prop>": "<token>" }
    }
  },
  "states": ["default", "..."],
  "accessibility": ["regla", "regla"],
  "missing_tokens": [
    { "token": "border/destructive", "proposal": "ref red/500 (light) / red/900 (dark), scope Stroke", "fallback": "border/default" }
  ],
  "notes": "aclaraciones o extensiones opcionales"
}
```

## Campos

- **archetype**: la "recipe". Le dice a Claude qué es el componente conceptualmente.
- **anatomy**: las partes y sus bindings. El orden de las partes = orden visual.
- **variants.overrides**: solo lo que CAMBIA por variante. Lo que no aparece, se hereda de anatomy.
- **missing_tokens**: el mecanismo anti-hardcode. Si esto tiene items, hay que resolverlos (crear el Map) antes o durante la generación.

## Campos para componentes contenedores (ej. Card, Dialog, Popover)

Estos campos son opcionales y solo aparecen en componentes con superficie, efectos o partes que se muestran/ocultan:

- **effect**: referencia a un **Effect Style** de Figma (ej. `shadow/sm`), NO a una variable de Maps. Las sombras
  viven como estilos, igual que la tipografía (`textStyle`). Se referencian por nombre del estilo.
- **visibleWhen**: condición de visibilidad de una parte, atada a una propiedad booleana del componente
  (ej. `"visibleWhen": "footer == true"`). La parte solo se muestra cuando la condición se cumple.
- **composition**: bloque con las reglas de presencia/oculto entre partes, gatilladas por las propiedades
  booleanas. Documenta cómo se reacomoda el layout cuando una sección está ausente (ej. "header=false oculta
  el grupo header y su gap"). Solo para contenedores con partes opcionales.

## Nota sobre tipos de asset

No todo lo que consume un componente es una variable de Maps. Hay tres tipos de asset:
- **Variables de Maps** → color, spacing, radius, size (fill, stroke, radius, gap, padding, width/height).
- **Text Styles** → tipografía compuesta (campo `textStyle`).
- **Effect Styles** → sombras/efectos (campo `effect`).
Los tres se referencian por nombre. Solo el primero vive en la colección Maps; los otros dos viven como estilos.
