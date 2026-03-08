# Userplane Agent Skills

Agent skills for integrating [Userplane](https://userplane.io) screen recording SDK into web applications. These skills help AI coding agents (Claude Code, Cursor, etc.) guide developers through correct Userplane integration.

## Skills

| Skill | Description |
|-------|-------------|
| [userplane-react](skills/userplane-react/) | React (Vite) integration using a provider component |
| [userplane-nextjs](skills/userplane-nextjs/) | Next.js App Router integration with SSR-safe client provider |
| [userplane-vue](skills/userplane-vue/) | Vue 3 (Vite) integration using a Vue plugin |
| [userplane-angular](skills/userplane-angular/) | Angular integration using APP_INITIALIZER |
| [userplane-nuxt](skills/userplane-nuxt/) | Nuxt 3 integration using a browser-only client plugin |
| [userplane-astro](skills/userplane-astro/) | Astro integration using client-side script blocks |
| [userplane-sveltekit](skills/userplane-sveltekit/) | SvelteKit integration using onMount in root layout |
| [userplane-tanstack-start](skills/userplane-tanstack-start/) | TanStack Start integration with search param schema |
| [userplane-static-html](skills/userplane-static-html/) | Static HTML site integration (no build step) |
| [userplane-web-sdk](skills/userplane-web-sdk/) | Web SDK API reference — initialization, recorder control, recording state |
| [userplane-metadata-sdk](skills/userplane-metadata-sdk/) | Metadata SDK — attach user context to recordings |
| [userplane-sensitive-data](skills/userplane-sensitive-data/) | Sensitive data redaction — blur configuration and third-party compatibility |
| [userplane-cdn](skills/userplane-cdn/) | CDN installation — script placement, CSP, redirects, browser support |
| [userplane-best-practices](skills/userplane-best-practices/) | Cross-cutting best practices for installation, SDK integration, metadata, and privacy |

## Installation

### Using the skills CLI (recommended)

```bash
npx skills add wizenheimer/userplane-agent-skills
```

This installs all Userplane skills and makes them available to your agent automatically.

### Claude Code (manual)

Copy a skill into your Claude Code skills directory:

```bash
cp -r skills/userplane-react ~/.claude/skills/
```

Or copy all skills:

```bash
cp -r skills/* ~/.claude/skills/
```

### claude.ai Projects

Upload the `SKILL.md` file from any skill directory as project knowledge.

### Cursor / Other Agents

Add the `SKILL.md` content to your agent's context or rules file.
