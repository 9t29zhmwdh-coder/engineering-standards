# TypeScript Coding Standards

Applies to every TypeScript/React frontend in the portfolio (Tauri frontends, web tools).

## 1. Compiler Strictness

- `strict: true` in every `tsconfig.json`, including `noImplicitAny`, `strictNullChecks`, and `noUncheckedIndexedAccess` where the codebase supports it.
- `noUnusedLocals` and `noUnusedParameters` are enabled; an unused import or variable is a build failure, not a lint warning left for later. This has caught real dead-code and stale-refactor bugs in this portfolio (a destructured store value nobody read, a leftover unused import after a refactor).
- `npm run build` (which runs `tsc` before bundling) is a required CI gate; a type error is never masked by a bundler that skips type checking.

## 2. State Management

- Prefer real framework state (`useState`, a store hook like Zustand) over manual patterns that mimic state without triggering re-renders. A `useRef` that is mutated directly and then cast to look like a `useState` tuple does not trigger a re-render on mutation; this has shipped as a real bug in this portfolio (a diagram canvas that silently never updated after data loaded, because the "state" was a ref in disguise). If a value needs to drive a re-render, it must be real state.
- Global/cross-component state uses a single store per concern (Zustand store, React Context) rather than prop-drilling through more than two or three levels.
- Derived values are computed from state at render time or in a memoized selector, not duplicated into a second piece of state that can drift out of sync with its source.

## 3. Component Conventions

- Functional components with hooks; no class components in new code.
- One component per file; the file name matches the exported component name.
- Props are typed with an explicit `interface Props { ... }` (or inline type for trivial components), never `any`.
- Side effects (`useEffect`) declare their full dependency array; a suppressed `// eslint-disable-next-line react-hooks/exhaustive-deps` requires a comment explaining why the omission is safe.

## 4. Naming and Local Shadowing

- Descriptive names throughout; a loop/map variable named the same as an imported hook or utility (for example, naming a `.map(t => ...)` callback variable `t` in a file that also imports a `useT()` translation hook as `t`) is a shadowing bug waiting to happen and is avoided by choosing a distinct name for one of the two.
- Boolean variables and props read as a predicate: `isLoading`, `hasKey`, `canSave`, not `loading` as a bare noun where a boolean is meant (both are acceptable individually; consistency within a file matters more than which convention is chosen).

## 5. Styling

- Tailwind CSS utility classes are the default styling approach across the portfolio's frontends; a missing `postcss.config.js` silently disables all Tailwind processing (Vite passes the raw `@tailwind` directives through unprocessed, producing a tiny, visibly broken CSS bundle), so its presence is checked whenever a new frontend is scaffolded.
- `@import` statements in a stylesheet (web font imports, for example) must precede every other at-rule, including `@tailwind` directives; ordering this incorrectly produces a build warning and, in stricter bundler configurations, a build failure.

## 6. Internationalization

- Every user-facing frontend supports English as the default language with additional languages (German, at minimum, across this portfolio) switchable at runtime, persisted to `localStorage`, never requiring a rebuild to change.
- Translation strings are centralized in a single `i18n.ts`/`i18n.tsx` module per frontend using a nested dictionary keyed by feature area, with a `useT()` hook for components and a plain `t()`/`getLang()` pair for non-component code (formatting helpers, non-React utility modules).
- A string is added to the dictionary in every supported language in the same commit that introduces it in the UI; a hardcoded string in only one language is a regression, not a follow-up task.

## 7. Package Management

- `package-lock.json` is committed and is the source of truth for reproducible installs; CI uses `npm ci`, never `npm install`, so the lock file is respected exactly.
- `npm audit` runs in CI; a new high/critical advisory blocks merge unless explicitly, temporarily accepted with a stated reason and a follow-up plan.

## 8. Testing

- Component and unit tests (Vitest/Jest + React Testing Library) cover business logic and non-trivial component behavior; trivial presentational components are covered by the fact that a broken build/type error would already fail CI.
- End-to-end tests (Playwright) are reserved for the critical user flows of each tool, not exhaustive coverage of every screen.
