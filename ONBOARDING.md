# Onboarding — LWC Basics (Badge 10)

**What this is:** a step-by-step guide to scaffold this repo, connect your Trailhead
Playground, create LWC components locally, deploy them to the org, and run the
per-unit Git workflow.

**What this isn't:** a Trailhead tutorial. Complete the hands-on challenges in the
Trailhead UI first — this doc only covers how your local repo drives the org.

---

## Before you start

- You need the `sf` CLI installed. Verify:
  ```bash
  sf --version
  ```
  This doc targets `@salesforce/cli` v2.143.6+. If you're on an older version, some
  flags may differ — spot-check with `sf <command> --help`.

- You need a Trailhead Playground ready for this badge. If you haven't created one
  yet, do that inside Trailhead first (the badge will prompt you).

- You need your playground username handy (it's in your Trailhead account, not in
  this file).

---

## Step 1 — Scaffold the SFDX project

Run this from the **parent directory** of the repo (one level up). The template
always creates a subfolder named after `--name`, so you can't be inside it yet:

```bash
cd ..
sf template generate project \
  --name trailhead-salesforce-lwc-basics \
  --output-dir . \
  --template standard \
  --lwc-language javascript \
  --manifest
cd trailhead-salesforce-lwc-basics
```

**What this does:**

| Flag | What it means |
|------|---------------|
| `--output-dir .` | Creates `trailhead-salesforce-lwc-basics/` right here — the repo root you already initialized |
| `--template standard` | Standard project layout: `force-app/`, `manifest/`, `sfdx-project.json`, etc. |
| `--lwc-language javascript` | LWC components default to `.js` (not TypeScript). Explicit is better than implicit. |
| `--manifest` | Generates a `manifest/package.xml` upfront so you don't forget it later |

> **Why `cd ..` first:** `--name` always creates a subfolder. If you run the command
> from inside the repo directory with `--output-dir .`, you end up with a nested
> `trailhead-salesforce-lwc-basics/trailhead-salesforce-lwc-basics/` — which is wrong.
> Running from the parent directory avoids this.

> **Why `--manifest` now:** generating the manifest at scaffold time means you can
> immediately retrieve metadata after login. Without it, you'd need to generate one
> as a separate step — easy to forget, annoying to debug.

Then commit the scaffolding as a clean starting point:

```bash
git add .
git commit -m "chore: scaffold SFDX project (LWC Basics)"
git push -u origin main
```

> **Why `main` not `master`:** it's the convention across all modern Git platforms
> and matches the branching strategy recommended in the `git/` reference docs.
> This repo was already initialized on `main` — keep it that way.

### If you already ran this inside the repo and got a nested directory

Move everything back up one level:

```bash
mv trailhead-salesforce-lwc-basics/* .
mv trailhead-salesforce-lwc-basics/.* . 2>/dev/null
rmdir trailhead-salesforce-lwc-basics
git add -A
git commit --amend -m "chore: scaffold SFDX project (LWC Basics)"
```

---

## Step 2 — Authenticate your Trailhead Playground

Log in via the browser:

```bash
sf org login web -a trailhead-playground
```

This opens a browser tab. Log in with your playground credentials, then come back
to the terminal.

**Verify the connection is live:**

```bash
sf data query \
  --query "SELECT Id, Name, OrganizationType FROM Organization" \
  --target-org trailhead-playground
```

You should see a JSON row with your org's name and `OrganizationType: "Developer Edition"`.
If you get an auth error, re-run the `login web` command — playground sessions expire.

Set it as the default org so you never have to type `--target-org` again:

```bash
sf config set target-org trailhead-playground
```

---

## Step 3 — Create a component and deploy

The LWC Basics badge has you build locally, then push to the org — not the other
way around. Your playground starts empty; there's nothing to retrieve.

Follow the Trailhead unit instructions to create each component. The raw unit
content is saved in `docs/` for quick reference without switching tabs:

| Unit | File |
|------|------|
| Unit 2 — Create Lightning Web Components | [`docs/unit-02-create-lwc.md`](docs/unit-02-create-lwc.md) |

From the CLI,
the generic pattern for a new LWC is:

```bash
sf lightning generate component \
  --name <componentName> \
  --type lwc \
  --output-dir force-app/main/default/lwc
```

Or use the VS Code Command Palette: `SFDX: Create Lightning Web Component`.

Once you've written the component files (`.html`, `.js`, `.js-meta.xml`), deploy
them to your playground:

```bash
sf project deploy start \
  --source-dir force-app/main/default/lwc/<componentName>
```

> This pushes only the component you just created. Targeted deploys are faster
> than pushing the whole `force-app/` tree, and they make it obvious what changed.

Verify the deploy succeeded:
```bash
sf project deploy report
```

Repeat this loop for each component the unit asks for. When the unit is complete,
commit the local files (see Step 4).

---

## Step 4 — Per-unit workflow

For each unit in LWC Basics, repeat this loop.

### 4.1 Create a feature branch

```bash
git checkout main
git pull origin main
git checkout -b feat/unit-0X-<slug>
```

**Naming convention:** `feat/unit-02-hello-world`, `feat/unit-03-properties`,
`feat/unit-04-events`. The `feat/` prefix makes intent scannable at a glance and
matches Conventional Commits types. Keep the slug short and dasherized.

### 4.2 Build the component locally

Complete the Trailhead hands-on challenges. Create components via the CLI
(`sf lightning generate component`) or VS Code Command Palette, then edit the
`.html`, `.js`, and `.js-meta.xml` files.

Write a Jest test for each component under `force-app/main/default/lwc/<name>/__tests__/`.
Run them locally before deploying:

```bash
npm test
```

### 4.3 Deploy to the playground

Push the component to your org:

```bash
sf project deploy start --source-dir force-app/main/default/lwc/<componentName>
```

Deploy only the component you worked on. Targeted deploys are faster and make it
clear what changed.

### 4.4 Commit and open a PR

```bash
git add force-app
git commit -m "feat(unit-0X): <what you did>"
git push -u origin feat/unit-0X-<slug>
gh pr create \
  --title "feat(unit-0X): <Unit Title>" \
  --body "LWC component and tests for Unit 0X."
```

**Conventional Commits:** use `feat`, `fix`, `docs`, `chore`, or `refactor` as the
type. The scope is the unit number. The description is one sentence, lowercase,
no period at the end. Examples:

```
feat(unit-02): add helloWorld LWC component
feat(unit-03): wire getRecord to display account name
docs(unit-02): add step-by-step challenge notes
```

### 4.5 Squash-merge and clean up

```bash
gh pr merge --squash --delete-branch
git checkout main
git pull origin main
```

**Why squash:** every commit on `main` is one complete, buildable change. `git bisect`
never lands on a broken WIP commit. The PR number is baked into the commit message
via `(#N)`, so `git log` → `gh pr view N` is always one hop. This is the default
recommended in the `git/NEAT-WORKFLOWS.MD` reference.

---

## Quick reference

| You want to… | Command |
|---|---|
| Auth | `sf org login web -a trailhead-playground` |
| Set default org | `sf config set target-org trailhead-playground` |
| Create LWC component | `sf lightning generate component --name <name> --type lwc --output-dir force-app/main/default/lwc` |
| Deploy component | `sf project deploy start --source-dir force-app/main/default/lwc/<name>` |
| Deploy report | `sf project deploy report` |
| New unit branch | `git checkout -b feat/unit-0X-<slug>` |
| Open a PR | `gh pr create --title "feat(unit-0X): <title>" --body "<body>"` |
| Merge and clean up | `gh pr merge --squash --delete-branch` |

---

## Recovery tips

**"No default org set" error:**
```bash
sf config set target-org trailhead-playground
```

**Auth expired (playground sessions time out):**
```bash
sf org login web -a trailhead-playground
```

**`gh` command not found:**
Install the GitHub CLI: https://cli.github.com — or use the web UI for PRs.

**Accidentally committed to `main` instead of a branch:**
```bash
git checkout -b feat/unit-0X-<slug>   # rescue the commit onto a branch
git checkout main
git reset --hard origin/main           # rewind main to match remote
```
