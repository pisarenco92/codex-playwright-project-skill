---
name: playwright-project-setup
description: Set up or modernize a maintainable TypeScript Playwright end-to-end test project using a layered App/Page/Component/Steps/Tasks architecture, robust fixtures, environment-aware configuration, CI-friendly defaults, and Playwright best practices. Use when asked to initialize Playwright, create an E2E automation framework, add Playwright to an existing repository, or restructure a Playwright suite into a scalable framework.
---

# Playwright project setup

Build a production-quality Playwright Test project in TypeScript. Preserve the useful architecture of the reference project—thin specs, an `App` facade, page/component objects, reusable steps/tasks, fixtures, and environment-aware configuration—while preferring native Playwright capabilities and current best practices over custom abstractions.

Read these references before editing:

- `references/architecture.md` — target structure and responsibility boundaries.
- `references/best-practices.md` — mandatory Playwright coding rules.
- `references/setup-checklist.md` — setup, migration, and final validation checklist.

## Workflow

### 1. Inspect before changing anything

Inspect the repository first. Determine:

- package manager from lockfiles and existing scripts;
- Node/TypeScript module conventions;
- whether Playwright is already installed;
- existing test directories, CI workflows, lint/format tooling, and environment handling;
- application start command and target/base URL when available;
- whether the repository is greenfield or an existing application that must not be disrupted.

Do not overwrite working project configuration wholesale. Merge with existing conventions unless they are clearly broken or conflict with the requirements below.

### 2. Choose the smallest useful scaffold

For a new Playwright framework, prefer this structure unless the repository already has an equivalent convention:

```text
.
├── playwright.config.ts
├── tests/
├── src/
│   ├── api/
│   ├── fixtures/
│   ├── internal/testwright/
│   │   ├── index.ts
│   │   └── pom/
│   └── ui/
│       ├── App.ts
│       ├── AppTasks.ts
│       ├── components/
│       ├── sections/
│       ├── pages/
│       └── tasks/
└── .env.example              # only if env vars are used
```

Create only folders and abstractions that are needed now. Empty extension points may use `.gitkeep` only when the repository convention requires tracked empty directories.

### 3. Install Playwright correctly

Use the repository's package manager. Prefer `@playwright/test` as a dev dependency.

For greenfield setup, use the current supported Playwright version instead of pinning an obsolete version copied from an example project. Do not replace unrelated dependencies.

Browser installation must be explicit and CI-safe:

- local bootstrap may use `playwright install`;
- Linux CI should use `playwright install --with-deps` unless the CI image already contains compatible Playwright browsers;
- do not add expensive browser downloads to `postinstall` unless the repository explicitly wants that behavior.

### 4. Configure TypeScript for test code

Use strict TypeScript. Match the repository's module system instead of forcing CommonJS or ESM.

At minimum preserve strict type checking and include Playwright/test sources. Avoid handwritten replacements for Node's `process` types when `@types/node` is available.

Add a `typecheck` script when the project has no equivalent.

### 5. Configure Playwright

Create or update `playwright.config.ts` with sensible defaults:

- `testDir` points to the real E2E/spec directory;
- isolated tests and parallel execution are allowed by default;
- `forbidOnly` is enabled in CI;
- retries are enabled in CI and normally disabled locally;
- CI workers favor reproducibility; use 1 unless the repository intentionally shards or has proven safe parallel capacity;
- set a finite CI `globalTimeout` so reports can still be produced if a run hangs;
- HTML reporting is available and does not auto-open in CI;
- capture trace on first retry;
- capture screenshots on failure;
- record video only when the project needs it; prefer retry/failure-only modes;
- `baseURL` comes from environment/config, not duplicated literals in tests;
- define Chromium, Firefox, and WebKit projects when cross-browser coverage is expected; if the user explicitly wants only one browser, configure only that browser.

Do not hide product flakiness by adding large arbitrary timeouts or retries everywhere.

### 6. Create the test fixture facade

Create a single project-facing test import, e.g. `src/internal/testwright/index.ts`, that extends Playwright's base test with fixtures such as:

- `app`: page/section step facade;
- `tasks`: cross-page/common workflows.

Re-export `expect` and only the Playwright helpers that genuinely improve ergonomics.

Specs should normally import `test` and `expect` from this project fixture module, not mix imports from multiple competing fixture files.

### 7. Implement the layered UI model

Follow the responsibility boundaries in `references/architecture.md`.

Key rule: preserve the example project's useful separation, but avoid framework magic for its own sake.

- Base classes may wrap `Page` or `Locator` when that improves consistency.
- Do not add memoization decorators, locator proxies, inheritance trees, or custom DSLs unless a concrete current use case justifies them.
- Prefer simple getters, composition, typed helper functions, and native `Locator` APIs.

### 8. Write one representative test when the application target is known

If the repository exposes a runnable target and enough product information is available, add one small smoke test that proves the scaffold works.

The smoke test must:

- verify user-visible behavior;
- use the project fixture;
- use resilient locators;
- contain at least one web-first assertion;
- be independent of other tests;
- avoid destructive behavior and external third-party dependencies.

If the target application or base URL cannot be inferred, do not invent product behavior. Finish the framework setup without a fake passing test and document the missing runtime input.

### 9. Add developer scripts and documentation

Keep scripts concise and predictable. Typical useful scripts are:

```text
test              run the normal Playwright suite
test:ui           Playwright UI mode
test:headed       headed execution
test:debug        Playwright debug mode
test:smoke        smoke-tagged tests, when tags exist
test:report       open/show the latest HTML report
typecheck         TypeScript no-emit check
lint              repository linter, when configured
```

Do not create aliases that merely duplicate each other.

Update README documentation with:

- prerequisites;
- install/browser-install commands;
- how to set the base URL/environment;
- common test commands;
- architecture summary;
- how to add a new page/component/step/task;
- report/trace debugging instructions.

### 10. Validate before finishing

Run the checks that exist or that you added. At minimum attempt:

1. dependency install using the lockfile-safe command;
2. TypeScript check;
3. lint/format checks if configured;
4. `playwright test --list`;
5. the representative smoke test when a runnable target exists.

Fix setup errors rather than leaving obviously broken generated code.

If browser binaries, network access, secrets, or an unavailable application prevent execution, say exactly which validation could not run. Do not fabricate passing results.

## Non-negotiable quality rules

- Every async Playwright action/assertion must be awaited unless the API intentionally returns a promise that is returned to the caller.
- Never use `page.waitForTimeout()` or Node sleeps as synchronization for UI state.
- Do not default to `force: true` to bypass actionability checks.
- Prefer `getByRole`, `getByLabel`, `getByPlaceholder`, `getByText`, or stable test IDs over CSS/XPath implementation selectors.
- Use CSS/XPath only when a stable user-facing or explicit test contract cannot identify the target; keep fallback selectors short and localized.
- Prefer Playwright web-first assertions over manual `isVisible()`/`textContent()` polling assertions.
- Tests must not depend on execution order or another test's side effects.
- Keep secrets out of source control; provide `.env.example` with names only when useful.
- Do not make test data global if parallel tests could collide. Generate or fixture-isolate data per test/worker.
- Mock or control third-party dependencies instead of testing another company's availability.
- Keep specs readable at the user-workflow level. Push reusable UI details into components/steps and cross-page orchestration into tasks.

## Final response

Summarize:

- files created/changed;
- architecture decisions;
- commands to install/run/debug;
- checks actually executed and their results;
- any runtime values the user still needs to provide.
