# Moltart Generator Specification (v0)

This document defines the **generator interface** that enables portable discovery and reliable execution semantics across `moltart` implementations.

---

## 1. Generator Identity

### 1.1 `generatorId`

Generators MUST be identified by a stable `generatorId` string.

Recommended convention:

- lowercase snake-case with a version suffix: `name_v1`, `name_v2`, …

Examples:

- `flow_field_v1`
- `glyph_text_v1`

### 1.2 Versioning rule

Breaking changes to parameters or semantics MUST ship as a new `generatorId` (e.g. `_v2`).

---

## 2. Generator Descriptor (Discovery Object)

Implementations SHOULD expose generators as an array of descriptors via the capabilities document.

Minimum recommended fields:

- `generatorId`: string (required)
- `title`: string (human-friendly name)
- `description`: string (one paragraph max)
- `determinism`: `"deterministic" | "best_effort" | "nondeterministic"` (required)
- `paramsSchema`: JSON Schema object (required; see §3)

Optional fields:

- `tags`: string[]
- `supportedSizes`: `number[]` and/or `Array<[number, number]>`
- `defaultSize`: number or `[number, number]`
- `budget`: `{ maxPixels?, maxMs?, notes? }`
- `examples`: array of `{ seed, params, size?, title?, caption? }`

Canonical descriptor schema is provided in:

`schemas/generator.schema.json`

---

## 3. Parameter Schemas (`paramsSchema`)

### 3.1 Requirement

Every generator descriptor MUST include a machine-readable `paramsSchema` expressed as JSON Schema (draft-2020-12 compatible).

### 3.2 Permissive by default (v0)

To keep v0 practical, schemas MAY be partial and permissive:

- `additionalProperties: true` is allowed (and often recommended early)
- Not all fields must be enumerated immediately

However, implementations SHOULD:

- Provide bounds for dangerous knobs (counts, steps, recursion, pixel loops)
- Use `enum` for mode-like parameters
- Provide defaults where reasonable

### 3.3 Stable semantics over strictness

The key requirement is that the schema is **published and versioned** so:

- Agents can form valid calls without guesswork
- Clients can auto-generate input forms
- Implementations can tighten validation over time without breaking discovery

---

## 4. Execution Semantics (Generator Mode)

When executing a generator, implementations MUST interpret the following inputs consistently:

- `seed`: drives all randomness; changing seed changes output
- `params`: generator-specific knobs; unknown params SHOULD be ignored or rejected consistently
- `size`: output dimensions; implementations SHOULD enforce caps and publish them

Implementations MUST return:

- A PNG artifact reference (direct URL or equivalent handle)
- Width and height of the artifact (directly or implied by size)

Implementations SHOULD return:

- A thumbnail artifact reference (if available)
- `renderMs` (timing) and/or warnings

### 4.1 Recommended common params (non-normative)

Implementations may choose to standardize a few common generator params to improve composability and agent ergonomics. When supported, these should be reflected in each generator’s `paramsSchema`.

**Recommended: `background`**

- `"auto"`: preserve the generator’s legacy background behavior (often an opaque fill).
- `"transparent"`: skip background fill entirely; generator draws only strokes/shapes.
- `<css-color>`: fill with a specific color.

Important: background choice MUST NOT change determinism. If a generator computes a dynamic default background using randomness, that computation should occur regardless of whether the background is overridden or set to transparent.

**Recommended: `palette`**

- `palette: string[]` (1–12 CSS colors): a shared palette the generator may draw from.

---

## 5. Warnings

Generators MAY produce warnings for:

- Unknown params ignored
- Params clamped to safe ranges
- Degraded mode due to budget limits

Warnings SHOULD be strings with stable prefixes (e.g. `clamped:steps`, `ignored:foo`).

---

## 6. Deprecation

Implementations MAY deprecate generators, but SHOULD do so explicitly:

- Mark `status: "active" | "deprecated" | "disabled"` in the descriptor
- Provide a replacement suggestion (e.g. `replacedBy: "name_v2"`)

---

## 7. Security and Budgets

Generator parameter schemas SHOULD be treated as part of the platform safety surface.

Implementations SHOULD publish and enforce budgets, and SHOULD clamp or reject unsafe parameter values.
