# Effect Layers: Deep Dive

Essential semantics of Effect's Layer system, focusing on memoization, laziness, and correct usage patterns.

## Core Principle: Identity-Based Memoization

**Layers are lazy by default and memoized by object identity (reference equality), not by type or value.**

When building a layer composition, Effect maintains a `MemoMap` keyed by layer object reference:
- First encounter: evaluate the layer, store in MemoMap
- Subsequent encounters: return cached result

```typescript
// SAME layer reference used twice = ONE instance
const layer = Layer.effect(MyTag, Effect.succeed("value"))
const composed = layer.pipe(Layer.merge(layer)) // Single instance!

// DIFFERENT layer references = TWO instances
const layer1 = Layer.effect(MyTag, Effect.succeed("value"))
const layer2 = Layer.effect(MyTag, Effect.succeed("value"))
const composed2 = layer1.pipe(Layer.merge(layer2)) // Two instances!
```

## Layer.fresh: Escaping Memoization

`Layer.fresh(layer)` wraps a layer to escape memoization entirely:

```typescript
const layer = Layer.effect(MyTag, createResource())

// Without fresh: single instance
const single = layer.pipe(Layer.merge(layer))

// With fresh: two separate instances
const two = layer.pipe(Layer.merge(Layer.fresh(layer)))
```

**When is Layer.fresh actually needed?**

Only when you:
1. Have the **same layer reference** appearing multiple times
2. **Want separate instances** instead of sharing

```typescript
// CORRECT use of Layer.fresh
const baseLayer = makeExpensiveLayer()
const needsBothInstances = Layer.merge(
  baseLayer,                  // First instance
  Layer.fresh(baseLayer)      // Second (separate) instance
)
```

## Factory Functions Don't Need Fresh

A factory function that builds a new layer on each call already returns different layer objects:

```typescript
// Each call returns a NEW layer object - no memoization between them!
const createTestLayer = (config: Config) => {
  return Layer.effect(MyService, makeService(config))
}

// These are DIFFERENT objects - memoization does NOT occur between them
const layer1 = createTestLayer({ option: true })
const layer2 = createTestLayer({ option: false })

// Layer.fresh is UNNECESSARY here!
// WRONG: return Layer.fresh(composed)
// CORRECT: return composed
```

**Key insight**: Memoization is per-build, not global. Each time you build a layer composition, a fresh MemoMap is created.

## Layer.memoize: Explicit Lazy Memoization

`Layer.memoize` creates a layer that is lazily built and explicitly memoized within a scope:

```typescript
const memoized = yield* Layer.memoize(expensiveLayer)

// First use: builds the layer
const ctx1 = yield* Effect.provide(myEffect, memoized)

// Second use: returns cached result (within same scope)
const ctx2 = yield* Effect.provide(myEffect, memoized)
```

## ManagedRuntime: Sharing Layers Across Runs

`ManagedRuntime` stores a MemoMap that persists across all effects:

```typescript
const runtime = ManagedRuntime.make(myLayers)

// Both runs share the same memoization cache
await runtime.runPromise(effect1) // Builds layers
await runtime.runPromise(effect2) // Reuses cached layers
```

## Summary Table

| Scenario | Memoization Behavior |
|----------|---------------------|
| Same layer ref in composition | Shared (single instance) |
| Different layer refs (same type) | Separate instances |
| `Layer.fresh(layer)` | Escapes memoization, always new |
| Factory function calls | Different objects = no sharing |
| Same `it.layer()` block | Shared within block |
| Different `it.layer()` blocks | Separate (new MemoMap each time) |
| Same `ManagedRuntime` | Shared across runs |

## Rules of Thumb

1. **Don't use `Layer.fresh` with factory functions** - they already return new objects
2. **Use `Layer.fresh` only when** you have the same layer reference twice and want separate instances
3. **For shared test infrastructure** (databases, containers), use vitest `globalSetup`
4. **For per-test-group isolation**, rely on `it.layer()` creating new MemoMaps
5. **Memoization is per-build** - different builds never share, same build always shares (unless Fresh)
