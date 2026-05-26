# Requirements — {Feature Name}

## Purpose

{Why this feature exists. The user-visible problem it solves or the business outcome it enables. One paragraph.}

## Stack profile

`livewire` | `vue` | `backend-only`

Detected from `.claude/agentic-profile` / `composer.json`. UI tasks in this spec target this profile.

## Acceptance criteria

Bulleted list of verifiable conditions that, taken together, mean the feature is done.

- {e.g. "Logged-in clinic admin can navigate to /billing and see current subscription tier"}
- {e.g. "Cancelling a subscription marks `subscriptions.cancelled_at` and disables paid features within 1 request cycle"}

## Technical decisions

Decisions already made during planning that must be carried into implementation. Capture every concrete detail — package names, model names, route names, env vars, library choices, third-party services.

- **Models**: `Subscription` (new), `User` (modify — add `subscription_id`)
- **Routes**: `/billing` (GET, name `billing.show`), `/billing/cancel` (POST, name `billing.cancel`)
- **AI**: Prism PHP with Anthropic provider, model `claude-sonnet-4-6`
- **Queue**: jobs dispatched on the `billing` queue; ensure Horizon supervisor covers it
- **Authorisation**: only users with role `clinic_admin` can access; enforced via `EnsureClinicAdmin` middleware

## Out of scope

Things that came up during planning but explicitly are NOT part of this feature.

- {e.g. "Refund handling — separate spec"}
- {e.g. "Annual billing — current iteration is monthly only"}

## Open questions

Anything still unresolved at spec time. The implementer should flag these before starting if they materially affect the work.

- {e.g. "Do we expose the cancellation reason field publicly or internally only?"}
