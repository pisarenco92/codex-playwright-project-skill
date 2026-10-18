# Setup and validation checklist

Use this as a final gate. Do not claim completion while known setup errors remain.

## Repository integration

- [ ] Existing package manager preserved.
- [ ] Existing module system preserved.
- [ ] Existing scripts/config merged instead of destructively replaced.
- [ ] No unrelated dependencies removed or reformatted without need.
- [ ] Lockfile updated with the repository's package manager.

## Playwright configuration

- [ ] `testDir` is correct.
- [ ] `forbidOnly` is enabled on CI.
- [ ] Local retries are normally 0.
- [ ] CI retries are finite.
- [ ] CI workers are stable/reproducible or deliberately sharded.
- [ ] CI `globalTimeout` is finite.
- [ ] HTML reporter is configured without auto-opening in CI.
- [ ] Trace is retained on first retry or another deliberate failure-focused mode.
- [ ] Screenshots are captured on failure.
- [ ] `baseURL` is environment/config driven when applicable.
- [ ] Browser projects match the requested coverage.

## Framework design

- [ ] Specs use the project `test` fixture import.
- [ ] `app` fixture exposes page/section steps.
- [ ] `tasks` fixture exposes cross-page/common workflows when needed.
- [ ] Components are rooted by `Locator`.
- [ ] Page objects hold selectors/component composition.
- [ ] Steps hold page-level behavior/assertions.
- [ ] Tasks hold reusable cross-page/setup workflows.
- [ ] No unnecessary custom locator proxy/decorator/framework magic was added.

## Reliability

- [ ] No fixed sleeps for synchronization.
- [ ] No un-awaited Playwright actions/assertions.
- [ ] No default use of `force: true`.
- [ ] No brittle XPath/CSS selectors where semantic locators are possible.
- [ ] Web-first assertions are used for dynamic UI state.
- [ ] Tests do not rely on order.
- [ ] Parallel tests cannot collide on shared test data.
- [ ] Third-party dependencies are controlled/mocked where appropriate.

## Security/configuration

- [ ] No real credentials/tokens committed.
- [ ] `.env`/auth-state outputs are gitignored where used.
- [ ] `.env.example` contains variable names/examples only, not secrets.
- [ ] URLs and environment values are not unnecessarily duplicated across specs.

## Developer experience

- [ ] README explains install/run/debug/report commands.
- [ ] Scripts use the repository's package manager.
- [ ] `test`, `test:ui`, `test:headed`, `test:debug`, and `typecheck` exist when useful and not already covered by equivalents.
- [ ] CI workflow is added/updated only when the repository uses that CI provider or the user requested it.

## Validation commands

Adapt commands to the package manager.

Typical npm validation:

```bash
npm ci
npm run typecheck
npm run lint
npx playwright test --list
npx playwright test --project=chromium
```

If browsers are not installed locally:

```bash
npx playwright install
```

Typical Linux CI browser bootstrap:

```bash
npx playwright install --with-deps
```

If the application must be started locally, use Playwright `webServer` when practical so local and CI execution share the same lifecycle.

## Expected completion report

The agent should report only facts it verified, e.g.:

```text
Created:
- playwright.config.ts
- src/internal/testwright/index.ts
- src/ui/App.ts
- src/ui/AppTasks.ts
- tests/smoke.spec.ts

Validation:
- typecheck: passed
- lint: passed
- playwright --list: passed
- chromium smoke: passed

Not run:
- Firefox/WebKit: browser binaries unavailable in this environment
```

Never report a test as passing when it was merely generated or listed.
