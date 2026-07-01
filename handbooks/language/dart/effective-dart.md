# Effective Dart — Standards for this Project

ENFORCEMENT: advisory

## Naming

- Classes, enums, typedefs, type params: `UpperCamelCase`
- Libraries, packages, directories, source files: `lowercase_with_underscores`
- Variables, parameters, named params, getters/setters: `lowerCamelCase`
- Constants: `lowerCamelCase` (not SCREAMING_SNAKE — Dart convention)
- Private members: `_prefixedWithUnderscore`

## Types

- Prefer type annotations on public APIs; rely on inference inside function bodies
- Avoid `dynamic` unless interoperating with untyped APIs — annotate the boundary
- Use `Object?` over `dynamic` when the value truly can be anything nullable
- Never use `var` for top-level or class-level variables where the type is non-obvious

## Domain layer contract (HARD RULE)

The `lib/domain/` package must have **zero Flutter or DB imports**. Any PR that adds:
- `import 'package:flutter/...`
- `import 'package:drift/...`
- `import 'package:sqflite/...`

...to a file under `lib/domain/` is a blocking finding.

## Error handling

- Prefer typed exceptions over raw `Exception` or `Error`
- Use `sealed` classes for domain error variants (exhaustive switch)
- Never swallow exceptions with an empty catch block
- Log the original error at the boundary; rethrow typed domain errors inward

## Async

- Prefer `async`/`await` over raw `Future.then` chains
- Always `await` or `unawaited()` a Future — never fire-and-forget without annotation
- Use `Stream` for reactive data (Drift returns streams — use them)

## Testing

- Domain entities and use-cases: plain `dart test` (no Flutter test host needed)
- Repositories: test against an in-memory Drift database (`NativeDatabase.memory()`)
- Providers (Riverpod): use `ProviderContainer` directly, no widget needed for pure logic tests
- Target: **>80% coverage on `lib/domain/` and `lib/application/`**
