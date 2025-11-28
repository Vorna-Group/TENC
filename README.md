# TENC — Token Efficient Nested Codec

Public documentation and specification of the TENC format. This repository contains documentation only (specification and examples), not an implementation.

- Released by: Vorna Group LLC
- Conceived and implemented by: Felix Orlov and Nick Medved

## Quick links

- Specification: `spec/specification.md`
- Examples: `spec/examples.md`
- Contributing: `CONTRIBUTING.md`
- Security: `SECURITY.md`
- License: `LICENSE`

## What is TENC?

TENC is a compact, reversible, DOM-like markup for LLM prompt compression and lightweight document representation. It is designed for deterministic parsing, minimal token overhead, and lossless HTML round-trip at the DOM/tree level.

TENC represents documents as a single-line, DOM-like structure:

```tenc
<tag[.class...][#id][@primary](attr=value ... ) TEXT <child> ... >
```

- `.class` → `class="…"`, `#id` → `id="…"`, `(k=v …)` → attributes
- `@primary` encodes a tag’s primary attribute (e.g., `<a@…>` ↔ `href`)
- For tags without a natural primary, TENC 0.3 uses `data-primary`

TENC 0.2 introduced dictionary definitions and references to compress repeated values. TENC 0.3 formalizes the `@primary` map and provides a reliable `data-primary` fallback.

## Status and Versioning

```text
TENC Version: 0.3
Status: Pre-final
Compatibility: Strict superset of Mini-DOM and TENC 0.1 / 0.2 syntax.
```

## Scope

This repository is a specification and documentation resource. It is intended for implementers and users of the TENC format. For code and tooling, refer to the respective implementation repositories (if any).

## Contributing

Issues and PRs are welcome. Please see `CONTRIBUTING.md` for guidelines.

## Security

Please follow the private disclosure process in `SECURITY.md`. Do not post sensitive details in public issues.

## License

MIT — see `LICENSE`.
*** End Patch

