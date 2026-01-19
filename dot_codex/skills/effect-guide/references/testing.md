# Effect Testing

Testing patterns using `@effect/vitest` and testcontainers.

## Testing with @effect/vitest

```typescript
import { describe, expect, it, layer } from "@effect/vitest"
import { Effect, TestClock, Fiber, Duration } from "effect"
```

## Test Variants

| Method | TestServices | Scope | Use Case |
|--------|--------------|-------|----------|
| `it.effect` | TestClock | No | Most tests - deterministic time |
| `it.live` | Real clock | No | Tests needing real time/IO |
| `it.scoped` | TestClock | Yes | Tests with resources (acquireRelease) |
| `it.scopedLive` | Real clock | Yes | Real time + resources |

### it.effect - Use for Most Tests (with TestClock)

```typescript
it.effect("processes after delay", () =>
  Effect.gen(function* () {
    const fiber = yield* Effect.fork(
      Effect.sleep(Duration.minutes(5)).pipe(
        Effect.map(() => "done")
      )
    )
    yield* TestClock.adjust(Duration.minutes(5))
    const result = yield* Fiber.join(fiber)
    expect(result).toBe("done")
  })
)
```

### it.live - When You Need Real Time/External IO

```typescript
it.live("calls external API", () =>
  Effect.gen(function* () {
    yield* Effect.sleep(Duration.millis(100))  // Actually waits
    // Real HTTP calls, file system, etc.
  })
)
```

### TestClock Patterns

```typescript
// Always fork effects that sleep, then adjust clock
it.effect("timeout test", () =>
  Effect.gen(function* () {
    const fiber = yield* Effect.fork(
      Effect.sleep(Duration.seconds(30)).pipe(
        Effect.timeout(Duration.seconds(10))
      )
    )
    yield* TestClock.adjust(Duration.seconds(10))
    const result = yield* Fiber.join(fiber)
    expect(result._tag).toBe("None")  // Timed out
  })
)
```

## Sharing Layers Between Tests

```typescript
import { layer } from "@effect/vitest"

layer(AccountServiceLive)("AccountService", (it) => {
  it.effect("finds account by id", () =>
    Effect.gen(function* () {
      const service = yield* AccountService
      const account = yield* service.findById(testAccountId)
      expect(account.name).toBe("Test")
    })
  )

  // Nested layers
  it.layer(AuditServiceLive)("with audit", (it) => {
    it.effect("logs actions", () =>
      Effect.gen(function* () {
        const accounts = yield* AccountService
        const audit = yield* AuditService
      })
    )
  })
})

// Use real clock with layer
layer(MyService.Live, { excludeTestServices: true })("live tests", (it) => {
  it.effect("uses real time", () =>
    Effect.gen(function* () {
      yield* Effect.sleep(Duration.millis(10))  // Actually waits
    })
  )
})
```

## Property-Based Testing

FastCheck is re-exported from `effect/FastCheck`. Use `Arbitrary.make()` to create arbitraries from Schema.

```typescript
import { it } from "@effect/vitest"
import { Effect, FastCheck, Schema, Arbitrary } from "effect"

// Synchronous property test - array syntax
it.prop("symmetry", [Schema.Number, FastCheck.integer()], ([a, b]) =>
  a + b === b + a
)

// Synchronous property test - object syntax
it.prop("symmetry with object", { a: Schema.Number, b: FastCheck.integer() }, ({ a, b }) =>
  a + b === b + a
)

// Effectful property test
it.effect.prop("symmetry in effect", [Schema.Number, FastCheck.integer()], ([a, b]) =>
  Effect.gen(function* () {
    yield* Effect.void
    return a + b === b + a
  })
)

// With custom options
it.effect.prop(
  "with options",
  [Schema.Number],
  ([n]) => Effect.succeed(n === n),
  { fastCheck: { numRuns: 200 } }
)

// Create arbitrary from Schema manually
const accountArb = Arbitrary.make(Account)
it.prop("account test", [accountArb], ([account]) =>
  account.name.length > 0
)
```

## Testing with Testcontainers

Use `@testcontainers/postgresql` for integration tests against real PostgreSQL.

### Container Layer Setup

```typescript
// test/utils.ts
import { PgClient } from "@effect/sql-pg"
import { PostgreSqlContainer } from "@testcontainers/postgresql"
import { Data, Effect, Layer, Redacted } from "effect"

export class ContainerError extends Data.TaggedError("ContainerError")<{
  cause: unknown
}> {}

export class PgContainer extends Effect.Service<PgContainer>()("test/PgContainer", {
  scoped: Effect.acquireRelease(
    Effect.tryPromise({
      try: () => new PostgreSqlContainer("postgres:alpine").start(),
      catch: (cause) => new ContainerError({ cause })
    }),
    (container) => Effect.promise(() => container.stop())
  )
}) {
  static ClientLive = Layer.unwrapEffect(
    Effect.gen(function*() {
      const container = yield* PgContainer
      return PgClient.layer({
        url: Redacted.make(container.getConnectionUri())
      })
    })
  ).pipe(Layer.provide(this.Default))
}
```

### Using in Tests

```typescript
import { it, expect } from "@effect/vitest"
import { PgClient } from "@effect/sql-pg"
import { Effect } from "effect"
import { PgContainer } from "./utils.ts"

it.layer(PgContainer.ClientLive, { timeout: "30 seconds" })("AccountRepository", (it) => {
  it.effect("creates and retrieves account", () =>
    Effect.gen(function*() {
      const sql = yield* PgClient.PgClient
      yield* sql`CREATE TABLE accounts (id TEXT PRIMARY KEY, name TEXT NOT NULL)`
      yield* sql`INSERT INTO accounts (id, name) VALUES ('acc_1', 'Cash')`
      const rows = yield* sql`SELECT * FROM accounts WHERE id = 'acc_1'`
      expect(rows[0].name).toBe("Cash")
    })
  )
})
```

## Shared Database Container (Global Setup)

For faster tests, use vitest's `globalSetup` to share a single container.

### Step 1: Create Global Setup File

```typescript
// vitest.global-setup.ts
import { PostgreSqlContainer, StartedPostgreSqlContainer } from "@testcontainers/postgresql"

let container: StartedPostgreSqlContainer

export async function setup({ provide }: { provide: (key: string, value: unknown) => void }) {
  container = await new PostgreSqlContainer("postgres:alpine").start()
  provide("dbUrl", container.getConnectionUri())
}

export async function teardown() {
  await container?.stop()
}
```

### Step 2: Update Vitest Config

```typescript
// vitest.config.ts
export default defineConfig({
  test: {
    globalSetup: ["./vitest.global-setup.ts"],
    hookTimeout: 120000,
  }
})
```

### Step 3: Create Shared Layer

```typescript
// packages/persistence/test/Utils.ts
import { PgClient } from "@effect/sql-pg"
import { Layer, Redacted } from "effect"
import { inject } from "vitest"

export const SharedPgClientLive = Layer.effect(
  PgClient.PgClient,
  Effect.gen(function*() {
    const url = inject("dbUrl") as string
    return yield* PgClient.make({ url: Redacted.make(url) })
  })
)
```

### Step 4: Use in Tests

```typescript
// Uses global container - FAST
const TestLayer = RepositoriesLayer.pipe(
  Layer.provideMerge(MigrationLayer),
  Layer.provideMerge(SharedPgClientLive)
)
```

## When Layer.fresh IS Needed in Tests

**Module-level constant layers with different configurations per test:**

```typescript
// packages/persistence/src/Layers/AuthServiceLive.ts
export const AuthServiceLive = Layer.effect(AuthService, make)  // Module-level constant!

// test/AuthService.test.ts
const createTestLayer = (options: { autoProvisionUsers?: boolean }) => {
  const AuthConfigLayer = Layer.effect(AuthServiceConfig, Effect.succeed({
    autoProvisionUsers: options.autoProvisionUsers ?? true,
  }))

  // CORRECT: Layer.fresh escapes memoization per test layer build
  return Layer.fresh(AuthServiceLive).pipe(
    Layer.provideMerge(AuthConfigLayer),
  )
}
```

**When NOT needed**: Factory functions returning new compositions already return new objects.
