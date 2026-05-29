# requiems-api-skills — Design Plan

> **Status: Spike / Investigation** This is a design document, not a finished
> project. The code described here does not exist yet. This document is under
> review by the maintainer team before implementation begins.

---

Installable skill packages for
[Requiems API](https://github.com/bobadilla-tech/requiems-api) — enabling AI
agents to interact with Requiems API endpoints without manual copy-pasting of
documentation.

## Overview

Requiems API exposes its endpoint documentation as structured YAML files. This
repository packages that documentation as installable skill bundles so that AI
agents (Claude, Cursor, Copilot, etc.) can discover, understand, and call
Requiems API endpoints natively.

Instead of copying documentation by hand, users install this package once and
their agent immediately gains full context on every available endpoint.

```bash
npm install @requiems/api-skills
```

## Problem

Requiems API maintains structured documentation in
`apps/dashboard/config/api_docs/`. Today, users who want their AI agent to
understand the API must:

1. Locate the relevant `.yml` file in the repository.
2. Copy its contents manually.
3. Paste it into their editor or agent context.
4. Repeat every time the API changes.

This is error-prone and produces stale, inconsistent agent contexts across
users.

## Why We Believe This Is Worth Solving

AI-assisted development is becoming a standard part of developer workflows. As
Requiems API grows in the number of endpoints and user base, the gap between
"API exists" and "agent can use it effectively" will widen. Today this friction
is manageable; at scale it becomes a real adoption barrier.

Specifically:

- Every new endpoint added to `requiems-api` requires a manual update on the
  user's side to keep their agent context current. This does not scale.
- Inconsistent agent context leads to incorrect API calls, which generates
  avoidable support load.
- Competing API platforms (e.g., Stripe, Twilio) already ship AI-ready context
  packages. Not having one puts Requiems API at a disadvantage in developer
  experience.
- The documentation infrastructure already exists in `api_docs/`. The marginal
  cost of packaging it properly is low compared to the adoption benefit.

## Approaches Considered

### Option A — Direct Git submodule (discarded)

Users add `requiems-api` as a git submodule and point their agent at the
`api_docs/` folder directly. Simple to set up, but requires users to manage
submodule updates manually and exposes them to the full repo rather than a
clean, minimal package.

### Option B — Static file download via CDN (discarded)

Publish the skill files as static assets on a CDN (e.g., jsDelivr over GitHub
raw). No packaging step needed. Downside: no versioning, no dependency
management, no way to pin a specific version. If a file moves or is renamed, all
users break silently.

### Option C — Per-skill individual packages (deferred)

Publish one npm package per endpoint (e.g., `@requiems/skill-sentiment`,
`@requiems/skill-sudoku`). Maximally granular, but multiplies maintenance
overhead significantly. Suitable as a future improvement once the single-package
approach is proven.

### Option D — npm package with automated sync pipeline (chosen)

Packages all skills into a single installable npm package. A GitHub Action keeps
the content in sync with `requiems-api` weekly, and a second action publishes a
new version when content changes. Users get standard dependency tooling
(pinning, updating, changelogs) with zero ongoing manual effort. This is the
approach implemented in this repository.

## Proposed Solution

This repository provides a pipeline with two automated stages:

**Continuous Integration (weekly)** A GitHub Action fetches the latest
documentation from `requiems-api`, transforms it into skill bundles, and commits
the updated files to this repository.

**Continuous Deployment (on change)** When skill content changes, a second
GitHub Action publishes a new package version to the npm registry. Users who
depend on the package receive updates through their normal dependency workflow.

## Proposed Repository Structure

```
requiems-api-skills/
├── .github/
│   └── workflows/
│       ├── sync.yml          # CI: weekly fetch + transform from requiems-api
│       └── publish.yml       # CD: publish to npm on content change
├── skills/
│   └── <category>/
│       └── <endpoint-name>.md  # One skill file per endpoint
├── scripts/
│   └── build.ts              # Transform raw API docs into skill format (Deno)
├── package.json
└── README.md
```

## Skill Format

Each skill file is a Markdown document with a structured front-matter header and
a natural-language description of the endpoint:

```markdown
---
name: sentiment-batch
method: POST
path: /v1/sentiment/batch
description: Analyze sentiment for a batch of up to 50 text items.
---

## Usage

Send an array of strings. Returns a result object per item with `score`,
`label`, and the original `text`. Results are returned in the same order as the
input.

## Parameters

| Field | Type             | Required | Description |
| ----- | ---------------- | -------- | ----------- |
| items | array of strings | yes      | 1–50 items  |

## Errors

- `400` — malformed JSON body
- `422` — validation failed (empty array, >50 items, or empty string in array)
```

## Automation Plan

### sync.yml (CI)

Runs every Monday at 08:00 UTC.

1. Checks out both `requiems-api` and this repository.
2. Runs `scripts/build.ts` via Deno to regenerate skill files.
3. If any skill file changed, opens (or updates) a pull request with the diff.
4. A maintainer reviews and merges; publish runs automatically on merge.

### publish.yml (CD)

Triggers on push to `main` when files under `skills/` change.

1. Bumps the patch version in `package.json`.
2. Runs `npm publish` using the `NPM_TOKEN` secret.
3. Creates a GitHub Release with the changelog.

## POC Plan

Before building the full pipeline, a small proof of concept will validate the
core assumption: that the existing `api_docs/` YAML files can be automatically
transformed into skill bundles and packaged for local installation.

### Scope

The POC covers only the transformation and local packaging step. It does not
include GitHub Actions, npm publishing, or CI/CD. The goal is to answer one
question: _does the data we have today produce a usable skill package?_

### Steps

**Step 1 — Pick two or three representative endpoints** Choose endpoints with
different characteristics from `api_docs/` — one with simple parameters and a
flat response (e.g., `sudoku`), one with a richer nested response structure
(e.g., `exercises`), and one with an external API dependency (e.g.,
`spellcheck`, which calls the LanguageTool API). This tests that the transformer
handles different levels of complexity, not just the happy path.

**Step 2 — Write `scripts/build.ts`**
A minimal Deno TypeScript script that:
- Reads a `.yml` file from `api_docs/`
- Extracts the relevant fields (name, method, path, description, parameters,
  errors, examples)
- Writes a `.md` file in the skill format defined in this document

No GitHub Actions, no automation. Just: `deno run --allow-read --allow-write scripts/build.ts --source ./sample-docs --output ./skills`

**Step 3 — Set up `package.json`** Minimal `package.json` that includes only the
`skills/` directory in the published files. Verify locally with `npm pack` —
this generates a `.tgz` file that shows exactly what a user would download.

**Step 4 — Install locally and verify**

```bash
npm pack
npm install ./requiems-api-skills-0.0.1.tgz
```

Open the installed files and confirm a real agent tool (Claude Code) can load
and use at least one skill correctly.

### Success criteria

- `build.ts` runs without errors on the three selected endpoints
- The generated `.md` files are valid skill format (correct front-matter, readable body)
- `npm pack` produces a `.tgz` with only the `skills/` folder inside
- At least one generated skill can be loaded by Claude Code without manual
  editing

### Out of scope for POC

- Automated sync with `requiems-api` (that is Step 3 of the full plan)
- Publishing to the npm registry (requires org credentials)
- Handling all edge cases in `api_docs/` structure
- Tests

## Known Limitations and Risks

**Sync lag**: Skills are regenerated weekly. If an endpoint is added or changed
mid-week, the package will not reflect it until the next sync cycle (or a manual
trigger).

**Format coupling**: The build script assumes the current structure of
`api_docs/` YAML files. Changes to that schema in `requiems-api` will break the
sync until `scripts/build.ts` is updated.

**npm registry dependency**: Publishing depends on an org-level npm token. Token
rotation or org changes require updating the `NPM_TOKEN` secret in this
repository's settings.

**Agent compatibility**: Skill format is optimized for Claude Code. Other agents
(Cursor, Copilot, etc.) may require additional adapters or a different directory
layout.

**No granular versioning**: Today, all skills are published as a single package.
If only one endpoint changes, users re-download everything. A per-skill package
approach is a future improvement.
