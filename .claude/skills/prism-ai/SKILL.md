---
name: prism-ai
description: >
  Build LLM features in Laravel with Prism PHP. Use when the task involves generating text from
  Claude/OpenAI/Gemini, streaming responses to a Livewire component or Vue/Inertia page, tool
  use / function calling, structured output, or AI cost logging. Trigger phrases: "add AI", "use
  Claude", "stream a response", "chat component", "LLM tool call", "Prism", "/prism-ai".
---

# Prism AI

Build LLM features in Laravel using [Prism PHP](https://prismphp.com) — a Laravel-native multi-provider abstraction (Anthropic, OpenAI, Gemini, Ollama). One config, swap providers via env. Mirrors what the Vercel AI SDK gives Next.js projects, in idiomatic Laravel.

This skill teaches the **patterns** Prism users get wrong (streaming inside Livewire, tool definitions, structured output, cost logging). For API surface details, run Laravel Boost's `search-docs` tool with queries like `["prism text generation", "prism streaming", "prism tool calling"]` — it returns version-pinned docs.

## When to use

- Adding any AI/LLM call to a Laravel app
- Streaming an LLM response into a Livewire component or Vue page
- Defining tools for the LLM to call (function calling)
- Returning structured output (typed objects, not raw text)
- Logging cost/usage per request

## Install

```bash
composer require prism-php/prism
php artisan vendor:publish --tag=prism-config
```

Add provider keys to `.env`:

```dotenv
ANTHROPIC_API_KEY=
OPENAI_API_KEY=
GEMINI_API_KEY=
```

`config/prism.php` ships with reasonable defaults. Override per-environment via env vars.

## Pattern 1 — Basic text generation

Keep LLM calls **inside dedicated service classes**, not controllers or Livewire components. Easier to test, mock, and swap.

```php
namespace App\Services\AI;

use Prism\Prism\Facades\Prism;
use Prism\Prism\Enums\Provider;

class SymptomSummariser
{
    public function summarise(string $patientNotes): string
    {
        $response = Prism::text()
            ->using(Provider::Anthropic, config('ai.summariser.model'))
            ->withSystemPrompt(view('prompts.symptom-summariser'))
            ->withPrompt($patientNotes)
            ->withMaxTokens(800)
            ->asText();

        return $response->text;
    }
}
```

- Put prompts in Blade views (`resources/views/prompts/*.blade.php`) so they can use `@include`, variables, and stay diffable
- Pin the model name in `config/ai.php`, not in the service — makes per-env overrides easy
- Always set `withMaxTokens()` — runaway responses are the #1 cost surprise

## Pattern 2 — Streaming into Livewire

Livewire 4 supports server-sent updates via `wire:stream`. Combine with Prism's `asStream()`:

```php
// app/Livewire/AiChat.php
namespace App\Livewire;

use Livewire\Component;
use Prism\Prism\Facades\Prism;
use Prism\Prism\Enums\Provider;

class AiChat extends Component
{
    public string $prompt = '';
    public string $response = '';

    public function send(): void
    {
        $this->response = '';

        $stream = Prism::text()
            ->using(Provider::Anthropic, config('ai.chat.model'))
            ->withPrompt($this->prompt)
            ->asStream();

        foreach ($stream as $chunk) {
            $this->stream(to: 'response', content: $chunk->text, replace: false);
            $this->response .= $chunk->text;
        }
    }

    public function render()
    {
        return view('livewire.ai-chat');
    }
}
```

The view:

```blade
<flux:card>
    <flux:input wire:model="prompt" placeholder="Ask..." />
    <flux:button wire:click="send">Send</flux:button>

    <div wire:stream="response" class="prose mt-4">{{ $response }}</div>
</flux:card>
```

Notes:
- `wire:stream="response"` is the live target; the property name must match
- `replace: false` appends chunks; `replace: true` overwrites
- For long streams, also call `$this->dispatch('stream-tick')` periodically to keep the browser awake

## Pattern 3 — Streaming into Vue/Inertia

Inertia doesn't support server-streamed component updates. Stream via a separate SSE endpoint and consume with `EventSource` in Vue:

```php
// routes/web.php
Route::get('/ai/chat/stream', function (Request $request) {
    return response()->stream(function () use ($request) {
        $stream = Prism::text()
            ->using(Provider::Anthropic, config('ai.chat.model'))
            ->withPrompt($request->string('prompt'))
            ->asStream();

        foreach ($stream as $chunk) {
            echo "data: " . json_encode(['text' => $chunk->text]) . "\n\n";
            ob_flush();
            flush();
        }
    }, 200, [
        'Content-Type' => 'text/event-stream',
        'Cache-Control' => 'no-cache',
        'X-Accel-Buffering' => 'no',
    ]);
})->middleware('auth');
```

```vue
<!-- resources/js/Pages/AiChat.vue -->
<script setup>
import { ref } from 'vue'

const prompt = ref('')
const response = ref('')

function send() {
    response.value = ''
    const es = new EventSource(`/ai/chat/stream?prompt=${encodeURIComponent(prompt.value)}`)
    es.onmessage = (e) => {
        response.value += JSON.parse(e.data).text
    }
    es.onerror = () => es.close()
}
</script>
```

Gotcha: `X-Accel-Buffering: no` is required when nginx is in front, otherwise the response buffers until complete.

## Pattern 4 — Tool / function calling

Define tools as PHP classes implementing Prism's `Tool` contract. Keep tools **side-effect-free where possible** and idempotent — the LLM may call the same tool multiple times.

```php
use Prism\Prism\Tool;

class SearchPatientsTool extends Tool
{
    public function __construct()
    {
        $this->as('search_patients')
            ->for('Search patients by name or email')
            ->withStringParameter('query', 'The search term', required: true)
            ->using(fn (string $query) => Patient::search($query)->limit(5)->get()->toArray());
    }
}

$response = Prism::text()
    ->using(Provider::Anthropic, config('ai.chat.model'))
    ->withPrompt('Find patients named Ahmed')
    ->withTools([new SearchPatientsTool])
    ->withMaxSteps(3) // cap tool-call loops
    ->asText();
```

- Always set `withMaxSteps()` — without it, a stuck tool loop can run forever
- Validate tool parameters in the `using` callback; do not trust the LLM to honour types
- For tools that mutate state (write to DB, call external APIs), wrap in a Policy/Gate check using the authenticated user from the request

## Pattern 5 — Structured output

Return typed objects, not free-form text, when downstream code needs to consume the result:

```php
use Prism\Prism\Schema\ObjectSchema;
use Prism\Prism\Schema\StringSchema;
use Prism\Prism\Schema\NumberSchema;

$schema = new ObjectSchema(
    name: 'triage_decision',
    description: 'Triage outcome for a patient symptom report',
    properties: [
        new StringSchema('severity', 'low | medium | high | critical'),
        new NumberSchema('confidence', '0–1 confidence score'),
        new StringSchema('reasoning', 'One paragraph reasoning'),
    ],
    requiredFields: ['severity', 'confidence', 'reasoning'],
);

$response = Prism::structured()
    ->using(Provider::Anthropic, config('ai.triage.model'))
    ->withSchema($schema)
    ->withPrompt($symptomReport)
    ->asStructured();

$decision = $response->structured; // typed array matching schema
```

## Pattern 6 — Cost & usage logging

Every Prism response exposes `$response->usage->promptTokens`, `completionTokens`, and (for some providers) `cost`. Log these to a `ai_usage_logs` table keyed by user and feature so you can attribute spend.

```php
$response = Prism::text()->...->asText();

AiUsageLog::create([
    'user_id'           => auth()->id(),
    'feature'           => 'symptom_summariser',
    'provider'          => 'anthropic',
    'model'             => $response->meta->model,
    'prompt_tokens'     => $response->usage->promptTokens,
    'completion_tokens' => $response->usage->completionTokens,
]);
```

Create a `php artisan ai:cost-report` console command that aggregates by feature for monthly invoicing or budgeting.

## Testing

Prism ships a `Prism::fake()` helper. Use it in Pest tests so you never hit a real provider in CI:

```php
use Prism\Prism\Facades\Prism;
use Prism\Prism\Testing\TextResponseFake;

it('summarises symptoms', function () {
    Prism::fake([
        TextResponseFake::make()->withText('Patient reports headache and nausea.'),
    ]);

    $result = app(SymptomSummariser::class)->summarise('long notes...');

    expect($result)->toBe('Patient reports headache and nausea.');
});
```

For tool-call testing, fake the tool call sequence with `ToolCallResponseFake`. Always assert on `$response->usage` for cost-sensitive tests.

## Critical rules

- Never commit API keys. Always reference `env('ANTHROPIC_API_KEY')` via `config/prism.php`.
- Wrap every Prism call in a try/catch for `Prism\Exceptions\PrismException`. Surface to the user as a friendly retry, never as a stack trace.
- For long-running prompts (`>10s`), dispatch to a queued Job and notify the user via broadcast or Livewire polling. Don't block HTTP requests.
- Use Boost's `search-docs` with queries `["prism streaming", "prism tools", "prism structured"]` before writing non-trivial Prism code — the API evolves and this skill may lag.
