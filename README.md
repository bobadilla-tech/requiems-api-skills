<p align="center">
  <p align="center">
    <a href="https://requiems.xyz/?utm_source=github&utm_medium=logo" target="_blank">
      <img src="https://raw.githubusercontent.com/bobadilla-tech/requiems-api/refs/heads/main/apps/dashboard/app/assets/images/logo.png" alt="Requiems API" width="280" />
    </a>
  </p>
  <p align="center">
    All-in-one backend for SaaS products.
  </p>
  <p align="center">
    <i>A product by <a href="https://bobadilla.tech">Bobadilla Technologies</a></i>
  </p>
</p>

# Requiems API Skills

Installable AI skills that give your coding agent direct knowledge of [Requiems API](https://requiems.xyz) — authentication, fraud detection, payments intelligence, and global data, all through one unified API.

Drop a skill into your agent and start calling Requiems API endpoints without copying docs or writing boilerplate.

## Install in Claude Code

Register this repository as a Claude Code plugin marketplace:

```
/plugin marketplace add bobadilla-tech/requiems-api-skills
```

Then browse and install:

1. Select **Browse and install plugins**
2. Select **requiems-api-skills**
3. Pick the skill set you want (e.g. `identity-skills`, `payments-skills`)
4. Select **Install now**

Or install directly:

```
/plugin install identity-skills@requiems-api-skills
/plugin install payments-skills@requiems-api-skills
```

Once installed, mention the skill naturally: _"Use the identity skill to protect this signup endpoint."_

## Install in OpenCode

Copy the skill file into your OpenCode skills directory:

```bash
npm install @requiems/api-skills
mkdir -p ~/.opencode/skills/requiems-identity
cp node_modules/@requiems/api-skills/skills/identity.md ~/.opencode/skills/requiems-identity/SKILL.md
```

## Install in GitHub Copilot

Place the skill file in your repo's `.github/copilot-instructions` folder or include it as a custom instruction set in your Copilot workspace settings.

```bash
npm install @requiems/api-skills
mkdir -p .github/copilot-instructions
cp node_modules/@requiems/api-skills/skills/identity.md .github/copilot-instructions/requiems-identity.md
```

## Install Manually (any agent)

Skills are plain Markdown files. Works anywhere that accepts custom instructions or a `SKILL.md` convention:

```bash
npm install @requiems/api-skills
# Then copy the .md file from node_modules/@requiems/api-skills/skills/ wherever your agent expects it
```

## What's Inside

Each skill gives your agent:

- Full endpoint reference (paths, params, response shapes)
- Copy-paste request examples
- Error codes and handling patterns
- Engine composition guides (e.g. `/v1/signup/protect`)

## Links

- [Requiems API docs](https://requiems.xyz/en/apis)
- [Official client libraries](https://github.com/bobadilla-tech/requiems-api-clients)
- [requiems.xyz](https://requiems.xyz)

---

> Contributing or building skills? See [docs/](docs/).
