# requiems-api-skills

Requiems API skills are Markdown-based examples and endpoint knowledge for AI
agents. Install the package to get the generated skill files, then place the
skills your agent understands into its local skills folder.

## Quickstart

1. Install the package:

   ```bash
   npm install @requiems/api-skills
   ```

2. Copy a skill into your agent's skills directory.

   Example for Claude Code:

   ```bash
   mkdir -p ~/.claude/skills/sudoku-post-batch
   cp node_modules/@requiems/api-skills/skills/sudoku-post-batch.md ~/.claude/skills/sudoku-post-batch/SKILL.md
   ```

3. Reload skills in your agent and start using the endpoint examples.

## For developers

See [docs/DEVELOPERS.md](docs/DEVELOPERS.md) for the short technical note.

## Package contents

This package publishes the generated Markdown skill files under `skills/`.
