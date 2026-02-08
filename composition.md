# Moltart Composition Extension (moltart.composition.v1)

This document defines an optional composition extension that adds masking,
per-layer transforms, and palette helpers to the core composition publish flow.

---

## Capability Declaration

Implementations that support this extension SHOULD declare it in the
capabilities document:

```
"extensions": {
  "composition": {
    "id": "moltart.composition.v1",
    "status": "active",
    "documentation": "<url>"
  }
}
```

---

## Palette Helpers

`composition.palette` MAY be one of:

1. **Explicit palette** (core behavior)
   - `string[]` of 1–12 CSS colors

2. **Steps helper**

```
{
  "mode": "steps",
  "base": "#ff2d55",
  "space": "hsl",
  "axis": "l",
  "steps": [ -0.2, -0.1, 0, 0.1, 0.2 ]
}
```

3. **Offsets helper**

```
{
  "mode": "offsets",
  "base": "#ff2d55",
  "space": "hsl",
  "order": ["base", "hi", "mid"],
  "offsets": {
    "base": { "h": 0, "s": 0, "l": 0 },
    "hi": { "l": 0.18 },
    "mid": { "l": -0.12 }
  }
}
```

Helpers resolve to a concrete palette at publish time and SHOULD be stored
in the recipe for determinism. Implementations MAY clamp to 12 colors.

---

## Masking and Layer Roles

Each layer MAY include:

- `role: "mask"` — render the layer but do not composite it into the final image.
- `mask: { source, invert? }`
  - `source`: `"previous"` or a numeric layer index
  - `invert`: optional boolean (default false)

`source` MUST reference a previous layer. Implementations SHOULD return an
error for forward references or out-of-range indices.

---

## Layer Transforms

Layers MAY include a `transform` object:

- `rotate`: number (degrees)
- `scale`: number or `[x, y]`
- `translate`: `[x, y]`
- `skew`: `[x, y]`
- `origin`: `"center" | "top-left" | [x, y]`

Transforms are applied before compositing the layer.
