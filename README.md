# requiems-api-skills

Installable skill package for [Requiems API](https://github.com/bobadilla-tech/requiems-api),
enabling AI agents to interact with its endpoints natively.

> This project is currently in POC (proof of concept) phase.
> See [DESIGN.md](./DESIGN.md) for the full proposal.

---

## Prerequisites

### To use the skills

No extra dependencies. Clone this repo and the `skills/` folder is ready to use.

### To regenerate the skills from source

Required only if you need to rebuild the skill files (e.g. after new endpoints are
added to `requiems-api`):

- [Deno](https://deno.land/) installed on your system.

  ```bash
  curl -fsSL https://deno.land/install.sh | sh
  ```

- The `requiems-api` repository cloned at the same level as this repo:

  ```
  ~/projects/
  ├── requiems-api/        ← required as documentation source
  └── requiems-api-skills/ ← this repo
  ```

---

## Generate skills

From the root of this repository:

```bash
deno run --allow-read --allow-write scripts/build.ts \
  --source ../requiems-api/apps/dashboard/config/api_docs \
  --output ./skills
```

This reads all `.yml` files from `api_docs/` and generates one `.md` file per endpoint
in the `skills/` folder. A successful run ends with:

```
Done. 96 skills generated, 0 files skipped.
```

---

## Install locally for testing

```bash
npm pack
npm install ./requiems-api-skills-0.0.1.tgz
```

`npm pack --dry-run` shows which files would be included in the package without creating the `.tgz`.

---

## Test a skill in Claude Code

1. Create the skill folder in the Claude Code global directory:

   ```bash
   mkdir -p ~/.claude/skills/skill-name
   ```

2. Copy the generated `.md` as `SKILL.md`:

   ```bash
   cp skills/skill-name.md ~/.claude/skills/skill-name/SKILL.md
   ```

   Example with sudoku:

   ```bash
   mkdir -p ~/.claude/skills/sudoku-post-batch
   cp skills/sudoku-post-batch.md ~/.claude/skills/sudoku-post-batch/SKILL.md
   ```

3. Open Claude Code and run `/reload-skills` to pick up the changes.

4. Verify with `/skills` that the skill appears in the list.

---

## API key

Skills make real API calls. You need an API key from
[requiems.xyz](https://requiems.xyz).

The recommended way to configure it is in the Claude Code global `settings.json`
(`~/.claude/settings.json`):

```json
{
  "env": {
    "REQUIEMS_API_KEY": "your-api-key-here"
  }
}
```

This makes `$REQUIEMS_API_KEY` available in all Claude Code sessions without
exposing it in code or git.

For other agents that do not read Claude Code's `settings.json` (e.g. Gemini CLI),
add the key to `~/.bashrc` as well:

```bash
echo 'export REQUIEMS_API_KEY="your-api-key-here"' >> ~/.bashrc
source ~/.bashrc
```

---

## Repository structure

```
requiems-api-skills/
├── scripts/
│   └── build.ts       # Deno script that transforms YAML docs into skill .md files
├── skills/            # Generated files — one per endpoint
├── package.json       # npm package configuration
└── DESIGN.md          # Full project plan
```

---
