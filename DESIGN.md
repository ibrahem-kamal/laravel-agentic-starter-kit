# Design System

UI rules for Laravel projects using this starter kit. Two profiles supported — pick based on what's installed.

## Profile detection

- `livewire/livewire` in composer.json → **Livewire / Flux profile** (Section A)
- `inertiajs/inertia-laravel` in composer.json → **Vue / Inertia profile** (Section B)

Shared rules (typography, spacing, colour, accessibility) apply to both — see Section C.

---

## Section A — Livewire / Flux profile

### Primitives

Use **Flux v2** components for every interactive UI element. Do not hand-build a button, input, modal, badge, dropdown, or table when a `<flux:*>` exists for it.

Common primitives:

| Need | Component |
|------|-----------|
| Button | `<flux:button>` |
| Text input | `<flux:input>` |
| Textarea | `<flux:textarea>` |
| Select | `<flux:select>` |
| Checkbox | `<flux:checkbox>` |
| Modal | `<flux:modal>` |
| Card / container | `<flux:card>` |
| Badge / pill | `<flux:badge>` |
| Tooltip | `<flux:tooltip>` |
| Toast / notification | `<flux:toast>` |
| Data table | `<flux:table>` |
| Date picker | `<flux:date-picker>` |

Run the `fluxui-development` skill before composing components — it has the canonical syntax and variant catalogue.

### Composition

- Pages live in `resources/views/livewire/{area}/{page}.blade.php`
- Layout extends `<x-layouts.app>` (or whatever the project's root layout is)
- Inside a layout, structure pages as: page header → main content grid → optional sidebar/sidedrawer
- Cards group related actions; never stack more than 3 cards above the fold without a section heading

### Loading states

- Every action button shows `wire:loading` state — disable, swap label, or show spinner
- Long lists use `wire:loading.delay` to avoid flicker
- For LLM streaming use `wire:stream` (see `prism-ai` skill)

### Icons

Heroicons only (shipped with Flux). Use the `icon` attribute on Flux components when available:

```blade
<flux:button icon="trash" variant="danger">Delete</flux:button>
```

For standalone icons, use Blade's heroicon helper: `<x-heroicon-o-user />`.

---

## Section B — Vue / Inertia profile

### Primitives

Use **Headless UI** for accessible interactive primitives. Use **Heroicons Vue** for icons. No third UI kit (no Vuetify, no PrimeVue, no Naive UI).

| Need | Primitive |
|------|-----------|
| Button | Plain `<button>` with utility classes (no library needed) |
| Modal / Dialog | `<Dialog>` from `@headlessui/vue` |
| Dropdown | `<Menu>` from `@headlessui/vue` |
| Combobox / autocomplete | `<Combobox>` from `@headlessui/vue` |
| Tabs | `<TabGroup>` from `@headlessui/vue` |
| Disclosure / accordion | `<Disclosure>` from `@headlessui/vue` |
| Toggle | `<Switch>` from `@headlessui/vue` |
| Form input | `<input>` styled with utility classes, validated via `useForm` |

### Composition

- Pages live in `resources/js/Pages/{Area}/{Name}.vue`
- Share layouts via Inertia's persistent layout pattern: `defineOptions({ layout: AppLayout })`
- Use composables for reusable client logic (`resources/js/Composables/`)
- Use `useForm` from `@inertiajs/vue3` for every form — never bare `axios`/`fetch`

### Icons

Heroicons Vue:

```vue
import { TrashIcon } from '@heroicons/vue/24/outline'
```

Outline for navigation/labels (24x24), solid for emphasis or active states (20x20).

---

## Section C — Shared rules (both profiles)

### Tailwind v4

Tailwind v4 is the styling layer for both profiles. Conventions:

- Use utility classes directly on elements; avoid `@apply` in CSS files
- Custom design tokens live in `resources/css/app.css` as CSS custom properties under `@theme`:

```css
@import "tailwindcss";

@theme {
    --color-brand-50:  oklch(0.97 0.02 250);
    --color-brand-500: oklch(0.55 0.18 250);
    --color-brand-900: oklch(0.25 0.10 250);
}
```

- Tailwind generates `bg-brand-500`, `text-brand-900` etc automatically from `@theme` vars
- Don't add component classes (`.btn-primary { … }`) in CSS — use components instead (Flux, or Vue SFC)

### Dark mode

- Required, not optional. Every page must work in `dark:` mode.
- Use the system preference detector + a manual toggle persisted to localStorage.
- Test every new component in dark mode before declaring done.

### Spacing scale

Tailwind defaults (`gap-2`, `gap-4`, `gap-6`, `gap-8`). Don't add custom one-off spacings like `gap-[14px]` — pick the nearest scale value.

Page-level padding: `px-4 sm:px-6 lg:px-8`.

### Typography

- Headings: `text-2xl font-semibold` (h1), `text-xl font-semibold` (h2), `text-lg font-medium` (h3)
- Body: `text-sm` for dense UI (tables, panels), `text-base` for prose
- Avoid `font-bold` — use `font-semibold` for headings, `font-medium` for emphasis

### Colour

- Brand colour for primary CTAs only — never for backgrounds of decorative chrome
- Use neutral greys (slate / zinc / gray — pick one and stick) for everything except status and CTAs
- Status colours: green (success), amber (warning), red (danger/error), blue (info)

### Accessibility minimums

- Every interactive element has a visible focus ring (`focus-visible:ring-2 focus-visible:ring-brand-500`)
- Buttons and links have discernible text or `aria-label`
- Form inputs have associated `<label>` (not placeholder-only)
- Colour contrast ≥ 4.5:1 for body text, 3:1 for large text
- Modals trap focus and return it on close (Flux and Headless UI do this automatically — don't roll your own)
- Test keyboard navigation: tab order should match visual order

### Responsive design

Mobile-first. Default styles target mobile; use `sm:`, `md:`, `lg:` to expand. Breakpoints:

- `sm:` 640px — small tablets, large phones
- `md:` 768px — tablets
- `lg:` 1024px — small laptops
- `xl:` 1280px — desktops

Test every page at 375px (iPhone SE) before declaring done.

### Forms

- Required indicator: red asterisk in the label, not in the input
- Validation errors below the input, red text, with `aria-describedby` linking input to error
- Disable submit until the form is dirty AND valid (Livewire: `wire:dirty`, Vue: `useForm`'s `isDirty`)
- Show optimistic state on submit — disable button, swap label to "Saving…"

### Data display

- Empty states: a centred message + suggested action, not just blank space
- Loading states: skeleton screens for tables and lists, not spinners
- Pagination: 25 rows default, configurable via select; show total count

---

## When in doubt

- Run the `frontend-design` skill (Anthropic / community) for production-grade aesthetics
- Run `fluxui-development` or `tailwindcss-development` for syntax / component questions
- Match the existing project's tone — if it's a clinical SaaS, restraint; if a marketing site, energy
