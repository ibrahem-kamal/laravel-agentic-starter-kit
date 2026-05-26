# Agentic Coding Rules (Laravel)

This file is `@`-imported from your project's `CLAUDE.md`. It establishes how you (the coding agent) should approach work in this Laravel project. Project-specific rules in `CLAUDE.md` always override what's here.

## Planning mode

- **Always ask clarifying questions** before non-trivial work. Never assume tech stack, design choices, or feature scope.
- Use sub-agents (`Explore`, `Plan`) for research. Multiple in parallel when the queries are independent.
- For any feature that touches 3+ files or has multiple valid approaches, write a spec first via `/create-laravel-spec`. Skip for typos and one-line fixes.

## Stack detection

- If `composer.json` contains `livewire/livewire` → use the **Livewire profile**. Build UI with Livewire 4 components and Flux v2 primitives. Invoke `livewire-development`, `fluxui-development`, `tailwindcss-development` skills.
- If `composer.json` contains `inertiajs/inertia-laravel` → use the **Vue/Inertia profile**. Build UI with Vue 3 + Inertia. Invoke `frontend-design`, `tailwindcss-development` skills.
- If both are present, treat one as primary (check `.claude/agentic-profile`) and the other as an escape hatch for pages that don't fit the primary paradigm.
- For backend-only work (jobs, console commands, services, models, migrations) the profile doesn't matter — follow Laravel conventions.

## Implementation discipline

- **Never implement features in the main session if a sub-agent can do it.** For specs with multiple parallelisable tasks, dispatch via `/implement-laravel-feature`. Act as coordinator.
- Use `php artisan make:*` commands with `--no-interaction` to scaffold every new file. Don't hand-write what artisan generates.
- For non-auth model IDs use UUIDs: `$table->uuid('id')->primary()` + `HasUuids` trait.
- Form Requests for validation when controllers grow past a trivial inline `validate()`.

## Verification gates

After every feature (large or small), run **all three**:

```bash
vendor/bin/pint --dirty --format agent
php artisan test --compact
npm run build   # only if frontend files changed
```

If any fails, do not declare done. Investigate root cause — don't suppress with `--no-verify` or skip tests.

## Database changes

- `php artisan make:migration` then `php artisan migrate`. Never `migrate:fresh` on a shared branch.
- Never edit an existing migration after it's shipped to a branch others may have pulled. Write a new migration to alter.
- Always create a factory alongside a model: `php artisan make:model {Name} --factory`.

## Authorisation

- Use Policies or Gates for every model that has cross-tenant or cross-user access concerns.
- Test cross-tenant access explicitly in Pest: user from Tenant A getting 403 on Tenant B's resource is mandatory coverage.

## AI / LLM work

- Use Prism PHP via the `prism-ai` skill (bundled). Don't add another LLM SDK without strong justification.
- Always set `withMaxTokens()` and `withMaxSteps()` on Prism calls.
- For prompts longer than ~10s of work, dispatch to a queued Job. Don't block HTTP requests.
- Log token usage to a dedicated table for cost attribution.

## Testing

- Pest, not PHPUnit. `php artisan make:test --pest {Name}Test` to scaffold.
- Test names: `it('does the thing')` not `test_does_the_thing`.
- Use factory states for setup, not manual `create()` calls with literal data.
- One `it()` per behaviour. Don't pack multiple assertions about unrelated behaviours into one test.

## Skill catalogue (bundled)

| Skill | When |
|-------|------|
| `bootstrap-laravel-agentic` | First-time install or upgrade |
| `create-laravel-spec` | Plan a feature → spec folder with parallelisable tasks |
| `implement-laravel-feature` | Execute a spec wave-by-wave via sub-agents |
| `prism-ai` | Any LLM call (Anthropic, OpenAI, Gemini) |
| `ship-laravel` | Pre-push checklist + PR draft |
| `checkpoint` | Save WIP commit mid-feature |

## Recommended global skills (if available)

These ship with iki's global Claude config (or as Anthropic / community skills). Invoke them when relevant — don't duplicate their content into project code:

- `laravel-best-practices` — Laravel idioms, N+1, caching, security
- `livewire-development` — Livewire 4 specifics
- `fluxui-development` — Flux v2 components
- `fortify-development` — auth flows, 2FA, passkeys
- `pest-testing` — Pest 4 patterns including browser tests
- `tailwindcss-development` — Tailwind v4
- `frontend-design` — production-grade UI design (Vue / generic)
- `pr-review-toolkit:review-pr` — multi-agent PR review
- `superpowers:brainstorming`, `superpowers:systematic-debugging`, `superpowers:test-driven-development`

## graphify

If `graphify-out/` exists in this project, prefer `graphify query "<question>"` over raw grep for codebase questions. After modifying code, run `graphify update .` to keep the graph current.

If `graphify-out/wiki/index.md` exists, use it for navigation in place of broad file browsing.

## When the spec and the installed stack disagree

The spec (Trello card, planning doc, ticket) is binding for **behaviour and file structure**. The installed stack is binding for **package names and API surface**. If they conflict, use what's installed and adapt.

Examples of common drift:
- Spec says "Breeze" → use whatever auth scaffold is installed (Fortify, Jetstream, custom)
- Spec says "Livewire 3 syntax" → use Livewire 4
- Spec says "Tailwind v3 classes" → use v4
- Spec says "shadcn / Headless UI" → use whatever the project uses (Flux for Livewire, Headless UI for Vue)

When in doubt, run Laravel Boost's `search-docs` for version-pinned guidance before writing.

## Responses

- Be concise. Focus on what's important.
- State decisions and results directly. Don't narrate internal deliberation.
- Match response weight to task weight. Simple question = direct answer, not headers and sections.
