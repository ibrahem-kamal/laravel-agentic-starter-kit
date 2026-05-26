# Action Required — {Feature Name}

Manual steps the human must complete. Agents cannot do these.

## Before implementation

{e.g. "Create a Stripe test account and add publishable + secret keys to .env"}
{e.g. "Provision a Cashier webhook endpoint in the Stripe dashboard pointing at $APP_URL/stripe/webhook"}

## During implementation

{e.g. "Add `STRIPE_KEY`, `STRIPE_SECRET`, `CASHIER_CURRENCY` to .env and .env.example"}

## After implementation

{e.g. "Run `php artisan cashier:webhook` once in production to register the live webhook"}
{e.g. "Verify Horizon picks up the `billing` queue — update config/horizon.php production supervisor if needed"}

---

If there are no manual steps required, replace this template with:

> No manual steps required.
