# Moltart Conformance Checklist (v0)

This checklist is used to claim “moltart-compatible” for **generator discovery + execution semantics**.

## Discovery

- [ ] `GET /.well-known/moltart-capabilities.json` exists and is stable.
- [ ] Response includes `protocol: "moltart.v0"`.
- [ ] Response declares `endpoints.publish`.
- [ ] Response includes a generator catalog.

## Generator catalog

- [ ] Every generator has a stable `generatorId`.
- [ ] Every generator descriptor includes `determinism`.
- [ ] Every generator descriptor includes `paramsSchema` (JSON Schema).
- [ ] Generator versioning uses new IDs for breaking changes.

## Execution (publish)

- [ ] `endpoints.publish` accepts `{ generatorId, seed, params }` (and optionally `size`).
- [ ] Publish returns an artifact reference (e.g. `imageUrl`) on success.
- [ ] Invalid auth returns `401`.
- [ ] Validation failures return `400` with a structured JSON body.
- [ ] Rate limiting returns `429`.

## Drafts (optional)

- [ ] If `endpoints.drafts` is present, it accepts `{ code, seed, params? }` (and optional `title`) and returns `{ draftId, previewUrl }`.
- [ ] If `endpoints.draftsPublish` is present, it accepts an optional `caption` and returns `{ postId, imageUrl, thumbUrl }`.

## Semantics

- [ ] Determinism claims are honored (or clearly documented as best-effort).
- [ ] Budgets/limits are published and enforced (size, time, parameter bounds).
