---
name: zig-coding
description: Write or review Zig code for declaration order, naming, ownership, safety, concurrency, and error handling. Use for substantive Zig changes or reviews, not documentation-only work.
---

# Zig coding

## Layout and naming

- Use the pinned compiler's language and standard-library APIs and run `zig fmt` on changed sources.
- Use braces for statement control flow. Concise value-producing `if` and `switch` expressions are fine. Prefer guards over deeply nested branches.
- Separate function-body steps by purpose with blank lines; keep an acquisition beside its `defer` or `errdefer`.
- Use `snake_case` for values and fields, `camelCase` for functions, and `PascalCase` for types and type factories. Normalize acronyms: `GpuBuffer`, `readJson`, `EntityId`. Preserve external ABI spelling.
- Name file-as-struct files `PascalCase.zig`; other source files use `snake_case.zig`. Use `root.zig` for a directory's public module, without creating directories for self-contained files.
- Keep human-facing messages and test descriptions lowercase; error tags remain `PascalCase`.
- Use `self` for receivers. Prefer meaningful names and distinguish IDs, indices, handles, counts, capacities and units.
- Do not add implementation comments to Zig source, except required license/generated notices. Put non-obvious contracts in separate API or architecture documentation.

## Imports and declaration order

Import a single used declaration directly. When using several declarations, import their module once and keep derived aliases immediately beside it, namespaces before types. Follow ZLS `source.organizeImports`: `std`, `builtin`/`root`, `build_options`, package imports, then file imports. Keep category spacing and do not group unrelated aliases together. Import sibling implementations directly rather than routing them through a facade.

Keep declarations private unless another module needs them. Use this order in namespace files, file-as-struct types and nested containers, skipping empty groups:

1. Imports, direct imported aliases and intentional re-exports.
2. Fields grouped by ownership and purpose; preserve enum/union tag order.
3. Public constants and types, then public variables.
4. Public functions, lifecycle and primary operations first.
5. Private constants and types, then private variables.
6. Private functions, grouped by responsibility and after callers where practical.
7. Standalone compile-time checks and tests.

`Self = @This()` belongs in private declarations. A computed type-factory call belongs with types, not import aliases. Separate groups, functions and multiline types with one blank line; adjacent simple constants may stay together.

Keep module roots thin and exports curated. Inspect functions around 60 lines and files around 400 lines for mixed responsibilities; refactor files at 500 nonblank, non-comment lines unless splitting would obscure ownership or create artificial fragments. Do not compress code to meet a count.

## Construction and ownership

- Prefer `var value: Type = try .init(...)` when inferred initialization is supported. Use literals for trivial non-owning structs.
- Use `const` unless mutable storage or reassignment is needed. Use `*const Self` for nonmutating methods and `*Self` for mutation.
- Successful initialization establishes invariants; failed initialization releases partial resources. Register cleanup immediately: `defer` for scope-owned resources, `errdefer` for ownership transferred on success.
- Give every allocation and native resource an owner and release point. Retain the selected allocator until allocations and deferred users are finished. Copying an owning value does not clone it.
- Specify owned versus borrowed returns and what invalidates borrows. Stable IDs do not imply stable addresses. Scratch allocations must not escape their reset boundary.
- Release resources before ending struct `deinit()` with `self.* = undefined`; save return values before invalidation. Destruction is not automatically idempotent.
- Never read `undefined` storage before initialization. Localize unsafe casts and ABI adaptation behind explicit invariants.

## Safety and errors

Validate external data, recoverable stale handles, bounds and resource failures with runtime checks in every build. Use error unions for failures and optionals for expected absence. Propagate errors unless the current boundary can handle them meaningfully; do not silently swallow them.

Use assertions for programmer invariants, with no required side effects. Assertions do not replace runtime validation or synchronization. Keep expensive structural checks entirely inside `if (builtin.optimize == .debug)`; compile-time type/schema checks belong in explicit `comptime` evaluation. Do not disable safety to hide a failure or use `unreachable` for an inconvenient runtime case.

State failure guarantees precisely. Rejection, atomic publication and rollback are different contracts; test state preservation when promised. Check size arithmetic and bound externally driven allocation and queued work.

## Concurrency

Give mutable state a thread owner or explicit synchronization. Jobs receive disjoint writes or immutable reads, including reachable payloads and aliases. A const pointer, generation check or atomic reference count does not make its payload thread-safe.

Publish completed data through synchronized queues or launch/join boundaries. Define atomic ordering and lock order where used. Do not invoke callbacks or wait for dependent work while holding locks they may need. Cancellation is not completion: join workers and callbacks before freeing their storage. Keep allocator owners stable and respect allocator thread safety.

## Abstractions and verification

Prefer concrete types, composition and explicit dependencies. Use generics where they express actual reuse; avoid reflection and type gymnastics that obscure navigation. Profile before introducing complex unsafe optimizations.

Verify changed behavior and meaningful failure paths with the owning package's actual checks. Test ownership cleanup and allocation failure where relevant. Public tests should exercise observable contracts without exposing private declarations for testing. Report checks actually run and distinguish cross-compilation from runtime validation.
