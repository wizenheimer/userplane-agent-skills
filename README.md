# Userplane Agent Plugins

[Userplane](https://userplane.io) screen recording SDK as a Claude Code plugin, Cursor plugin, and Codex plugin, with a skills pack for other AI coding agents.

Gives your agent 18 skills (14 framework/reference skills plus 4 workflow skills), both production Userplane MCP servers, and purpose-built workflows for **integrating**, **auditing**, **debugging**, and **privacy-proofing** a Userplane install.

---

## Installation

### Claude Code plugin (recommended for Claude Code)

Registers skills + MCP + subagents + slash commands in one go.

```
# one-time: register this repo as a plugin marketplace
/plugin marketplace add userplanehq/userplane-agent

# install the plugin
/plugin install userplane@userplane-marketplace
```

After install, verify:

```
/plugin          # userplane listed and enabled
/mcp             # userplane-workspace and userplane-docs listed
```

### Cursor plugin (recommended for Cursor)

Registers skills + MCP + subagents + slash commands + Cursor rules in one go.

Once published, install `userplane` from the Cursor Marketplace.

For local development, symlink this repo into Cursor's local plugin directory:

```bash
mkdir -p ~/.cursor/plugins/local
ln -sfn "$(pwd)" ~/.cursor/plugins/local/userplane
```

Reload Cursor, then verify:

```text
Cursor Settings -> Rules      # Userplane skills/rules are visible
Agent slash menu              # userplane-* commands are visible
MCP settings                  # userplane-workspace and userplane-docs are visible
```

### Codex plugin (recommended for Codex)

Registers skills + MCP in Codex. The Codex plugin uses a thin `plugins/userplane` wrapper that points to the canonical root skills and MCP config.

```bash
# one-time: register this repo as a plugin marketplace
codex marketplace add userplanehq/userplane-agent
```

Then open `/plugins` in Codex and install `userplane` from the `Userplane` marketplace.

After install, verify:

```text
/plugins         # userplane listed and enabled
/mcp             # userplane-workspace and userplane-docs listed
```

### Skills CLI (for Cursor without plugins, Windsurf, Codex, and other agents)

Installs all 18 skills into your agent's global skills directory. Works with any tool that consumes the open [Agent Skills](https://agentskills.io) standard.

```bash
npx skills add userplanehq/userplane-agent
```

### Claude Code (manual — skills only, no plugin)

Copy one skill:

```bash
cp -r skills/userplane-nextjs ~/.claude/skills/
```

…or all of them:

```bash
cp -r skills/* ~/.claude/skills/
```

### claude.ai Projects

Upload any `skills/<name>/SKILL.md` as project knowledge.

### Other agents

Paste the relevant `SKILL.md` into your agent's rules / context file.

---

## Workflow commands

| Workflow | Claude Code | Cursor | Purpose |
|---|---|---|---|
| Integrate | `/userplane:integrate` | `/userplane-integrate` | Detect framework and wire up the matching install (provider, script, CSP, env) |
| Audit | `/userplane:audit` | `/userplane-audit` | Read-only PASS/FAIL checklist against the matching framework skill |
| Debug | `/userplane:debug <id \| description>` | `/userplane-debug <id \| description>` | Fetch and analyze a recording via the workspace MCP |
| Privacy | `/userplane:privacy` | `/userplane-privacy` | Audit blur attrs, `setMetadata` leakage, CSP `frame-src` |

Each command delegates to exactly one subagent (`integrate-agent`, `audit-agent`, `debug-agent`, `privacy-agent`) — no shared agents, no modes.

---

## Usage

### Claude Code slash commands

```
/userplane:integrate
/userplane:audit
/userplane:debug rec_01HX2KYN
/userplane:debug "Sarah's checkout failure yesterday"
/userplane:privacy
```

### Cursor slash commands

```
/userplane-integrate
/userplane-audit
/userplane-debug rec_01HX2KYN
/userplane-debug "Sarah's checkout failure yesterday"
/userplane-privacy
```

**`/userplane:integrate`** detects your framework from `package.json` (Next.js, Nuxt, Angular, Vue, SvelteKit, Astro, TanStack Start, React/Vite, or static HTML), loads the matching skill, and writes the integration — provider component, script tag, env var, CSP headers. It finishes with a summary of files changed and the next step (usually: set your write key and restart the dev server). If Userplane is already installed, it tells you to run `/userplane:audit` instead.

**`/userplane:audit`** is read-only. It checks your existing install against the matching framework skill and produces a PASS/FAIL checklist — provider wiring, script placement, SSR hazards, `setUser`/`setMetadata` usage, CSP headers, env var consistency — with file:line citations and a concrete diff for every failure.

**`/userplane:debug`** accepts a recording ID or a natural-language description. With an ID, it resolves the workspace and fetches the recording directly via the workspace MCP. With a description ("Sarah's checkout failure yesterday"), it searches recordings by user and time range, confirms which one to analyze, then fetches console, network, and action data in parallel. It builds a correlated timeline, identifies the root cause, and proposes a fix with a code reference when the repo is accessible.

**`/userplane:privacy`** scans the repo for privacy issues: missing `data-userplane-blur` attributes on PII-adjacent inputs, raw PII in `setMetadata` calls, CSP `frame-src` gaps for third-party iframes (Stripe, Auth0, etc.), and inline handlers that leak sensitive data into the DOM. It reports findings ranked by severity with file:line citations and concrete diffs.

### Codex workflow skills

Invoke the Codex workflows by natural language or by naming the skill explicitly:

```text
$userplane-integrate Install Userplane in this app.
$userplane-audit Verify my Userplane setup.
$userplane-debug rec_01HX2KYN
$userplane-debug Sarah's checkout failure yesterday.
$userplane-privacy Check whether recordings expose PII.
```

**`userplane-integrate`** detects the framework, loads the matching framework skill, and writes the install as concrete edits.

**`userplane-audit`** is read-only. It produces the same PASS/FAIL integration checklist as `/userplane:audit`, with file:line citations and suggested diffs.

**`userplane-debug`** resolves a recording through the `userplane-workspace` MCP server, fetches console/network/action evidence after confirmation, and reports a root-cause timeline.

**`userplane-privacy`** is read-only. It audits blur coverage, metadata PII leakage, CSP iframe gaps, and inline DOM leaks.

### Natural-language activation

The skills also activate automatically when a prompt matches. No slash command is needed:

- *"set up Userplane in this Astro site"* → `userplane-astro`
- *"what's the setMetadata signature?"* → `userplane-metadata-sdk`
- *"why is the recorder iframe blocked?"* → `userplane-cdn` + `userplane-sensitive-data`
- *"what should I avoid when shipping Userplane to prod?"* → `userplane-best-practices`
- *"audit my Userplane setup"* → `userplane-audit`
- *"debug this Userplane recording"* → `userplane-debug`

---

## Skills

### Framework integrations

| Skill | Framework | Integration pattern |
|---|---|---|
| [userplane-react](skills/userplane-react/) | React (Vite) | Provider component |
| [userplane-nextjs](skills/userplane-nextjs/) | Next.js (App Router) | SSR-safe client provider |
| [userplane-vue](skills/userplane-vue/) | Vue 3 (Vite) | Vue plugin |
| [userplane-angular](skills/userplane-angular/) | Angular | `APP_INITIALIZER` |
| [userplane-nuxt](skills/userplane-nuxt/) | Nuxt 3 | Browser-only client plugin |
| [userplane-astro](skills/userplane-astro/) | Astro | Client-side script blocks |
| [userplane-sveltekit](skills/userplane-sveltekit/) | SvelteKit | `onMount` in root layout |
| [userplane-tanstack-start](skills/userplane-tanstack-start/) | TanStack Start | Search-param schema |
| [userplane-static-html](skills/userplane-static-html/) | Static HTML | No build step |

### SDK references

| Skill | Covers |
|---|---|
| [userplane-web-sdk](skills/userplane-web-sdk/) | Initialization, recorder control, recording state |
| [userplane-metadata-sdk](skills/userplane-metadata-sdk/) | `setUser` / `setMetadata` — attach user context to recordings |

### Cross-cutting guides

| Skill | Covers |
|---|---|
| [userplane-cdn](skills/userplane-cdn/) | Script placement, CSP, redirects, browser support |
| [userplane-sensitive-data](skills/userplane-sensitive-data/) | Blur config, third-party iframe compatibility |
| [userplane-best-practices](skills/userplane-best-practices/) | Cross-cutting install, SDK, metadata, privacy |

### Codex workflow skills

| Skill | Workflow |
|---|---|
| [userplane-integrate](skills/userplane-integrate/) | Detect framework and install Userplane as concrete edits |
| [userplane-audit](skills/userplane-audit/) | Read-only PASS/FAIL integration audit |
| [userplane-debug](skills/userplane-debug/) | Recording root-cause analysis via workspace MCP |
| [userplane-privacy](skills/userplane-privacy/) | Read-only PII, blur, metadata, and CSP audit |

---

## MCP servers

Declared in `.mcp.json` for Claude Code/Codex and `mcp.json` for Cursor.

| Server | URL | Purpose |
|---|---|---|
| `userplane-workspace` | `https://api.userplane.io/mcp` | Workspaces, projects, links, domains, recordings + console/network/actions viewers. OAuth 2.1 (DCR). |
| `userplane-docs` | `https://docs.userplane.io/mcp` | Documentation search. |

**First call** opens a browser for OAuth. Tokens are stored by the host agent. If a refresh silently fails, run `/mcp` to re-authenticate.

### Optional Codex smoke test

`codex marketplace add` updates local Codex config, so this repo does not run it as an automated test. To smoke-test manually:

```bash
codex marketplace add userplanehq/userplane-agent
```

Then install `userplane` from `/plugins` and verify `/mcp` shows `userplane-workspace` and `userplane-docs`.

---

## What's inside the plugin

```
.agents/plugins/
  marketplace.json
.claude-plugin/
  plugin.json
  marketplace.json
.cursor-plugin/
  plugin.json
.mcp.json                  # Claude Code MCP config
mcp.json                   # Cursor MCP config
plugins/userplane/         # Codex plugin wrapper
  .codex-plugin/plugin.json
  .mcp.json -> ../../.mcp.json
  skills -> ../../skills
rules/                     # Cursor rules
  userplane-workflows.mdc
agents/                    # 4 subagents (1:1 with commands)
  integrate-agent.md
  audit-agent.md
  debug-agent.md
  privacy-agent.md
commands/                  # 4 JTBD slash commands
  integrate.md
  audit.md
  debug.md
  privacy.md
skills/                    # 18 skills: 14 framework/reference + 4 Codex workflows
  userplane-react/
  userplane-nextjs/
  userplane-integrate/
  userplane-audit/
  userplane-debug/
  userplane-privacy/
  …
```

---

## License

MIT.
