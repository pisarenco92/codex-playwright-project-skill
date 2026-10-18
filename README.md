# Playwright Project Setup Skill for Codex

A reusable agent skill for setting up or modernizing a production-quality **Playwright + TypeScript** test automation project.

The skill is designed for **OpenAI Codex** and is distributed through the open `skills` CLI, so you can keep this repository in GitHub and install it into any project with `npx skills`.

The framework guidance is based on a real Playwright project structure and preserves useful ideas such as thin specs, page/component objects, reusable steps/tasks, fixtures, an application facade, and environment-aware configuration while enforcing current Playwright reliability and maintainability practices.

## What the skill does

When invoked, Codex inspects the target repository before making changes and then creates or adapts a Playwright framework without blindly overwriting existing conventions.

It guides Codex to:

- preserve the existing package manager, module system, linting, formatting, and CI conventions;
- configure Playwright Test and strict TypeScript;
- create a scalable `App / Pages / Components / Steps / Tasks` architecture when appropriate;
- create a single extended Playwright fixture entry point;
- prefer user-facing locators and stable test IDs over brittle XPath/CSS selectors;
- use Playwright auto-waiting and web-first assertions instead of fixed sleeps;
- use environment-driven `baseURL` and keep secrets out of source control;
- configure CI-safe retries, workers, reports, traces, screenshots, and finite run timeouts;
- create a representative smoke test only when application behavior can be determined safely;
- validate the generated project before claiming setup is complete.

## Repository layout

```text
codex-playwright-project-skill/
├── README.md
├── VERSION
├── examples/
│   └── prompts.md
└── skills/
    └── playwright-project-setup/
        ├── SKILL.md
        └── references/
            ├── architecture.md
            ├── best-practices.md
            └── setup-checklist.md
```

The installable skill is:

```text
skills/playwright-project-setup/
```

`SKILL.md` contains the skill metadata and main instructions. The `references/` directory contains detailed architecture, Playwright practices, and setup validation guidance that Codex reads when it applies the skill.

# Installation

The recommended installation method is the **Skills CLI**. You do not need to install the CLI globally; `npx` runs it directly.

## Install from GitHub into the current project

Run this command from the project where you want Codex to use the skill:

```bash
npx skills add pisarenco92/codex-playwright-project-skill \
  --skill playwright-project-setup \
  --agent codex
```

Project scope is the default. This is useful when the skill should travel with a specific repository and be available to the team.

## Install globally for Codex

If you want the skill available when using Codex in **all of your projects**, install it globally:

```bash
npx skills add pisarenco92/codex-playwright-project-skill \
  --skill playwright-project-setup \
  --agent codex \
  --global
```

Short flags are also supported:

```bash
npx skills add pisarenco92/codex-playwright-project-skill \
  -s playwright-project-setup \
  -a codex \
  -g
```

The Skills CLI handles the appropriate Codex skill location for the selected scope.

## Non-interactive installation

Useful for bootstrap scripts or CI environments:

```bash
npx --yes skills add pisarenco92/codex-playwright-project-skill \
  --skill playwright-project-setup \
  --agent codex \
  --yes
```

For a global non-interactive install:

```bash
npx --yes skills add pisarenco92/codex-playwright-project-skill \
  --skill playwright-project-setup \
  --agent codex \
  --global \
  --yes
```

## Inspect the repository before installing

You can ask the CLI which skills it discovers:

```bash
npx skills add pisarenco92/codex-playwright-project-skill --list
```

You should see:

```text
playwright-project-setup
```

## Test a local clone

```bash
git clone https://github.com/pisarenco92/codex-playwright-project-skill.git
cd codex-playwright-project-skill
npx skills add . --list
```

# Updating the skill

After you update this GitHub repository, users can check installed skills for newer versions with:

```bash
npx skills check
```

Update installed skills with:

```bash
npx skills update
```

# Removing the skill

Remove the project-scoped installation with:

```bash
npx skills remove playwright-project-setup --agent codex
```

For a global installation:

```bash
npx skills remove playwright-project-setup --agent codex --global
```

# Using the skill in Codex

Explicit invocation is the most predictable approach.

## Basic setup

```text
Use $playwright-project-setup to set up a Playwright TypeScript automation project in this repository.
```

## Existing application

```text
Use $playwright-project-setup to add Playwright to this repository.

Inspect the existing project first. Preserve its package manager, module system,
linting, formatting, environment configuration, and CI conventions.

Create a maintainable smoke-test framework without overwriting unrelated configuration.
```

## New test automation repository

```text
Use $playwright-project-setup to create a Playwright TypeScript automation framework.

Use the layered App/Page/Component/Steps/Tasks architecture from the skill,
but create only abstractions that are needed by the first tests.
```

## Bootstrap against a known environment

```text
Use $playwright-project-setup to bootstrap this repository for UI testing.

Base URL: https://example.test

Start with Chromium smoke coverage and prepare the configuration so Firefox
and WebKit can be enabled for regression coverage.
```

More examples are available in [`examples/prompts.md`](examples/prompts.md).

# How Codex should behave

The skill intentionally tells Codex **not to scaffold blindly**. It should first inspect:

- `package.json` and lockfiles;
- TypeScript/module configuration;
- existing test and source layout;
- lint/format configuration;
- CI workflows;
- environment/config conventions;
- application start command and base URL;
- any existing Playwright setup.

It then chooses the smallest useful framework that fits the repository.

# Target Playwright architecture

For a non-trivial suite, the skill generally prefers:

```text
.
├── playwright.config.ts
├── tests/
└── src/
    ├── api/
    ├── fixtures/
    ├── internal/
    │   └── testwright/
    │       ├── index.ts
    │       └── pom/
    └── ui/
        ├── App.ts
        ├── AppTasks.ts
        ├── components/
        ├── sections/
        ├── pages/
        └── tasks/
```

This is a guideline rather than a mandatory directory tree. The skill explicitly tells Codex not to create empty layers or framework abstractions without a current use case.

# Playwright principles enforced by the skill

1. Prefer semantic locators such as `getByRole()` and `getByLabel()`; use stable test IDs when needed.
2. Avoid long CSS/XPath selectors tied to DOM implementation details.
3. Use auto-waiting and web-first assertions instead of `waitForTimeout()` sleeps.
4. Await every asynchronous Playwright action and assertion.
5. Do not use `force: true` as a default workaround.
6. Keep tests independent and safe to run in any order.
7. Keep parallel test data isolated.
8. Prefer API/setup fixtures for prerequisites when the setup UI itself is not under test.
9. Keep secrets and generated auth state out of source control.
10. Report only validations that were actually executed.

# CI guidance

The skill guides Codex toward stability-oriented Playwright CI defaults, including:

- `forbidOnly` in CI;
- finite retries in CI and normally zero retries locally;
- conservative CI worker counts unless safe parallel capacity is known;
- a finite `globalTimeout`;
- explicit browser/system-dependency installation;
- HTML reports;
- failure-focused screenshots and traces rather than recording everything continuously.

These are defaults and should be adapted to the target project's actual CI infrastructure.

# Maintaining this repository

The skill source of truth is:

```text
skills/playwright-project-setup/
```

When improving the skill:

1. update `SKILL.md` or files in `references/`;
2. update `VERSION` when appropriate;
3. test discovery locally with `npx skills add . --list`;
4. commit and push the changes;
5. consumers can run `npx skills check` and `npx skills update`.

## Validate before publishing

```bash
npx skills add . --list
```

The output should include `playwright-project-setup`.

# Requirements

- Node.js with `npx`
- Codex
- Git, when installing from a GitHub repository

No custom installation script is required.
