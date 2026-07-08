# Rust Coding Standards

Applies to every Rust crate in the portfolio (Tauri backends, CLI tools, shared core crates).

## 1. Toolchain and Edition

- Edition 2021 or later; new crates start on the latest stable edition available at creation time.
- MSRV (minimum supported Rust version) is declared explicitly in `Cargo.toml` (`rust-version`) once a crate has external consumers; portfolio-internal crates track current stable.
- `rustfmt` with default settings runs in CI (`cargo fmt --check`); formatting is never a matter of personal preference in a PR review.

## 2. Linting

- `cargo clippy --workspace -- -D warnings` is a required, non-excludable CI gate for every crate in the workspace. A crate is never excluded from `check`/`clippy`/`test` to work around a failure; the failure is fixed. (This portfolio has previously hidden real bugs, including a Tokio runtime panic, behind a `--exclude` flag; that pattern is explicitly disallowed.)
- Warnings are treated as errors in CI (`-D warnings`); locally, address a warning rather than silencing it with `#[allow(...)]` unless there is a documented reason (a comment stating why, directly above the attribute).

## 3. Error Handling

- Library crates define their own error enum (`thiserror`), giving callers structured errors they can match on. Application/binary crates may use `anyhow` at the top level where a caller only needs to propagate and log.
- `unwrap()`/`expect()` are acceptable only in tests, in `main()` startup code where a failure is genuinely unrecoverable, or where a prior check makes the `None`/`Err` case provably unreachable (and that reasoning is stated in a comment). They are not acceptable in request-handling or command-handling code paths.
- Never shadow the standard library's `Result`/`Option` with a local type alias without verifying every generic-arity usage still resolves; a local `type Result<T> = std::result::Result<T, MyError>` alias silently breaks any code expecting the two-argument `std::result::Result`, including trait implementations like `Serialize::serialize`. Prefer explicitly importing `std::result::Result` where both are needed in the same file.

## 4. Async and Runtime Discipline

- Do not mix runtime-management strategies. Pick one entry pattern per binary and apply it consistently:
  - `#[tokio::main] async fn main() { ... }`, **or**
  - a plain `fn main()` that constructs its own runtime and calls `.block_on(...)` exactly once at the top level.
- Inside a framework that already owns the async runtime (Tauri's `.setup()` callback, for example), never call `tokio::runtime::Handle::current()` or construct a second runtime. Use the framework's own runtime-agnostic helpers (`tauri::async_runtime::block_on`/`spawn` for Tauri) instead. Mixing these patterns produces one of two 100%-reproducible panics: "Cannot start a runtime from within a runtime" or "there is no reactor running", both of which have shipped to production in this portfolio before being caught.
- Async commands/handlers that take a reference-containing parameter (a `State<'_, T>` in Tauri, a borrowed context in an async trait) must return a `Result`, not a bare value; this is enforced by the framework's own trait bounds where applicable, and should be treated as a hard rule even where it is not.

## 5. Dependencies

- Every crate declares every dependency it directly uses in its own `Cargo.toml`; relying on a dependency transitively provided by another workspace member (which happens to compile today because of the cargo build cache) is a defect waiting to surface on a clean checkout.
- Shared dependencies across workspace members are promoted to `[workspace.dependencies]` and referenced via `{ workspace = true }`, so version bumps happen in one place.
- `cargo audit` runs in CI on every PR; a new advisory against a portfolio crate's dependency tree blocks merge unless explicitly, temporarily ignored with a comment stating the advisory ID, the reason it doesn't apply, and a plan to remove the exception.

## 6. sqlx Specifically

- Compile-time-checked queries (`sqlx::query!`/`query_as!`) require either a live `DATABASE_URL` or a committed `.sqlx` offline query cache (`cargo sqlx prepare`) for the build to succeed on a clean checkout, including in CI. Whichever mode a repository uses, document it in the README's Quick Start.
- The `.sqlx` cache, when used, is committed to version control and regenerated (`cargo sqlx prepare`) whenever a query changes; a stale cache produces a confusing compile error unrelated to the actual change.

## 7. Testing

- Unit tests live alongside the code they test (`#[cfg(test)] mod tests` in the same file) for anything not requiring external I/O.
- Integration tests that need a database run against a real, ephemeral SQLite/Postgres instance created in a test fixture, not against a shared, stateful test database.
- `cargo test --workspace` is a required CI gate alongside `check` and `clippy`; a crate with zero tests is a tracked gap in that repository's `ROADMAP.md`, not a silent omission.

## 8. Dead Code Discipline

- An unused `use` import, an unused function parameter, or a variable marked `mut` without ever being mutated are removed, not suppressed with `#[allow(unused)]`, unless there is a forward-looking reason documented in a comment.
- A method that is defined but never called anywhere in the codebase is either wired up or removed; "dead but might be useful later" is a YAGNI violation, not a justification.
