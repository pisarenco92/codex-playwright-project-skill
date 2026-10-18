# Reference architecture

This architecture is adapted from the supplied example project, but clarifies ownership so the suite stays maintainable as it grows.

## Target tree

```text
playwright.config.ts
.env.example                         # optional; never commit real secrets
src/
├── api/                             # API clients / API-level setup helpers
├── fixtures/                        # test data, routes, builders, domain fixtures
├── internal/
│   └── testwright/
│       ├── index.ts                 # extended Playwright test + expect
│       └── pom/
│           ├── BasePage.ts
│           ├── BaseComponent.ts
│           ├── BaseSteps.ts         # optional
│           ├── BaseTasks.ts         # optional
│           └── index.ts
└── ui/
    ├── App.ts                       # exposes page/section step objects
    ├── AppTasks.ts                  # exposes cross-page/common tasks
    ├── components/                  # reusable bounded UI components
    ├── sections/                    # persistent/reusable page sections
    ├── pages/
    │   └── Feature/
    │       ├── FeaturePage.ts       # locators/component composition
    │       └── FeatureSteps.ts      # user actions + page assertions
    └── tasks/                       # workflows reusable across pages/features
tests/
└── feature.spec.ts                  # thin scenario/spec layer
```

Do not create all files mechanically. Create what the current project actually uses.

## Layer responsibilities

### Specs (`tests/*.spec.ts`)

Specs communicate behavior and intent. They should read like scenarios, not DOM scripts.

Good:

```ts
test('user can filter tours by age', async ({ app, tasks }) => {
  await tasks.navigate.toTours();
  await app.tours.filterByAge('18-35');
  await app.tours.expectEveryTourAgeRange('18-35');
});
```

Avoid putting long selector chains, raw polling, or page internals directly in specs unless the behavior is truly test-specific.

### `App`

`App` is the stable access point for page/section step objects.

```ts
export class App {
  constructor(private readonly page: Page) {}

  get tours() {
    return new TourPageSteps(this.page);
  }

  get header() {
    return new Header(this.page.getByRole('banner'));
  }
}
```

If repeated getter construction becomes measurably problematic, use simple instance fields or explicit caching. Do not introduce decorators only to imitate the sample.

### Page objects

A page object owns page-scoped locators and component composition. It should not become a grab bag of unrelated workflows.

```ts
export class TourPage extends BasePage {
  readonly heading = this.page.getByRole('heading', { level: 1 });
  readonly ageFilter = this.page.getByRole('checkbox', { name: '18 to 35' });
  readonly tourCards = this.page.getByTestId('tour-card');
}
```

Prefer accessible/user-facing locator APIs. If the product provides stable test IDs, use them as an explicit testing contract.

### Components

A component receives a root `Locator` and scopes its locators to that root.

```ts
export class TourCard extends BaseComponent {
  readonly title = this.root.getByRole('heading');
  readonly ageRange = this.root.getByTestId('age-range');

  async open(): Promise<void> {
    await this.title.click();
  }
}
```

Components may contain primitive behavior that is intrinsic to that component. Business workflows spanning multiple components belong in steps/tasks.

### Steps

Steps express page-level user interactions and assertions. This mirrors the supplied example's `*Steps` classes while keeping synchronization native to Playwright.

```ts
export class TourPageSteps extends BaseSteps {
  readonly ui = new TourPage(this.page);

  async filterByAge(label: string): Promise<void> {
    await test.step(`Filter tours by age: ${label}`, async () => {
      await this.ui.ageFilter.check();
    });
  }

  async expectHeading(text: string): Promise<void> {
    await expect(this.ui.heading).toHaveText(text);
  }
}
```

Use `test.step` for meaningful report structure, not for every individual click.

### Tasks

Tasks represent workflows reusable across pages or domains: navigation, API-backed setup, authentication, email polling, test-data creation, cleanup.

```ts
export class NavigationTasks extends BaseTasks {
  async toTours(): Promise<void> {
    await this.page.goto('/tours');
  }
}
```

Use relative URLs when `baseURL` is configured.

### API helpers

Prefer API setup over slow UI setup where the UI behavior itself is not under test. Keep API helpers typed and deterministic.

Examples:

- create prerequisite entities;
- authenticate or seed data;
- cleanup generated data;
- stub/control third-party responses.

Do not move the behavior under test out of the UI just to make a test pass faster.

## Fixtures

Create a single extended test entry point:

```ts
import { test as base, expect } from '@playwright/test';
import { App } from '../../ui/App';
import { AppTasks } from '../../ui/AppTasks';

type Fixtures = {
  app: App;
  tasks: AppTasks;
};

export const test = base.extend<Fixtures>({
  app: async ({ page }, use) => {
    await use(new App(page));
  },
  tasks: async ({ page }, use) => {
    await use(new AppTasks(page));
  },
});

export { expect };
```

Add worker-scoped fixtures only for expensive shared resources that are safe to reuse per worker. Keep backend data unique across parallel tests.

## Authentication

When many tests require login, prefer a Playwright setup project plus `storageState` rather than repeating a slow UI login in every test. Keep authentication state files out of Git.

Use separate authenticated states when roles differ.

## Environment/configuration

Prefer one clear source of truth:

- environment variables for runtime values and secrets;
- small typed config helpers for validation/defaults;
- optional checked-in non-secret environment profiles only if the repository genuinely needs them.

Do not combine multiple config libraries, `.env`, JSON profiles, and hard-coded URLs unless there is a clear reason.

Tests should navigate with relative paths when `baseURL` is available.
