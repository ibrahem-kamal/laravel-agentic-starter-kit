# Recommended global skills catalogue

Skills referenced by `CLAUDE.agentic.md` that the bundle does NOT ship copies of. The bootstrap skill reads this file at Step 8 to decide what to offer the user.

Each entry has:
- `name` — skill identifier (the value of `name:` in its SKILL.md frontmatter)
- `description` — one sentence shown in the multi-select prompt
- `source` — where it comes from (Boost / skills.sh URL / search-skills.sh / etc.)
- `install` — exact `npx skills add ...` command if a canonical URL is known; otherwise `search skills.sh`
- `notes` — anything the bootstrap should know about detection or install order

The list is intentionally small. If you want more skills, edit this file and re-run the bootstrap with `--force`.

---

## Ships with Laravel Boost

If the user installed Boost at Step 3 of the bootstrap, treat these as present (Boost registers them under `~/.claude/skills/` or via its own plugin namespace). Don't offer to install them separately — they're already there.

### laravel-best-practices
- description: Laravel idioms for controllers, models, queries, caching, auth, validation, jobs, routes. Apply when writing or reviewing Laravel PHP.
- source: Laravel Boost
- install: `composer require laravel/boost --dev && php artisan boost:install` (Step 3 of the bootstrap)

### livewire-development
- description: Livewire 3/4 components, wire: directives, reactivity patterns, drag-and-drop, real-time validation.
- source: Laravel Boost
- install: (Boost)

### fluxui-development
- description: Flux v2 components for Livewire — buttons, inputs, modals, tables, date pickers, kanban, badges.
- source: Laravel Boost
- install: (Boost)

### fortify-development
- description: Auth flows (login, register, password reset, email verification, 2FA, passkeys) via Laravel Fortify.
- source: Laravel Boost
- install: (Boost)

### pest-testing
- description: Pest PHP test syntax, datasets, mocking, browser tests, architecture tests for Laravel.
- source: Laravel Boost
- install: (Boost)

### tailwindcss-development
- description: Tailwind v3/v4 utility classes, responsive grids, dark mode, theming.
- source: Laravel Boost
- install: (Boost)

---

## Not in Boost — offer to install

These don't ship with Boost. Offer them in the multi-select.

### frontend-design
- description: Anthropic's production-grade UI design skill. Generates polished, distinctive interfaces (avoids generic AI aesthetics). Use for any new component or page.
- source: Anthropic skill catalogue
- install: `npx skills add https://github.com/anthropics/skills --skill frontend-design`
- notes: If install fails (URL not yet exact), print: "search skills.sh for 'frontend-design'".

### superpowers:brainstorming
- description: Required-before-creative-work skill. Explores user intent, requirements, and design alternatives before any implementation.
- source: obra/superpowers
- install: `npx skills add https://github.com/obra/superpowers --skill brainstorming`

### superpowers:systematic-debugging
- description: Disciplined debugging workflow — hypothesis, reproduce, isolate, verify. Use for any bug or test failure before proposing a fix.
- source: obra/superpowers
- install: `npx skills add https://github.com/obra/superpowers --skill systematic-debugging`

### superpowers:test-driven-development
- description: TDD workflow — write the failing test first, then the implementation, then refactor. Pairs well with `implement-laravel-feature`.
- source: obra/superpowers
- install: `npx skills add https://github.com/obra/superpowers --skill test-driven-development`

### superpowers:writing-plans
- description: Structured planning skill — produces an executable plan file before touching code. Lighter-weight alternative to `create-laravel-spec` for solo features.
- source: obra/superpowers
- install: `npx skills add https://github.com/obra/superpowers --skill writing-plans`

### superpowers:dispatching-parallel-agents
- description: General-purpose parallel sub-agent dispatch. Useful outside the spec → wave workflow.
- source: obra/superpowers
- install: `npx skills add https://github.com/obra/superpowers --skill dispatching-parallel-agents`

### pr-review-toolkit:review-pr
- description: Multi-agent PR review (code review, type design, comment quality, silent-failure hunter, test coverage).
- source: skills.sh — search "pr-review-toolkit"
- install: search skills.sh
- notes: URL not pinned in this kit; URLs for this toolkit have moved. Print the search hint.

---

## Detection rules

For each entry, the bootstrap should check (in order):

1. `<target-project>/.claude/skills/<name>/SKILL.md` — project-local install
2. `$HOME/.claude/skills/<name>/SKILL.md` — user-global install
3. For namespaced (`plugin:skill`) entries, also check `<target>/.claude/skills/<plugin>/<skill>/` and `$HOME/.claude/skills/<plugin>/<skill>/`
4. For Boost-bundled entries, treat as present if `vendor/laravel/boost/composer.json` exists in the target project

If none of those match, the skill is missing — offer it in the multi-select.

## Updating this catalogue

This file is hand-maintained. To add a skill:

1. Add an entry in the right section (`Ships with Boost` or `Not in Boost`)
2. Reference it from `CLAUDE.agentic.md` in the project root if it deserves a permanent mention
3. Open a PR against the starter-kit repo so others get it too
