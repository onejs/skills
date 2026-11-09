# One Framework Skills

Official agent skills for building web and native apps with [One](https://onestack.dev).

## Install in your app

Run from the app root:

```bash
npx skills add onejs/skills
```

Select the One skills and your coding agents in the installer. For all six skills with Codex and Claude Code, without prompts:

```bash
npx skills add onejs/skills --skill '*' --agent codex claude-code --yes
```

Project installs keep skills under `.agents/skills` and create the selected agent's links where needed. Codex reads the shared `.agents/skills` directory. Use `--agent cursor` for Cursor, or select it interactively. Add `--global` only when you want these skills available in every project.

Confirm discovery with `npx skills list`. Start a fresh agent session after installing so its skill catalog includes the new skills. Ask it to use `building-with-one` when building or changing a One app, then the relevant topic skill. Adding the install command alone does not install anything.

Update a project install with `npx skills update --project --yes`. Check the installed `one` package version before using newer APIs: these skills follow current One source and can describe APIs not yet present in an older release.

### Claude Code plugin

The same skills are packaged as a Claude plugin:

```text
/plugin marketplace add onejs/skills
/plugin install one@onestack-plugins
```

Use either the plugin or a skills CLI install for Claude to avoid duplicate entries. The marketplace's identifier remains `onestack-plugins`; the GitHub repository is `onejs/skills`.

## Make agents use the skills

Add this to your app's `AGENTS.md` or `CLAUDE.md`:

```markdown
Before changing this One app, read the installed `building-with-one` skill and
its references for the task. If missing, run `npx skills add onejs/skills` from
the app root and select your agent. Use the relevant One topic skill for
loaders, API routes, render modes, deployment, or recipes. Check APIs against
the installed `one` version.
```

## Skills

| Skill | Use |
| --- | --- |
| `building-with-one` | Routing, layouts, tabs/toolbars, hooks, configuration, SwiftUI/Compose UI, native modifiers and Liquid Glass |
| `one-api-routes` | HTTP endpoints, method dispatch, typed params and worker context |
| `one-loaders` | Server data, refetch, redirects, query validation and caching |
| `one-render-modes` | SSG/SSR/SPA/API, folder precedence and layout shells |
| `one-deployment` | Node, Vercel, Workers, static output and native release boundaries |
| `one-guides` | Auth, images, CSS, color scheme, Tamagui, MDX, social metadata, native builds and troubleshooting |

Each `SKILL.md` routes to focused references. Read only those needed. Source of truth: [One docs](https://onestack.dev/docs/introduction), [native reference](https://onestack.dev/native/overview), and the [One source](https://github.com/onejs/one). Contributions should verify API names and behavior against source and keep entrypoints lean.

## License

MIT
