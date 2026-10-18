# Example prompts

Use these as starting points after installing `$playwright-project-setup`.

## Add Playwright to an existing application

```text
Use $playwright-project-setup to add a Playwright TypeScript E2E framework to this repository.
Inspect the existing repository first. Preserve its package manager, module system,
linting, formatting, environment configuration, and CI conventions. Do not overwrite
unrelated configuration. Add one representative smoke test if the application can be
run and its behavior can be determined from the repository.
```

## Create a dedicated Playwright automation repository

```text
Use $playwright-project-setup to bootstrap this repository as a dedicated Playwright
TypeScript automation project. Use the layered App/Page/Component/Steps/Tasks structure
from the skill where it improves maintainability, but do not create unused abstractions.
Configure Chromium for PR smoke tests and make cross-browser regression easy to enable.
```

## Modernize an existing Playwright suite

```text
Use $playwright-project-setup to review and modernize this Playwright project.
Keep working behavior, but identify and replace brittle selectors, fixed sleeps,
un-awaited Playwright calls, duplicated configuration, shared mutable test data, and
unnecessary framework abstractions. Preserve the public test intent and validate the
suite after the refactor.
```

## Bootstrap against a known environment

```text
Use $playwright-project-setup to configure this repository for E2E testing.
Base URL: https://staging.example.com
Start with Chromium smoke coverage. Keep the configuration ready for Firefox and WebKit
regression projects. Add environment handling without committing secrets.
```

## CI-focused setup

```text
Use $playwright-project-setup to set up Playwright and GitHub Actions for this repository.
Use lockfile-safe dependency installation, explicit Playwright browser/dependency setup,
CI-only retries, reproducible workers, a finite global timeout, HTML report artifacts,
and failure-focused traces/screenshots. Do not add CI configuration that conflicts with
existing repository workflows.
```
