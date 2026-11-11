---
title: AI-Assisted Test Automation with Playwright
nav_order: 40
recipe: false          # draft: unlisted until reviewed; set to true (and remove the three lines below) to list it
nav_exclude: true
search_exclude: true
sitemap: false
summary: Add a Playwright test suite to your Liferay Workspace that checks what real users see, and let your AI coding agent write, run and triage the tests.
description: Add Playwright end-to-end tests to an existing Liferay Workspace, and use an AI coding agent to write, run and triage them.
tags: [Playwright, Testing, AI coding agents, Claude Code, Liferay Workspace]   # shown on the recipe and its card, and used by the filter on the home page
---

# AI-Assisted Test Automation with Playwright
{: .no_toc }

{: .note }
**DRAFT for review.**

1. TOC
{:toc}

## What You'll Build

A **Playwright end-to-end test suite** in your existing Liferay Workspace. It signs in as real test users, such as a site member or a partner, and as Guest, and checks what each of them sees in a real browser. Your **AI coding agent** writes the tests from your acceptance criteria, runs them, and tells you why a test failed. You review each test before you commit it.

```text
your-workspace/
├── .claude/skills/e2e-tests/SKILL.md   ← how your agent writes, runs and triages the tests
├── .agents/skills/e2e-tests/SKILL.md   ← the same skill, for other agents
├── client-extensions/ …                ← your existing workspace, unchanged
└── tests/e2e/
    ├── playwright.config.js
    ├── auth.setup.js                   ← signs each test user in once per run
    ├── lib/liferay.js                  ← Liferay sign-in and helpers
    ├── smoke.spec.js                   ← first tests: Guest and a signed-in member
    └── test-users.properties           ← your test users (git-ignored)
```

## Ingredients

| Ingredient | Type | Guide |
| --- | --- | --- |
| Your Liferay Workspace, with a running Liferay DXP | Liferay Workspace | [Liferay Workspace](https://learn.liferay.com/w/dxp/development/tooling/liferay-workspace) |
| Playwright Test | Open-source test runner | [playwright.dev](https://playwright.dev/docs/intro) |
| An AI coding agent: Claude Code, or another agent that reads Agent Skills | AI coding agent | [Claude Code](https://code.claude.com/docs) |
| Playwright MCP server (optional) | Lets the agent open and inspect pages itself | [Playwright MCP](https://github.com/microsoft/playwright-mcp) |

**Download:**

| File | What it is |
| --- | --- |
| [liferay-playwright-e2e.zip](downloads/liferay-playwright-e2e.zip) | The `tests/e2e` suite and the `e2e-tests` agent skill, laid out to unzip at your workspace root |

**Before you start, you need:**

- A Liferay Workspace (any DXP version with the standard Login portlet) and its Liferay running, locally or on a test environment you can reach.
- Node.js 20 or later.
- Google Chrome installed. The suite uses your installed Chrome, so Playwright doesn't need to download a browser.
- One or more **test users** that can sign in. Use ordinary accounts, never the administrator.

{: .important }
Never test as the administrator. An admin session or a Basic Auth request can read things a real user can't, so the test passes even when the user's view is broken. The suite signs in as each test user through the normal sign-in page.

## Steps

### Step 1: Add the Suite to Your Workspace

1. Unzip [liferay-playwright-e2e.zip](downloads/liferay-playwright-e2e.zip) at your workspace root, the folder that holds `gradle.properties` and `settings.gradle`. It adds `tests/e2e/`, `.claude/skills/e2e-tests/` and `.agents/skills/e2e-tests/`, and changes nothing else. If your workspace already has a `tests/e2e` folder, unzip somewhere else first and merge by hand.
1. Install Playwright Test:

   ```bash
   cd tests/e2e
   npm install
   ```

`tests/e2e/.gitignore` keeps the test users file, the saved sessions, `node_modules` and the reports out of Git, so you don't need to change your workspace's own `.gitignore`.

| File | What it does |
| --- | --- |
| `lib/liferay.js` | Signs a test user in through `/c/portal/login`, which shows the Login portlet on any site. Then it checks whose session it is by calling `/o/headless-admin-user/v1.0/my-user-account`. It also has `sessionFetch` for calling a headless API from the page as that user. |
| `env.js` | Works out the base URL, the test users file, and which test users to sign in |
| `auth.setup.js` | Runs first: signs each test user in once and saves the session to `.auth/<alias>.json` |
| `playwright.config.js` | Uses your installed Chrome, keeps a trace and a screenshot of each failure, and writes an HTML report |
| `smoke.spec.js` | Three first tests: as Guest, the home page loads signed out and the Control Panel answers `404`; the `member` test user is signed in |

### Step 2: Add Your Test Users

1. In Liferay, create a test user, or use an existing non-admin account, with the roles and site membership that the user you're testing would have.
1. **Sign in once as that user by hand**, in a private browser window. New users often have to change their password, accept the terms of use, or set a security question at first sign-in. The suite stops at these screens and tells you which one it hit; it doesn't complete them for you.
1. Copy `tests/e2e/test-users.properties.example` to `tests/e2e/test-users.properties` and add each test user under a short alias:

   ```properties
   member.email=jane.member@example.com
   member.password=<password>
   ```

The setup signs in the `member` alias by default. To sign in others, list them when you run the tests (Step 3), for example `E2E_PERSONAS=member,partner`.

### Step 3: Run It

From `tests/e2e`:

```bash
npx playwright test
```

```text
  ✓  1 [setup] › auth.setup.js › sign in as member
  ✓  2 [liferay] › smoke.spec.js › Guest › the home page loads, signed out
  ✓  3 [liferay] › smoke.spec.js › Guest › the Control Panel answers 404 to Guest
  ✓  4 [liferay] › smoke.spec.js › member › is signed in as the member

  4 passed
```

To see the report, with a trace and a screenshot of any failure:

```bash
npx playwright show-report
```

The suite finds the address of your Liferay on its own:

| Setting | Used when |
| --- | --- |
| `LIFERAY_BASE_URL=https://test.example.com` | Always, when set: use it to test another environment |
| `web.server.host` in `bundles/portal-ext.properties` or `configs/local/portal-ext.properties` | When it's set, for example behind a tunnel or a proxy. Pages then carry links to that host, so the browser must use it too. |
| `http://localhost:8080` | Otherwise |

If you can't install Chrome, for example on a CI runner, run `npx playwright install chromium` and set `E2E_CHANNEL=chromium`.

### Step 4: Give Your Agent the Skill

The zip includes an `e2e-tests` skill: instructions your agent loads when you ask it for a test. It tells the agent:

- **Where things are:** the specs, the sign-in helper, the test users file.
- **How to write a test:** start from the acceptance criteria, read the real selectors from your fragments and client extensions, and prove a restriction from both sides.
- **How to triage a failure:** decide whether the product, the environment or the test is wrong.
- **What it must never do:** sign in as the administrator, print a password, or weaken a test to make it pass.

Claude Code finds the skill in `.claude/skills/` on its own. Other agents that read the Agent Skills layout find the copy in `.agents/skills/`. If your workspace has a `CLAUDE.md` or `AGENTS.md` that lists its skills, add a line:

```markdown
| Write, run, or triage an automated end-to-end test in `tests/e2e` | `e2e-tests` |
```

### Step 5: Let the Agent See the Page (Optional)

To pick selectors that work, the agent needs to see the real page, not guess from the code. The [Playwright MCP server](https://github.com/microsoft/playwright-mcp) lets it open a page, click through it, and read the page's accessibility tree, which is what `getByRole` matches. To add it to Claude Code:

```bash
claude mcp add playwright npx @playwright/mcp@latest
```

{: .note }
The MCP browser starts signed out, so it sees what Guest sees. Use it to explore the page's structure. Prove what a signed-in user sees with the suite itself, never by signing in as the administrator in the MCP browser.

### Step 6: Have the Agent Write a Test From Your Acceptance Criteria

Give the agent the same acceptance criteria you'd give a person. For example, for a members-only page:

💬 **Prompt:**

```
Using e2e-tests, write tests/e2e/member-resources.spec.js for ticket
PROJ-123, tagged @PROJ-123:

- A signed-in member opens /web/intranet/member-resources and sees the
  "Member Resources" heading and at least one resource card.
- Guest gets a 404 on that page, and the home page shows no link to it.

Find the page and the card's markup in our site initializer and fragments.
Test as the member alias and as Guest. Run the suite and show me the
result. If a test fails, tell me whether it's the product, the environment
or the test, and don't change any permissions to make it pass.
```

Expect a spec like this one. The paths and names come from your own site:

```javascript
import {expect, test} from '@playwright/test';

import {authFile} from './env.js';

const PAGE = '/web/intranet/member-resources';

test.describe('member', () => {
	test.use({storageState: authFile('member')});

	test('sees the member resources', {tag: '@PROJ-123'}, async ({page}) => {
		const response = await page.goto(PAGE);

		expect(response.status()).toBe(200);
		await expect(page.getByRole('heading', {name: 'Member Resources'})).toBeVisible();
		await expect(page.locator('.resource-card').first()).toBeVisible();
	});
});

test.describe('Guest', () => {
	test('gets a 404 and no link to the page', {tag: '@PROJ-123'}, async ({page}) => {
		const response = await page.goto(PAGE);

		expect(response.status()).toBe(404);

		await page.goto('/');

		await expect(page.locator(`a[href$="${PAGE}"]`)).toHaveCount(0);
	});
});
```

**Review it before you commit it.** Check that:

- Every user is expected to see **something**, so an empty or broken page can't pass.
- Each restriction is tested from both sides: one user sees it, another doesn't.
- No test signs in as the administrator or uses Basic Auth.

Then **watch it fail once**. Remove the member role's View permission on the page, run the suite, and check that the member test fails. Then restore the permission. A test you've never seen fail doesn't prove anything.

### Step 7: Triage Failures With the Agent

When a run fails, give the agent the failure rather than reading the trace yourself:

```
Using e2e-tests, run the suite. For each failure, open its trace and tell me
whether it's the product, the environment or the test, with the evidence.
Don't change any test until I agree.
```

The skill lists the usual Liferay causes:

| Symptom | Usual cause |
| --- | --- |
| Setup stops on `/c/portal/update_password`, `update_terms_of_use` or `update_reminder_query` | A first-login screen: sign in once by hand as that user |
| "The Login portlet says: Authentication failed" | Wrong password, an inactive user, or an instance that signs in by screen name rather than email |
| A list is empty for the test user but full for the admin | A missing permission (or, for Guest, a Service Access Policy), not missing data |
| A bare `403` with an empty body from a headless call | A Service Access Policy, an OAuth scope, or a call made without the `x-csrf-token` header |
| Navigation times out over `localhost` | `web.server.host` is set: use the public URL with `LIFERAY_BASE_URL` |
| Every test fails after a server restart | The server is still starting; the setup signs in again on the next run |

**Optional:** Playwright 1.56 and later include [test agents](https://playwright.dev/docs/test-agents) (*planner*, *generator* and *healer*) that you can add to Claude Code with `npx playwright init-agents --loop=claude`. The healer replays a failing test and proposes a fix. Use it when a selector really changed, never to make a real regression pass. The rules in your `e2e-tests` skill still apply.

## Result

From `tests/e2e`, `npx playwright test` signs each test user in once, runs every spec as those users and as Guest, and writes an HTML report, with a trace and a screenshot of each failure.

To build the suite up:

- Ask the agent for a test whenever you finish a ticket, from the ticket's acceptance criteria, so every feature you ship stays covered.
- Run every test for a ticket with `npx playwright test --grep @PROJ-123`.
- Run the suite against your test environment with `LIFERAY_BASE_URL` before you promote a release.
- **Going further:** run the suite in CI against a Liferay Docker container built from your workspace (`blade gw buildDockerImage`), with its test users created before the tests run.
