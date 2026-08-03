# Onboarding — LWC Basics (Badge 10)

**What this is:** a step-by-step guide to scaffold this repo, connect your Trailhead
Playground, retrieve org metadata, and run the per-unit Git workflow.

**What this isn't:** a Trailhead tutorial. Complete the hands-on challenges in the
Trailhead UI first — this doc only covers how your local repo mirrors the org.

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

## Step 3 — Retrieve baseline org metadata

Your playground comes pre-configured with metadata from the Trailhead challenge
(accounts, contacts, etc.). Pull it into the repo so you have a snapshot of the
starting state:

```bash
sf project retrieve start \
  --manifest manifest/package.xml \
  --target-org trailhead-playground
```

This downloads everything listed in `manifest/package.xml` into `force-app/`.

**Commit it separately** from the scaffolding. Mixing project structure and org
metadata into one commit makes it harder to revert one without the other:

```bash
git add .
git commit -m "chore: retrieve baseline org metadata"
git push
```

> **Long-term reasoning:** this baseline commit is your "before" picture. When you
> finish a unit and retrieve again, `git diff baseline..HEAD -- force-app` shows
> exactly what the unit changed. It's also your reset point — if the org gets into
> a weird state, you can re-create it from this snapshot.

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

### 4.2 Do the unit

Complete the Trailhead hands-on challenges in the Salesforce UI / Developer Console.
Your org now has new Apex classes, LWC components, or other metadata that doesn't
exist in the repo yet.

### 4.3 Pull the changes into the repo

```bash
sf project retrieve start \
  --manifest manifest/package.xml \
  --target-org trailhead-playground
```

> If you added **new** metadata types that aren't in `manifest/package.xml` yet
> (for example, a StaticResource for the first time), regenerate the manifest first:
> ```bash
> sf project generate manifest \
>   --from-org trailhead-playground \
>   --output-dir manifest
> ```
> Then run the `retrieve start` command above. The manifest is your "what to pull"
> checklist — if a metadata type isn't in it, `retrieve` skips it silently.

### 4.4 Commit and open a PR

```bash
git add force-app manifest
git commit -m "feat(unit-0X): <what you did>"
git push -u origin feat/unit-0X-<slug>
gh pr create \
  --title "feat(unit-0X): <Unit Title>" \
  --body "Consolidated metadata and code for Unit 0X."
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
| Pull org metadata | `sf project retrieve start --manifest manifest/package.xml` |
| Regenerate manifest | `sf project generate manifest --from-org trailhead-playground --output-dir manifest` |
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
