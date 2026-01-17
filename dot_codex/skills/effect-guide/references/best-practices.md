# Effect Best Practices

This document covers critical rules and patterns for using Effect.

## Critical Rules

### 1. NEVER Use `any` Type or Type Casts (`x as Y`)

```typescript
// WRONG - never use any
const result: any = someValue
const data = value as any
function process(input: any): any { ... }

// WRONG - never use type casts
const account = data as Account
const id = value as AccountId

// CORRECT - use proper types, generics, or unknown
const result: SomeType = someValue
function process<T>(input: T): Result<T> { ... }

// CORRECT - use Schema.make() for branded types
const id = AccountId.make(rawId)

// CORRECT - use Schema.decodeUnknown for parsing
const account = yield* Schema.decodeUnknown(Account)(data)
```

**Type-Safe Alternatives to Casting:**

```typescript
// Use type parameters for Option.some/none
const someValue = Option.some<Account>(account)
const noneValue = Option.none<Account>()

// Use Option.fromNullable for nullable values
const desc = Option.fromNullable(row.description)

// Use identity for compile-time type verification
import { identity } from "effect/Function"
const verified = identity<Account>(x)
```

### 2. NEVER Use `catchAll` When Error Type Is `never`

```typescript
// If the effect never fails, error type is `never`
const infallibleEffect: Effect.Effect<Result, never> = Effect.succeed(value)

// WRONG - catchAll on never is useless
infallibleEffect.pipe(
  Effect.catchAll((e) => Effect.fail(new SomeError({ message: String(e) })))
)

// CORRECT - if error is never, just use the effect directly
infallibleEffect
```

### 3. NEVER Use Global `Error` in Effect Error Channel

```typescript
// WRONG - global Error breaks Effect's typed error handling
const bad: Effect.Effect<Result, Error> = Effect.fail(new Error("failed"))

// CORRECT - use Schema.TaggedError for all domain errors
export class ValidationError extends Schema.TaggedError<ValidationError>()(
  "ValidationError",
  { message: Schema.String }
) {}

const good: Effect.Effect<Result, ValidationError> = Effect.fail(
  new ValidationError({ message: "failed" })
)
```

### 4. NEVER Use `{ disableValidation: true }`

```typescript
// WRONG - disableValidation is banned
const account = Account.make(data, { disableValidation: true })

// CORRECT - let Schema validate the data, always
const account = Account.make(data)
```

### 5. Don't Wrap Safe Operations in Effect Unnecessarily

```typescript
// WRONG - Effect.try for operations that don't throw
const result = Effect.try(() => someValue)
const mapped = Effect.succeed(array.map(fn))

// CORRECT - pure function for transformation
const mapAccounts = (rows: Row[]): Account[] => rows.map(toAccount)

// CORRECT - use Effect.try ONLY for operations that might throw
const parsed = Effect.try(() => JSON.parse(jsonString))
```

### 6. NEVER Use `Effect.catchAllCause` to Wrap Errors

```typescript
// WRONG - catchAllCause catches BOTH errors AND defects (bugs)
const wrapSqlError = (operation: string) =>
  <A, E, R>(effect: Effect.Effect<A, E, R>) =>
    Effect.catchAllCause(effect, (cause) =>
      Effect.fail(new PersistenceError({ operation, cause: Cause.squash(cause) }))
    )

// CORRECT - use catchAll or mapError for expected errors only
const wrapSqlError = (operation: string) =>
  <A, E, R>(effect: Effect.Effect<A, E, R>) =>
    Effect.mapError(effect, (error) =>
      new PersistenceError({ operation, cause: error })
    )
```

## Pipe Composition

### Chain `.pipe()` Calls for Long Pipelines

```typescript
// Chained pipes - easier to read and group related operations
const result = effect
  .pipe(Effect.map(transformA))
  .pipe(Effect.flatMap(fetchRelated))
  .pipe(Effect.catchTag("NotFound", handleNotFound))
  .pipe(Effect.withSpan("myOperation"))

// Group related operations
const result = effect
  .pipe(
    Effect.map(transformA),
    Effect.flatMap(fetchRelated)
  )
  .pipe(
    Effect.catchTag("NotFound", handleNotFound),
    Effect.catchTag("ValidationError", handleValidation)
  )
```

**Note:** `.pipe()` has a maximum of 20 arguments - TypeScript's type inference breaks beyond this.

## Schema Patterns

### Always Use Schema for Data Classes

```typescript
// WRONG - manual class with Equal/Hash implementation
export class AccountNode implements Equal.Equal {
  [Equal.symbol](that: unknown): boolean { ... }
  [Hash.symbol](): number { ... }
}

// CORRECT - Schema.TaggedClass with automatic Equal/Hash
export class AccountNode extends Schema.TaggedClass<AccountNode>()("AccountNode", {
  account: Account,
  children: Schema.Array(Schema.suspend((): Schema.Schema<AccountNode> => AccountNode))
}) {
  get hasChildren(): boolean {
    return this.children.length > 0
  }
}
```

### Recursive Schema Classes

```typescript
// Step 1: Declare encoded type interface
export interface TreeNodeEncoded extends Schema.Schema.Encoded<typeof TreeNode> {}

// Step 2: Define class with Schema.suspend
export class TreeNode extends Schema.TaggedClass<TreeNode>()("TreeNode", {
  value: Schema.String,
  children: Schema.Array(Schema.suspend((): Schema.Schema<TreeNode, TreeNodeEncoded> => TreeNode))
}) {}
```

### Prefer Schema.Class Over Schema.Struct

```typescript
// WRONG - Schema.Struct creates plain objects
export const Account = Schema.Struct({
  id: AccountId,
  name: Schema.String
})

// CORRECT - Schema.Class creates a proper class
export class Account extends Schema.Class<Account>("Account")({
  id: AccountId,
  name: Schema.String
}) {
  get displayName() {
    return `${this.name} (${this.id})`
  }
}
```

### Branded Types for IDs

```typescript
export const AccountId = Schema.NonEmptyTrimmedString.pipe(
  Schema.brand("AccountId")
)
export type AccountId = typeof AccountId.Type

const id = AccountId.make("acc_123")
```

### Never Use *FromSelf Schemas

```typescript
// WRONG
export class Account extends Schema.Class<Account>("Account")({
  parentId: Schema.OptionFromSelf(AccountId)  // NO!
}) {}

// CORRECT
export class Account extends Schema.Class<Account>("Account")({
  parentId: Schema.Option(AccountId)  // YES - encodes to JSON properly
}) {}
```

### Schema.TaggedError for Domain Errors

```typescript
export class AccountNotFound extends Schema.TaggedError<AccountNotFound>()(
  "AccountNotFound",
  { accountId: AccountId }
) {
  get message(): string {
    return `Account not found: ${this.accountId}`
  }
}

// Type guards are automatically derived
export const isAccountNotFound = Schema.is(AccountNotFound)
```

### Always Use `.make()` - Never `new`

```typescript
// CORRECT
const account = Account.make({ id, code: "1000", name: "Cash", ... })

// WRONG
const account = new Account({ id, code: "1000", name: "Cash", ... })
```

### Use Effect Variants for Decoding (Never Sync)

```typescript
// WRONG - throws exceptions
const account = Schema.decodeUnknownSync(Account)(data)

// CORRECT - returns Effect
const accountEffect = Schema.decodeUnknown(Account)(data)

const program = Effect.gen(function* () {
  const account = yield* Schema.decodeUnknown(Account)(data)
  return account
})
```

## Value-Based Equality

Effect provides value-based equality through `Equal` and `Hash`:

```typescript
const account1 = Account.make({ id, name: "Cash" })
const account2 = Account.make({ id, name: "Cash" })
Equal.equals(account1, account2)  // true - same values!

// WRONG
if (account1 === account2) { ... }

// CORRECT
if (Equal.equals(account1, account2)) { ... }
```

### Use Chunk Instead of Array

Plain arrays don't implement Equal/Hash:

```typescript
// WRONG - arrays break structural equality
export class AccountHierarchy extends Schema.Class<AccountHierarchy>("AccountHierarchy")({
  children: Schema.Array(Account)
}) {}

// CORRECT - Chunk implements Equal/Hash
export class AccountHierarchy extends Schema.Class<AccountHierarchy>("AccountHierarchy")({
  children: Schema.Chunk(Account)
}) {}

const c1 = Chunk.make(1, 2, 3)
const c2 = Chunk.make(1, 2, 3)
Equal.equals(c1, c2)  // true!
```

## Error Handling

### Using Effect.catchTag

```typescript
const program = fetchAccount(accountId).pipe(
  Effect.catchTag("AccountNotFound", (error) =>
    Effect.succeed(createDefaultAccount())
  ),
  Effect.catchTag("PersistenceError", (error) =>
    Effect.logError(`Database error: ${error.cause}`)
  )
)

// Or use Match for exhaustive handling
const handleError = Match.type<AccountError>().pipe(
  Match.tag("AccountNotFound", (err) => `Account ${err.accountId} not found`),
  Match.tag("PersistenceError", (err) => `Database error occurred`),
  Match.exhaustive
)
```

## Service Pattern (Context.Tag + Layer)

```typescript
// Service interface
export interface AccountService {
  readonly findById: (id: AccountId) => Effect.Effect<Account, AccountNotFound>
  readonly findAll: () => Effect.Effect<ReadonlyArray<Account>>
  readonly create: (account: Account) => Effect.Effect<Account, PersistenceError>
}

// Service tag
export class AccountService extends Context.Tag("AccountService")<
  AccountService,
  AccountService
>() {}
```

### Creating Layers

**Layer.effect** - When creation is effectful but doesn't need cleanup:

```typescript
const make = Effect.gen(function* () {
  const config = yield* Config
  const sql = yield* SqlClient.SqlClient

  return {
    findById: (id) => Effect.gen(function* () { ... }),
    findAll: () => Effect.gen(function* () { ... }),
    create: (account) => Effect.gen(function* () { ... })
  }
})

export const AccountServiceLive = Layer.effect(AccountService, make)
```

**Layer.scoped** - When service needs resource cleanup:

```typescript
const make = Effect.gen(function* () {
  const sql = yield* SqlClient.SqlClient
  const changes = yield* PubSub.unbounded<AccountChange>()

  yield* Effect.forkScoped(
    sql`LISTEN account_changes`.pipe(
      Stream.runForEach((change) => PubSub.publish(changes, change))
    )
  )

  return {
    findById: (id) => Effect.gen(function* () { ... }),
    subscribe: PubSub.subscribe(changes)
  }
})

export const AccountServiceLive = Layer.scoped(AccountService, make)
```

**Composing layers:**

```typescript
export const AccountServiceWithDeps = AccountServiceLive.pipe(
  Layer.provide(SqlClientLive),
  Layer.provide(ConfigLive)
)
```
