---
name: effect-guide
description: Comprehensive guide for using Effect-TS library correctly. Use when writing Effect code, including Schema patterns, error handling, service/layer patterns, @effect/sql database operations, or @effect/vitest testing. Provides critical rules to avoid common mistakes with Effect's type system, and best practices for functional TypeScript.
---

# Effect Guide

Guide for using Effect-TS correctly, covering critical rules, patterns, and best practices.

## Quick Reference

| Topic | Reference File |
|-------|----------------|
| Critical rules, Schema patterns, error handling, services | [references/best-practices.md](references/best-practices.md) |
| Layer memoization, Layer.fresh, composition | [references/layers.md](references/layers.md) |
| @effect/sql, SqlSchema, repositories | [references/sql.md](references/sql.md) |
| @effect/vitest, testcontainers, property tests | [references/testing.md](references/testing.md) |

## Critical Rules Summary

1. **NEVER use `any` or type casts** - use `Schema.make()`, `Schema.decodeUnknown`, `identity`
2. **NEVER use global `Error`** - use `Schema.TaggedError` for all domain errors
3. **NEVER use `catchAllCause`** - it catches defects (bugs); use `catchAll` or `mapError`
4. **NEVER use `*FromSelf` schemas** - use standard variants (`Schema.Option`, not `OptionFromSelf`)
5. **NEVER use Sync variants** - use `Schema.decodeUnknown` not `decodeUnknownSync`
6. **Use `.make()` not `new`** - all Schema classes have `.make()` constructor
7. **Use Chunk not Array** - for structural equality in domain models

## When to Read Reference Files

- **Writing any Effect code** → Read [best-practices.md](references/best-practices.md)
- **Composing layers or understanding memoization** → Read [layers.md](references/layers.md)
- **Database operations with @effect/sql** → Read [sql.md](references/sql.md)
- **Writing tests with @effect/vitest** → Read [testing.md](references/testing.md)
