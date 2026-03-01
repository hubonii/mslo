# 🏋️ Musclo — Laravel Masterclass: Phase 5 (Final!)

---

# PHASE 5: DeepSeek R1 AI Integration + ExerciseDB Import

> **Goals:**
> 1. Build an AI coaching service using the **Service Pattern** (best practice — keeps controllers thin)
> 2. Create the ExerciseDB import command to fill the database with 1,500+ real exercises with animated GIFs

---

## Step 5.1 — Create the DeepSeek Config File

Laravel uses config files to manage settings. Instead of hardcoding API keys in the service, we read them from `.env` through a config file. This makes your code portable — the same code runs in development and production with different keys.

**File:** `config/deepseek.php` — **Create this file** (it doesn't exist yet):

```php
<?php

return [
    'api_key'  => env('DEEPSEEK_API_KEY', ''),
    'base_url' => env('DEEPSEEK_BASE_URL', 'https://api.deepseek.com'),
    'model'    => env('DEEPSEEK_MODEL', 'deepseek-reasoner'),
];
```

> **How this works:**
> - `env('DEEPSEEK_API_KEY', '')` — reads `DEEPSEEK_API_KEY` from `.env`. If it's missing, defaults to empty string.
> - Later, in our service, we call `config('deepseek.api_key')` to access it.
> - **Why not read `.env` directly in the service?** Because Laravel caches config for performance in production (`php artisan config:cache`). If you use `env()` in random PHP files, it won't work after caching. Using `config()` always works. This is a **very common beginner mistake!**

Make sure your `.env` file has (we added this in Phase 1):

```env
DEEPSEEK_API_KEY=your_deepseek_key_here
DEEPSEEK_MODEL=deepseek-reasoner
```

> **Don't have a DeepSeek API key yet?** Go to [platform.deepseek.com](https://platform.deepseek.com), sign up, and generate a key. For testing, you can leave it empty — the service will return a friendly error message.

---

## Step 5.2 — Build the DeepSeekService

> **What is the Service Pattern?** Instead of cramming API call logic into a controller (messy, hard to test), we create a dedicated Service class. The controller asks the service: "Hey, ask the AI this question." The service handles all the dirty work: building prompts, making HTTP requests, handling errors.
>
> This is how professional Laravel apps are structured. It follows the **Single Responsibility Principle** — the controller handles HTTP, the service handles business logic.

**File:** `app/Services/DeepSeekService.php` — **Create this file:**

```php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;

class DeepSeekService
{
    private string $apiKey;
    private string $baseUrl;
    private string $model;

    public function __construct()
    {
        $this->apiKey  = config('deepseek.api_key');
        $this->baseUrl = config('deepseek.base_url', 'https://api.deepseek.com');
        $this->model   = config('deepseek.model', 'deepseek-reasoner');
    }

    /**
     * Build the system prompt that tells the AI WHO it is and WHAT context it has.
     *
     * The system prompt is like giving the AI its "personality" and "knowledge."
     * We inject the user's current workout data so the AI can give personalized advice.
     */
    private function buildSystemPrompt(array $context): string
    {
        $base = "You are Musclo AI Coach, an expert personal trainer and exercise scientist. "
            . "You provide advice grounded in progressive overload, periodization, and exercise science. "
            . "Be concise, actionable, and encouraging. Use data provided to give specific recommendations. "
            . "Never recommend dangerous overloads. Always prioritize form and safety.";

        // Inject the user's workout context (exercise, weight, reps, etc.)
        if (!empty($context)) {
            $base .= "\n\n--- USER CONTEXT ---\n";

            if (isset($context['exercise'])) {
                $base .= "Current Exercise: {$context['exercise']}\n";
            }
            if (isset($context['weight'])) {
                $base .= "Current Weight: {$context['weight']}kg\n";
            }
            if (isset($context['reps'])) {
                $base .= "Current Reps: {$context['reps']}\n";
            }
            if (isset($context['rir'])) {
                $base .= "Reps in Reserve (RIR): {$context['rir']}\n";
            }
            if (isset($context['history_summary'])) {
                $base .= "Recent Performance Summary: {$context['history_summary']}\n";
            }
        }

        return $base;
    }

    /**
     * Send a message to DeepSeek R1 and return the response.
     *
     * @param string $message — The user's question ("Should I go heavier?")
     * @param array $context  — Workout data from the frontend (exercise, weight, reps, etc.)
     * @return array — ['advice' => string, 'suggestions' => string[]] on success
     *                  ['error' => string] on failure
     */
    public function ask(string $message, array $context = []): array
    {
        $systemPrompt = $this->buildSystemPrompt($context);

        try {
            $response = Http::withToken($this->apiKey)
                ->timeout(30)                       // Wait max 30 seconds
                ->retry(2, 1000)                    // Retry once after 1 second if it fails
                ->post("{$this->baseUrl}/v1/chat/completions", [
                    'model'    => $this->model,
                    'messages' => [
                        ['role' => 'system', 'content' => $systemPrompt],
                        ['role' => 'user',   'content' => $message],
                    ],
                    'temperature' => 0.7,            // Creativity level (0 = robotic, 1 = wild)
                    'max_tokens'  => 800,             // Max response length
                ]);

            if ($response->successful()) {
                $content = $response->json('choices.0.message.content', '');
                return [
                    'advice'      => $content,
                    'suggestions' => $this->extractSuggestions($content),
                ];
            }

            // If the API returned an error (500, 429, etc.)
            Log::error('DeepSeek API error', [
                'status' => $response->status(),
                'body'   => $response->body(),
            ]);

            return ['error' => 'AI service returned an error. Please try again.'];

        } catch (\Illuminate\Http\Client\ConnectionException $e) {
            // Timeout or network error
            Log::error('DeepSeek connection timeout', [
                'exception' => $e->getMessage(),
            ]);
            return ['error' => 'AI service is temporarily unavailable. Please try again in a moment.'];
        }
    }

    /**
     * Generate follow-up suggestions based on the AI response.
     * In a more advanced version, you'd parse the AI's response to generate
     * context-aware suggestions. For now, we return smart defaults.
     */
    private function extractSuggestions(string $content): array
    {
        return [
            'Should I increase the weight next session?',
            'What accessories complement this exercise?',
            'How should I adjust my rest periods?',
        ];
    }
}
```

> **Let me trace the entire flow from frontend to AI:**
>
> ```
> 1. User is mid-workout doing Bench Press at 80kg × 8 reps
> 2. They tap the AI Chat button and ask: "Should I go heavier?"
> 3. React sends POST /api/chat:
>    {
>      "message": "Should I go heavier?",
>      "context": {
>        "exercise": "Barbell Bench Press",
>        "weight": 80,
>        "reps": 8,
>        "rir": 2,
>        "history_summary": "Last 3 sessions: 75kg×8, 77.5kg×8, 80kg×8"
>      }
>    }
> 4. Laravel validates via AskAIRequest ✅
> 5. AICoachController calls $this->deepSeek->ask(message, context)
> 6. DeepSeekService builds the system prompt with context injected
> 7. DeepSeek R1 receives:
>    System: "You are Musclo AI Coach... Current Exercise: Barbell Bench Press,
>            Current Weight: 80kg, Current Reps: 8, RIR: 2,
>            Recent Performance: Last 3 sessions: 75→77.5→80kg"
>    User: "Should I go heavier?"
> 8. DeepSeek responds: "Great progress! You've consistently added 2.5kg
>    per session. With 2 RIR, you have room to grow. Try 82.5kg × 8 next
>    session. If you can only get 6 reps, stay at 82.5kg until you hit 8."
> 9. Service returns { advice: "...", suggestions: [...] }
> 10. Controller sends it back to React as JSON
> 11. React displays it in the chat modal with streaming animation ✨
> ```

---

## Step 5.3 — AICoachController

**File:** `app/Http/Controllers/AICoachController.php`

Replace the entire file:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\AskAIRequest;
use App\Services\DeepSeekService;
use Illuminate\Http\JsonResponse;

class AICoachController extends Controller
{
    /**
     * Constructor Dependency Injection.
     * Laravel automatically creates a DeepSeekService instance and passes it here.
     * This is called "Dependency Injection" — we don't write "new DeepSeekService()"
     * ourselves. Laravel's Service Container handles it.
     */
    public function __construct(private DeepSeekService $deepSeek)
    {
    }

    /**
     * POST /api/chat
     * Ask the AI coach a question with optional workout context.
     */
    public function ask(AskAIRequest $request): JsonResponse
    {
        $result = $this->deepSeek->ask(
            $request->validated('message'),
            $request->validated('context', [])
        );

        // If the service returned an error, send 503 (Service Unavailable)
        if (isset($result['error'])) {
            return response()->json(['error' => $result['error']], 503);
        }

        return response()->json(['data' => $result]);
    }
}
```

> **`private DeepSeekService $deepSeek`** in the constructor is PHP 8's "constructor property promotion." This single line:
> 1. Declares a private property called `$deepSeek`
> 2. Sets its type to `DeepSeekService`
> 3. Accepts it as a constructor argument
> 4. Assigns it to `$this->deepSeek`
>
> All in ONE line. Without this feature, you'd need 5 lines of code!
>
> **503 status code** means "Service Unavailable" — the client should retry later. This tells the React frontend: "Don't show an error, show a 'try again' button."

---

## Step 5.4 — ExerciseDB Import Command

This is the command that fills your database with **1,500+ real exercises** including animated GIF URLs from ExerciseDB.

### First, create the ExerciseDB Service:

**File:** `app/Services/ExerciseDBService.php` — **Create this file:**

```php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Http;
use Illuminate\Support\Facades\Log;

class ExerciseDBService
{
    private string $baseUrl = 'https://www.exercisedb.dev/api/v1';
    private int $limit = 25; // API max per request

    /**
     * Fetch all exercises from ExerciseDB API (paginated).
     * Uses a Generator to yield batches — saves memory.
     *
     * Instead of loading 1500 exercises into memory at once,
     * it processes 25 at a time. This is critical for large datasets!
     */
    public function fetchAllExercises(): \Generator
    {
        $offset = 0;
        $totalPages = 1;

        do {
            $response = Http::timeout(30)
                ->retry(3, 2000)  // Retry 3 times, wait 2 seconds between retries
                ->get("{$this->baseUrl}/exercises", [
                    'limit'  => $this->limit,
                    'offset' => $offset,
                ]);

            if (!$response->successful()) {
                Log::error('ExerciseDB API error', [
                    'status' => $response->status(),
                    'offset' => $offset,
                ]);
                break;
            }

            $json = $response->json();
            $totalPages  = $json['metadata']['totalPages'] ?? 1;
            $currentPage = $json['metadata']['currentPage'] ?? 1;

            yield $json['data'] ?? [];

            $offset += $this->limit;
        } while ($currentPage < $totalPages);
    }
}
```

> **`\Generator` + `yield`** — this is an advanced PHP concept. Instead of returning ALL exercises at once (which would use tons of memory), the method "yields" them in batches of 25. Each time the `foreach` loop asks for the next batch, the method resumes where it left off and fetches the next page. Extremely memory-efficient! 🧠

### Now create the Artisan command:

```bash
php artisan make:command ImportExercisesCommand
```

**File:** `app/Console/Commands/ImportExercisesCommand.php`

Replace the entire file:

```php
<?php

namespace App\Console\Commands;

use App\Models\Exercise;
use App\Services\ExerciseDBService;
use Illuminate\Console\Command;

class ImportExercisesCommand extends Command
{
    protected $signature = 'exercises:import
        {--fresh : Delete all non-custom exercises before importing}';

    protected $description = 'Import exercises from ExerciseDB API (1500+ exercises with GIFs)';

    public function handle(ExerciseDBService $service): int
    {
        // If --fresh flag is passed, clear out old exercises first
        if ($this->option('fresh')) {
            $deleted = Exercise::where('is_custom', false)->delete();
            $this->info("🗑️ Deleted {$deleted} existing global exercises.");
        }

        $this->info('📡 Fetching exercises from ExerciseDB API...');
        $this->info('   This may take a few minutes (1500+ exercises).');
        $this->newLine();

        $bar = $this->output->createProgressBar();
        $bar->start();

        $imported = 0;
        $updated  = 0;

        foreach ($service->fetchAllExercises() as $batch) {
            foreach ($batch as $exerciseData) {
                $result = Exercise::updateOrCreate(
                    // Find exercise by ExerciseDB ID (so we can update later)
                    ['exercisedb_id' => $exerciseData['exerciseId']],
                    // The data to insert or update
                    [
                        'name'              => $exerciseData['name'],
                        'muscle_group'      => $exerciseData['targetMuscles'][0] ?? 'Other',
                        'secondary_muscles' => implode(', ', $exerciseData['secondaryMuscles'] ?? []),
                        'equipment'         => $exerciseData['equipments'][0] ?? null,
                        'body_part'         => $exerciseData['bodyParts'][0] ?? null,
                        'category'          => 'strength',
                        'instructions'      => implode("\n", $exerciseData['instructions'] ?? []),
                        'gif_url'           => $exerciseData['gifUrl'] ?? null,
                        'is_custom'         => false,
                        'user_id'           => null,
                    ]
                );

                $result->wasRecentlyCreated ? $imported++ : $updated++;
                $bar->advance();
            }
        }

        $bar->finish();
        $this->newLine(2);
        $this->info("✅ Done! Imported: {$imported}, Updated: {$updated}");

        return Command::SUCCESS;
    }
}
```

> **Key concepts:**
>
> | Concept | Explanation |
> |---|---|
> | `$signature` | Defines the command name and its options. You run it as `php artisan exercises:import` |
> | `{--fresh}` | An optional flag. Run `php artisan exercises:import --fresh` to delete old exercises first |
> | `updateOrCreate()` | If an exercise with this `exercisedb_id` exists → update it. If not → create it. This means you can run the command multiple times safely! No duplicates. |
> | `$result->wasRecentlyCreated` | Returns `true` if the record was just created, `false` if it was updated. Used for the progress counter. |
> | `$this->output->createProgressBar()` | Shows a nice progress bar in the terminal as exercises are imported |

### Run it!

```bash
php artisan exercises:import
```

> This will take 2-5 minutes as it fetches 1,500+ exercises from the ExerciseDB API. You'll see a progress bar. When done:
> ```
> ✅ Done! Imported: 1534, Updated: 0
> ```

Verify:

```bash
php artisan tinker
```

```php
App\Models\Exercise::count();
// Should return 1500+ (the 10 seed exercises + imported ones)

App\Models\Exercise::whereNotNull('gif_url')->count();
// Should return 1500+ (all ExerciseDB exercises have GIFs)

App\Models\Exercise::where('muscle_group', 'chest')->pluck('name')->take(5);
// Should show several chest exercises
```

---

## Step 5.5 — Test the AI Endpoint

Start the server if it's not running:

```bash
php artisan serve
```

### Test with curl:

```bash
curl -X POST http://localhost:8000/api/chat \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{
    "message": "Should I increase weight on bench press?",
    "context": {
      "exercise": "Barbell Bench Press",
      "weight": 80,
      "reps": 8,
      "rir": 2
    }
  }'
```

> **Expected response (with valid API key):**
> ```json
> {
>   "data": {
>     "advice": "Great question! With 2 RIR at 80kg×8, you have room to progress...",
>     "suggestions": [
>       "Should I increase the weight next session?",
>       "What accessories complement this exercise?",
>       "How should I adjust my rest periods?"
>     ]
>   }
> }
> ```
>
> **If you don't have an API key yet**, you'll get:
> ```json
> {"error": "AI service returned an error. Please try again."}
> ```
> That's fine! The endpoint works, and when you add your key to `.env` it will connect to DeepSeek.

---

# 🏆 The Entire Backend is COMPLETE!

## Final File Summary

```
musclo-api/
├── app/
│   ├── Console/Commands/
│   │   └── ImportExercisesCommand.php      ← Imports 1500+ exercises from ExerciseDB
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Auth/
│   │   │   │   ├── RegisterController.php  ← Creates user + token
│   │   │   │   ├── LoginController.php     ← Validates creds + token
│   │   │   │   └── LogoutController.php    ← Revokes token
│   │   │   ├── RoutineController.php       ← Full CRUD + lastLog
│   │   │   ├── ExerciseController.php      ← Search, filter, custom exercises
│   │   │   ├── WorkoutLogController.php    ← Save workout, PR detection, stats, history
│   │   │   └── AICoachController.php       ← AI chat endpoint
│   │   ├── Middleware/
│   │   │   └── ForceJsonResponse.php       ← Forces JSON for all API responses
│   │   ├── Requests/
│   │   │   ├── Auth/
│   │   │   │   ├── RegisterRequest.php
│   │   │   │   └── LoginRequest.php
│   │   │   ├── StoreRoutineRequest.php
│   │   │   ├── UpdateRoutineRequest.php
│   │   │   ├── StoreWorkoutLogRequest.php
│   │   │   └── AskAIRequest.php
│   │   └── Resources/
│   │       ├── UserResource.php
│   │       ├── RoutineResource.php
│   │       ├── ExerciseResource.php
│   │       ├── WorkoutLogResource.php
│   │       └── SetDataResource.php
│   ├── Models/
│   │   ├── User.php                        ← + relationships
│   │   ├── Routine.php                     ← belongsToMany exercises
│   │   ├── Exercise.php                    ← scopes, casts
│   │   ├── RoutineExercise.php             ← pivot model
│   │   ├── WorkoutLog.php                  ← calculateTotalVolume()
│   │   └── SetData.php                     ← volume accessor
│   └── Services/
│       ├── DeepSeekService.php             ← AI integration (Service Pattern)
│       └── ExerciseDBService.php           ← ExerciseDB API client
├── config/
│   └── deepseek.php                        ← AI config
├── database/
│   ├── migrations/
│   │   ├── create_routines_table.php
│   │   ├── create_exercises_table.php
│   │   ├── create_routine_exercise_table.php
│   │   ├── create_workout_logs_table.php
│   │   └── create_set_data_table.php
│   └── seeders/
│       └── ExerciseSeeder.php              ← 10 starter exercises
├── routes/
│   └── api.php                             ← 15+ endpoints
└── .env                                    ← Secrets
```

## API Endpoints Recap

| Method | URL | Auth | Purpose |
|---|---|---|---|
| `POST` | `/api/register` | ❌ | Create account |
| `POST` | `/api/login` | ❌ | Login, get token |
| `POST` | `/api/logout` | ✅ | Revoke token |
| `GET` | `/api/user` | ✅ | Current user info |
| `GET` | `/api/routines` | ✅ | List routines |
| `POST` | `/api/routines` | ✅ | Create routine |
| `GET` | `/api/routines/{id}` | ✅ | Show routine |
| `PUT` | `/api/routines/{id}` | ✅ | Update routine |
| `DELETE` | `/api/routines/{id}` | ✅ | Delete routine |
| `GET` | `/api/routines/{id}/last-log` | ✅ | Last workout for auto-fill |
| `GET` | `/api/exercises` | ✅ | Search/filter exercises |
| `POST` | `/api/exercises` | ✅ | Create custom exercise |
| `POST` | `/api/workouts` | ✅ | Save workout + auto PR detection |
| `GET` | `/api/workouts/history` | ✅ | Workout history (paginated) |
| `GET` | `/api/workouts/stats` | ✅ | Dashboard stats + muscle radar |
| `GET` | `/api/workouts/exercise/{id}/history` | ✅ | Progressive overload chart data |
| `GET` | `/api/workouts/{id}` | ✅ | Single workout detail |
| `DELETE` | `/api/workouts/{id}` | ✅ | Delete workout |
| `POST` | `/api/chat` | ✅ | Ask AI coach |

---

Your backend is **production-ready, enterprise-grade**, and structured so cleanly that the AI building your React frontend will have a perfect API to consume. You're way ahead of Hevy and Lyfta! 🚀💪
