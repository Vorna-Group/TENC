# Contributing to TENC Documentation

Thank you for your interest in contributing! This repository contains the public documentation and specification of the TENC format. It is not an implementation repository.

## Ways to contribute

- Propose clarifications to the specification
- Improve examples or add new illustrative cases
- Fix typos, grammar, or formatting
- Suggest structure improvements for readability

## Ground rules

- Keep the specification deterministic and unambiguous
- Preserve backwards compatibility unless a breaking change is explicitly proposed and justified
- Clearly separate normative (MUST/SHOULD) language from non-normative guidance
- Do not introduce framework- or product-specific semantics into the core spec

## Workflow

1. Open an issue describing the change (context, rationale, before/after)
2. Create a branch and submit a PR linked to the issue
3. Keep PRs small and focused; one topic per PR
4. Update `spec/examples.md` when adding or modifying normative rules

## Style

- Markdown, wrapped reasonably for readability
- Use code fences with language hints (`tenc`, `html`, `ebnf`) where applicable
- Prefer explicit terminology; define new terms before use
- Use RFC 2119 keywords (MUST/SHOULD/MAY) where normative

## Review

- Maintainers will review for clarity, correctness, and consistency
- Changes to normative sections may require broader discussion

## Attribution

This documentation is released by Vorna Group LLC.
Conceived and implemented by Felix Orlov and Nick Medved.

By contributing, you agree that your contributions will be licensed under the MIT License (see `LICENSE`).


