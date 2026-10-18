# Playwright best practices

These rules are mandatory unless the repository has a documented reason to differ.

## Test what users observe

Assert user-visible outcomes rather than implementation details.

Prefer:

```ts
await expect(page.getByRole('status')).toHaveText('Saved');
```

Avoid testing internal JavaScript state, CSS class names, or DOM structure when those details are not part of the product contract.

## Keep tests isolated

Every test must be runnable independently and in any order.

- no scenario chaining across test cases;
- no mutation of shared module-level test data;
- no dependence on another test creating records first;
- create unique backend entities per test or per worker when parallel execution is enabled.

Use fixtures, API setup, builders, or `testInfo` to create isolated data.

## Locator priority

Prefer locators in roughly this order:

1. `getByRole()` with accessible name;
2. `getByLabel()` for form fields;
3. `getByPlaceholder()` / `getByText()` / `getByAltText()` / `getByTitle()` where they represent stable user-facing behavior;
4. `getByTestId()` or another stable explicit testing contract;
5. short CSS only when necessary;
6. XPath only as a last resort.

Avoid selectors based on styling classes, generated hashes, deep DOM nesting, and `nth()` when a semantic unique locator is possible.

Use chaining/filtering to scope locators rather than writing long selector strings.

## Synchronization

Use Playwright auto-waiting and retrying assertions.

Good:

```ts
await page.getByRole('button', { name: 'Save' }).click();
await expect(page.getByRole('status')).toHaveText('Saved');
```

Bad:

```ts
await page.getByRole('button', { name: 'Save' }).click();
await page.waitForTimeout(2000);
expect(await page.getByRole('status').textContent()).toBe('Saved');
```

Use `expect.poll()` when the state is outside a locator and genuinely requires polling.

Do not add sleeps to make flaky tests appear stable.

## Assertions

Prefer web-first assertions:

- `toBeVisible()`
- `toBeHidden()`
- `toHaveText()`
- `toContainText()`
- `toHaveValue()`
- `toHaveURL()`
- `toHaveTitle()`
- `toHaveCount()`
- `toBeChecked()`
- `toBeEnabled()`

Avoid one-shot patterns such as:

```ts
expect(await locator.isVisible()).toBe(true);
```

## Actions

Do not use `{ force: true }` by default. If an action is not actionable, diagnose why it is obscured, disabled, unstable, or detached.

Do not use raw `evaluate()` for normal interactions that Playwright locators already support.

## Async correctness

Every Playwright action and assertion promise must be awaited or returned.

Incorrect:

```ts
app.tours.applyFilter();
```

Correct:

```ts
await app.tours.applyFilter();
```

This is especially important because missing `await` can produce false passes and errors after a test has completed.

## Timeouts

Use defaults where possible. Increase timeout only for a known slow boundary and keep the override narrow.

Prefer:

```ts
await expect(downloadCompleted).toPass({ timeout: 30_000 });
```

or a focused operation timeout over globally inflating every test timeout.

## Third-party dependencies

Do not make the suite a health check for services the product does not control.

For third-party HTTP dependencies:

- mock/fulfill the expected response when testing your product's handling;
- use a dedicated contract/integration suite if real external integration must be checked;
- keep those checks separate from the fast deterministic PR suite.

## Test data

Keep test data explicit and unique.

Good options:

- builders/factories;
- API-generated entities;
- `testInfo.testId` or worker indexes for unique identifiers;
- worker-scoped accounts with test-scoped records.

Avoid hard-coded shared emails/accounts that can collide across workers.

## Page objects and abstraction

Abstraction should reduce duplication without hiding what the test does.

Good abstraction:

```ts
await app.checkout.submitOrder();
await app.checkout.expectOrderConfirmed();
```

Over-abstraction to avoid:

```ts
await performAction('checkout', 'submit', { retryMagic: true });
```

Prefer composition over deep inheritance.

## Reporting and debugging

Capture enough evidence for failures without making every run heavy.

Recommended baseline:

- trace: first retry;
- screenshot: only on failure;
- HTML report: enabled;
- video: failure/retry-only if it is useful to the team.

Use `test.step()` around meaningful business-level steps to improve reports.

## CI

CI should be reproducible.

- install dependencies from the lockfile;
- install browser/system dependencies explicitly;
- fail when `test.only` is committed;
- use finite global timeout;
- use retries only in CI unless there is a specific local reason;
- default to one worker for constrained generic CI; shard or increase workers only when resource isolation is proven;
- retain Playwright artifacts/reports on failure according to the CI platform conventions.

## Cross-browser strategy

Do not execute every low-value test on every browser by habit.

A common strategy is:

- PR smoke/critical path on Chromium;
- scheduled or merge-gate critical regression across Chromium, Firefox, and WebKit;
- browser-specific tests only where behavior differs.

If the user's requirements explicitly demand all browsers on every run, follow that requirement instead.
