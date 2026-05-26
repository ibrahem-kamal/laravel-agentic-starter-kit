# Task {nn} — {Task Name}

Status: `pending` | `in_progress` | `complete`

Wave: {N}

Depends on: {task numbers, or "none"}
Blocks: {task numbers, or "none"}

Stack profile: `livewire` | `vue` | `backend-only`

## Description

{What this task builds and why it matters in the context of the broader feature. 2–4 sentences.}

## Dependency context

{Prose summary of what previously completed tasks produce that THIS task needs. The coder agent must not need to read other task files. Include: filenames, table names, model names, method signatures, enum values, route names — whatever this task references from prior work. If no dependencies, write "First-wave task — no prior context required."}

## Files to create

- `path/to/file.php` — {purpose}
- `path/to/another.blade.php` — {purpose}

## Files to modify

- `path/to/existing.php` — {what changes and why}

## Technical details

Include ONLY the subsections relevant to this task. Delete the others.

### Artisan commands

Exact commands to scaffold files. Use `--no-interaction` and the right flags.

```bash
php artisan make:migration create_subscriptions_table
php artisan make:model Subscription --factory
php artisan make:policy SubscriptionPolicy --model=Subscription
```

### Migration

```php
Schema::create('subscriptions', function (Blueprint $table) {
    $table->uuid('id')->primary();
    $table->foreignUuid('user_id')->constrained()->cascadeOnDelete();
    $table->string('stripe_id')->unique();
    $table->string('tier'); // 'starter' | 'pro' | 'enterprise'
    $table->timestamp('cancelled_at')->nullable();
    $table->timestamps();
});
```

Indexes: `(user_id, cancelled_at)` composite for active-subscription lookup.

### Model + Factory

- `fillable`: `['user_id', 'stripe_id', 'tier', 'cancelled_at']`
- Traits: `HasUuids`, `HasFactory`
- Casts: `'cancelled_at' => 'datetime'`
- Relationships: `user()` belongs-to
- Factory states: `active()`, `cancelled()`

### Authorisation

- Policy file: `app/Policies/SubscriptionPolicy.php`
- Methods: `view`, `cancel` — both require `$user->id === $subscription->user_id`
- Register in `AuthServiceProvider` (or auto-discovery if model is conventionally placed)

### UI — Livewire profile

- Component: `App\Livewire\Billing\SubscriptionPanel`
- View: `resources/views/livewire/billing/subscription-panel.blade.php`
- Flux components to use: `<flux:card>`, `<flux:button variant="danger">`, `<flux:badge>`
- Wire directives: `wire:click="cancel"`, `wire:loading` for the cancel button
- Mount: pass current `Subscription` via dependency injection

### UI — Vue/Inertia profile

- Page: `resources/js/Pages/Billing/Show.vue`
- Route: `Route::get('/billing', [BillingController::class, 'show'])->name('billing.show')`
- Props from controller: `subscription: { id, tier, cancelled_at }`
- Headless UI primitives: `Dialog` for cancel confirmation
- Inertia form helper: `useForm({}).post(route('billing.cancel'))`

### Routes

```php
Route::middleware(['auth', 'clinic-admin'])->group(function () {
    Route::get('/billing', [BillingController::class, 'show'])->name('billing.show');
    Route::post('/billing/cancel', [BillingController::class, 'cancel'])->name('billing.cancel');
});
```

### Pest test

File: `tests/Feature/BillingCancellationTest.php`

Required assertions:
- HTTP 200 from `/billing` for authorised user
- HTTP 403 for unauthorised user (cross-tenant access)
- Cancelling sets `cancelled_at` to now
- Cancelling dispatches `SubscriptionCancelled` event
- Pest test uses `RefreshDatabase` and the model's factory states

Skeleton:

```php
it('lets owner cancel their subscription', function () {
    $user = User::factory()->create();
    $sub = Subscription::factory()->for($user)->active()->create();

    actingAs($user)
        ->post(route('billing.cancel'))
        ->assertRedirect(route('billing.show'));

    expect($sub->fresh()->cancelled_at)->not->toBeNull();
});
```

## Acceptance criteria

Specific, verifiable bullets:

- [ ] Migration runs cleanly with `php artisan migrate:fresh`
- [ ] `Subscription` factory + `active()` and `cancelled()` states work in tinker
- [ ] Authorised user can GET `/billing` and see their subscription
- [ ] Unauthorised user gets 403 (cross-tenant)
- [ ] POST `/billing/cancel` sets `cancelled_at` and dispatches the event
- [ ] All Pest tests in `BillingCancellationTest.php` pass
- [ ] `vendor/bin/pint --dirty --format agent` reports no changes
