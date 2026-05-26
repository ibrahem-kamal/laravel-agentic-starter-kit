# Laravel Agentic Starter Kit

A portable Claude Code skill bundle for **Laravel + Livewire/Flux** or **Laravel + Vue/Inertia** projects. Installable into any new or existing Laravel project via the [skills.sh](https://www.skills.sh) CLI.

Inspired by [`leonvanzyl/agentic-coding-starter-kit`](https://github.com/leonvanzyl/agentic-coding-starter-kit) (Next.js / React), this kit ports the **agent workflow** — spec → parallel implementation → review → ship — to the Laravel ecosystem. No PHP code is shipped; this is pure agent tooling.

## Install

```bash
npx skills add https://github.com/ibrahem-kamal/laravel-agentic-starter-kit --skill bootstrap-laravel-agentic
```

Run from inside the target Laravel project. The `bootstrap-laravel-agentic` skill will:

1. Verify it's a Laravel project (offer to scaffold one if not)
2. Detect frontend stack (Livewire vs Vue/Inertia) and ask if neither is present
3. Prompt to install **Laravel Boost** (`composer require laravel/boost --dev` + `php artisan boost:install`)
4. Run `graphify init .` if `graphify` is on PATH
5. Copy the bundled skills into `<project>/.claude/skills/`
6. Write `CLAUDE.agentic.md` and `DESIGN.md` (without overwriting any existing `CLAUDE.md`)

Final step you do yourself: add this line to your project's `CLAUDE.md` (the bootstrap will create one if missing):

```
@CLAUDE.agentic.md
```

## What's in the bundle

| Skill | Purpose |
|-------|---------|
| `bootstrap-laravel-agentic` | The installer (only invoked once per project) |
| `create-laravel-spec` | Plan a feature → `specs/{feature}/` with wave-organised, self-contained task files |
| `implement-laravel-feature` | Dispatch coder sub-agents wave-by-wave with Pint + Pest verification gates between waves |
| `prism-ai` | Prism PHP patterns: streaming in Livewire, SSE for Vue, tool calling, structured output, cost logging |
| `ship-laravel` | Pre-push checklist (Pint, Pest, npm build, graphify update) + PR title/body draft |
| `checkpoint` | Save WIP as a git commit mid-feature, never on the default branch |

Plus two files copied to your project root:

- **`CLAUDE.agentic.md`** — agent rules: planning discipline, verification gates, stack detection, AI conventions, skill catalogue
- **`DESIGN.md`** — UI design system covering Flux/Livewire and Vue/Inertia profiles, Tailwind v4 conventions, dark mode, accessibility

## What's *not* in the bundle

- ❌ No PHP code (no migrations, controllers, models, Livewire components, Vue pages, Fortify scaffolding)
- ❌ No AI chat demo / boilerplate features
- ❌ No skills that already exist in iki's global Claude config (`livewire-development`, `fluxui-development`, `fortify-development`, `pest-testing`, `laravel-best-practices`, `tailwindcss-development`, `frontend-design`). The bundle assumes those are available and `CLAUDE.agentic.md` references them.
- ❌ No CI / GitHub Actions templates — projects vary too much

## Workflow

```
┌─────────────────────────────────────────────────────────────┐
│  /bootstrap-laravel-agentic  (one-time per project)         │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  /create-laravel-spec                                       │
│    → specs/{feature}/                                       │
│       ├── README.md (waves, dependency graph, status)       │
│       ├── requirements.md                                   │
│       ├── action-required.md                                │
│       └── tasks/task-{nn}-*.md  (self-contained)            │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  /implement-laravel-feature                                 │
│    Wave 1: [task-01] [task-02] [task-03]   parallel agents  │
│            ↓                                                │
│         Pint + Pest gate                                    │
│            ↓                                                │
│         Code review                                         │
│            ↓                                                │
│         git commit -m "feat: wave 1"                        │
│            ↓                                                │
│    Wave 2: …                                                │
└─────────────────────────────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  /ship-laravel                                              │
│    Pint, Pest, npm build, graphify update                   │
│    → drafts PR title + body                                 │
│    → surfaces the exact `gh pr create` command              │
└─────────────────────────────────────────────────────────────┘
```

## Requirements

- **PHP 8.4+** (Laravel 13 baseline)
- **Composer**
- **Node 22+** (only if you use the frontend build step)
- **Claude Code** (the CLI tool — this is a Claude Code skill bundle)
- Optional: [`graphify`](https://github.com/anthropics/graphify) CLI for the knowledge-graph layer

## Manual install (without skills.sh)

If you don't want `npx skills add`:

```bash
git clone https://github.com/ibrahem-kamal/laravel-agentic-starter-kit /tmp/agentic
cp -R /tmp/agentic/.claude/skills/* <your-project>/.claude/skills/
cp /tmp/agentic/CLAUDE.agentic.md /tmp/agentic/DESIGN.md <your-project>/
```

Then in Claude Code, run `/bootstrap-laravel-agentic` inside your project to finish the setup (Boost prompt, graphify init, etc.).

## Philosophy

Three opinions baked into this kit:

1. **Specs before code.** Every feature larger than a one-liner gets a `specs/{feature}/` folder before any implementation. The spec is the source of truth.
2. **Parallel coder sub-agents, single orchestrator.** The main session coordinates; sub-agents implement. The orchestrator never writes code.
3. **Verification is non-negotiable.** Pint clean + Pest green + (optionally) `npm run build` ok, after every wave. No exceptions, no `--no-verify`.

If those opinions don't fit your workflow, fork the bundle and adjust the skill SKILL.md files. They're plain markdown — easy to edit.

## Contributing

Issues and PRs welcome. The bundle is meant to be small and opinionated; large new skills probably belong in their own package.

## Licence

MIT. Adapt freely.

---

🤖 Built with help from [Claude Code](https://claude.com/claude-code).
