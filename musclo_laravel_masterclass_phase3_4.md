# 🏋️ Musclo — Laravel Masterclass: Phase 3 & 4

---

# PHASE 3: Authentication (Sanctum)

> **Goal:** Let users register, login, and logout. Every request after login sends a token so Laravel knows WHO is making the request.

## Step 3.1 — The ForceJsonResponse Middleware

First, a small but critical piece. Since we're building a pure API (no web pages), we need Laravel to ALWAYS respond with JSON, even for errors.

**File:** `app/Http/Middleware/ForceJsonResponse.php`

Replace the entire file:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class ForceJsonResponse
{
    public function handle(Request $request, Closure $next): Response
    {
        $request->headers->set('Accept', 'application/json');
        return $next($request);
    }
}
```

> **Why?** Without this, if you hit a route that doesn't exist, Laravel returns an HTML page saying "404 Not Found." That breaks the React frontend. With this middleware, it returns a clean JSON error: `{"message": "Not Found"}`.

Now **register** this middleware. Open:

**File:** `bootstrap/app.php`

Find the `->withMiddleware(function (Middleware $middleware) {` section and update it:

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->api(prepend: [
        \App\Http\Middleware\ForceJsonResponse::class,
    ]);
    $middleware->statefulApi();
})
```

> **What's happening here?**
> - `api(prepend: [...])` — adds our middleware to EVERY API request, and `prepend` means it runs FIRST (before anything else)
> - `statefulApi()` — tells Sanctum to support cookie-based auth for SPAs (our React frontend)

## Step 3.2 — CORS Configuration

CORS (Cross-Origin Resource Sharing) is a browser security feature. Our API runs on `localhost:8000` and our React frontend runs on `localhost:5173` — that's two different "origins." By default, the browser BLOCKS these cross-origin requests. We need to tell Laravel: "it's OK, let the frontend talk to me."

**File:** `config/cors.php`

Find and update these three values:

```php
'paths' => ['api/*', 'sanctum/csrf-cookie'],
'allowed_origins' => [env('FRONTEND_URL', 'http://localhost:5173')],
'supports_credentials' => true,
```

> - `paths` — which routes allow cross-origin requests (all API routes + Sanctum's CSRF cookie)
> - `allowed_origins` — ONLY our frontend URL can make requests (not random websites!)
> - `supports_credentials` — allows cookies to be sent with requests (needed for Sanctum sessions)

## Step 3.3 — Form Requests for Auth

### RegisterRequest
**File:** `app/Http/Requests/Auth/RegisterRequest.php`

Replace the entire file:

```php
<?php

namespace App\Http\Requests\Auth;

use Illuminate\Foundation\Http\FormRequest;

class RegisterRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true; // Anyone can register (no auth required)
    }

    public function rules(): array
    {
        return [
            'name'     => 'required|string|max:255',
            'email'    => 'required|email|unique:users,email',
            'password' => 'required|string|min:8|confirmed',
        ];
    }
}
```

> **Validation rules explained:**
> 
> | Rule | What it means |
> |---|---|
> | `required` | This field MUST be present |
> | `string` | Must be text (not a number or array) |
> | `max:255` | Name can't be longer than 255 characters |
> | `email` | Must be a valid email format |
> | `unique:users,email` | This email can't already exist in the `users` table — prevents duplicate accounts! |
> | `min:8` | Password must be at least 8 characters |
> | `confirmed` | The request must also contain a `password_confirmation` field that matches `password` |
>
> **If any rule fails**, Laravel automatically returns a `422 Unprocessable Entity` with the exact errors:
> ```json
> {"message": "The email has already been taken.", "errors": {"email": ["The email has already been taken."]}}
> ```
> You never write this error handling yourself — Laravel does it for you! 🎉

### LoginRequest
**File:** `app/Http/Requests/Auth/LoginRequest.php`

Replace the entire file:

```php
<?php

namespace App\Http\Requests\Auth;

use Illuminate\Foundation\Http\FormRequest;

class LoginRequest extends FormRequest
{
    public function authorize(): bool
    {
        return true;
    }

    public function rules(): array
    {
        return [
            'email'    => 'required|email',
            'password' => 'required|string',
        ];
    }
}
```

## Step 3.4 — UserResource (Response Formatter)

**File:** `app/Http/Resources/UserResource.php`

Replace the entire file:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'    => $this->id,
            'name'  => $this->name,
            'email' => $this->email,
        ];
    }
}
```

> **Why?** Without this, `$user->toArray()` would include `password`, `remember_token`, `email_verified_at`, and other things we DON'T want to expose. The Resource picks ONLY the fields we want.

## Step 3.5 — Auth Controllers

### RegisterController
**File:** `app/Http/Controllers/Auth/RegisterController.php`

Replace the entire file:

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use App\Http\Requests\Auth\RegisterRequest;
use App\Http\Resources\UserResource;
use App\Models\User;
use Illuminate\Http\JsonResponse;
use Illuminate\Support\Facades\Hash;

class RegisterController extends Controller
{
    public function store(RegisterRequest $request): JsonResponse
    {
        // Step 1: Create the user in the database
        $user = User::create([
            'name'     => $request->validated('name'),
            'email'    => $request->validated('email'),
            'password' => Hash::make($request->validated('password')),
        ]);

        // Step 2: Create a Sanctum token for instant login
        $token = $user->createToken('auth-token')->plainTextToken;

        // Step 3: Return the user data + token
        return response()->json([
            'user'  => new UserResource($user),
            'token' => $token,
        ], 201); // 201 = "Created"
    }
}
```

> **Let's trace what happens when someone registers:**
> 
> 1. React sends `POST /api/register` with `{name, email, password, password_confirmation}`
> 2. `RegisterRequest` runs validation. If it fails → automatic 422 error. We never reach the controller.
> 3. If validation passes → the `store()` method runs
> 4. `$request->validated('name')` gets ONLY the fields that passed validation (safe!)
> 5. `Hash::make()` hashes the password (so we never store it in plain text — `$2y$12$...`)
> 6. `User::create()` inserts a row into the `users` table
> 7. `createToken('auth-token')` creates a Sanctum token and stores it in `personal_access_tokens` table
> 8. `plainTextToken` gives us the actual string like `3|abc123xyz...` — this is what the frontend stores
> 9. We return the user data (via `UserResource` — safe!) and the token

### LoginController
**File:** `app/Http/Controllers/Auth/LoginController.php`

Replace the entire file:

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use App\Http\Requests\Auth\LoginRequest;
use App\Http\Resources\UserResource;
use App\Models\User;
use Illuminate\Http\JsonResponse;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class LoginController extends Controller
{
    public function store(LoginRequest $request): JsonResponse
    {
        // Step 1: Find the user by email
        $user = User::where('email', $request->validated('email'))->first();

        // Step 2: Check if user exists AND password matches
        if (! $user || ! Hash::check($request->validated('password'), $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        // Step 3: Create a new token
        $token = $user->createToken('auth-token')->plainTextToken;

        // Step 4: Return user + token
        return response()->json([
            'user'  => new UserResource($user),
            'token' => $token,
        ]);
    }
}
```

> **Security note:** We say "credentials are incorrect" — we DON'T say "user not found" or "wrong password." Why? If we said "user not found," a hacker could use that to figure out which emails are registered!
>
> **`Hash::check()`** compares the plain-text password from the request against the hashed password in the database. It returns `true` or `false`.

### LogoutController
**File:** `app/Http/Controllers/Auth/LogoutController.php`

Replace the entire file:

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class LogoutController extends Controller
{
    public function destroy(Request $request): JsonResponse
    {
        // Delete the token that was used for this request
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'message' => 'Logged out successfully.',
        ]);
    }
}
```

> **How this works:** The user sends a request with their token in the `Authorization` header. `$request->user()` finds the user from that token. `currentAccessToken()->delete()` removes that specific token from the database. Now that token is invalid — any future requests with it will get a 401 error.

---

# PHASE 4: The Core Engine (Business Logic)

## Step 4.1 — API Routes

This is the "table of contents" for your entire API. Every URL the frontend can call is defined here.

**File:** `routes/api.php`

Replace the entire file:

```php
<?php

use App\Http\Controllers\Auth\RegisterController;
use App\Http\Controllers\Auth\LoginController;
use App\Http\Controllers\Auth\LogoutController;
use App\Http\Controllers\RoutineController;
use App\Http\Controllers\ExerciseController;
use App\Http\Controllers\WorkoutLogController;
use App\Http\Controllers\AICoachController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

/*
|--------------------------------------------------------------------------
| PUBLIC ROUTES (no login required)
|--------------------------------------------------------------------------
*/
Route::post('/register', [RegisterController::class, 'store']);
Route::post('/login', [LoginController::class, 'store']);

/*
|--------------------------------------------------------------------------
| PROTECTED ROUTES (login required — Sanctum checks the token)
|--------------------------------------------------------------------------
*/
Route::middleware('auth:sanctum')->group(function () {

    // --- Auth ---
    Route::post('/logout', [LogoutController::class, 'destroy']);
    Route::get('/user', function (Request $request) {
        return new \App\Http\Resources\UserResource($request->user());
    });

    // --- Routines (full CRUD) ---
    Route::apiResource('routines', RoutineController::class);
    Route::get('/routines/{routine}/last-log', [RoutineController::class, 'lastLog']);

    // --- Exercises ---
    Route::get('/exercises', [ExerciseController::class, 'index']);
    Route::post('/exercises', [ExerciseController::class, 'store']);

    // --- Workouts ---
    Route::post('/workouts', [WorkoutLogController::class, 'store']);
    Route::get('/workouts/history', [WorkoutLogController::class, 'history']);
    Route::get('/workouts/stats', [WorkoutLogController::class, 'stats']);
    Route::get('/workouts/exercise/{exercise}/history', [WorkoutLogController::class, 'exerciseHistory']);
    Route::get('/workouts/{workoutLog}', [WorkoutLogController::class, 'show']);
    Route::delete('/workouts/{workoutLog}', [WorkoutLogController::class, 'destroy']);

    // --- AI Coach ---
    Route::post('/chat', [AICoachController::class, 'ask']);
});
```

> **Key concepts:**
> 
> | Concept | Explanation |
> |---|---|
> | `Route::post('/register', ...)` | When someone sends a POST request to `/api/register`, run `RegisterController`'s `store` method |
> | `Route::middleware('auth:sanctum')` | Everything inside this group REQUIRES a valid token. Without it → 401 Unauthorized |
> | `Route::apiResource('routines', ...)` | Creates 5 routes at once: GET /routines, POST /routines, GET /routines/{id}, PUT /routines/{id}, DELETE /routines/{id} |
> | `{routine}` vs `{workoutLog}` | Laravel auto-finds the model by ID. `{routine}` looks up `Routine::find($id)` for you! This is called **Route Model Binding**. |

> [!IMPORTANT]
> **Notice the order!** `Route::get('/workouts/stats', ...)` comes BEFORE `Route::get('/workouts/{workoutLog}', ...)`. If we put `{workoutLog}` first, Laravel would try to find a workout with ID "stats"! Routes are matched top to bottom.

## Step 4.2 — All API Resources

These control EXACTLY what JSON your API returns. Think of them as "filters" that pick only the safe, useful fields.

### ExerciseResource
**File:** `app/Http/Resources/ExerciseResource.php`

Replace the entire file:

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class ExerciseResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'                => $this->id,
            'exercisedb_id'     => $this->exercisedb_id,
            'name'              => $this->name,
            'muscle_group'      => $this->muscle_group,
            'secondary_muscles' => $this->secondary_muscles,
            'equipment'         => $this->equipment,
            'body_part'         => $this->body_part,
            'category'          => $this->category,
            'instructions'      => $this->instructions,
            'gif_url'           => $this->gif_url,
            'is_custom'         => $this->is_custom,
        ];
    }
}
```

### SetDataResource
**File:** `app/Http/Resources/SetDataResource.php`

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class SetDataResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'          => $this->id,
            'exercise_id' => $this->exercise_id,
            'exercise'    => new ExerciseResource($this->whenLoaded('exercise')),
            'set_number'  => $this->set_number,
            'set_type'    => $this->set_type,
            'weight_kg'   => (float) $this->weight_kg,
            'reps'        => $this->reps,
            'rir'         => $this->rir,
            'rpe'         => $this->rpe,
            'is_pr'       => $this->is_pr,
            'volume'      => $this->volume, // Uses our accessor!
        ];
    }
}
```

> **`$this->whenLoaded('exercise')`** — this is SMART. It only includes the exercise data IF we pre-loaded it with `->with('exercise')`. If we didn't, it just skips it. This prevents N+1 query problems (a very common performance bug).

### WorkoutLogResource
**File:** `app/Http/Resources/WorkoutLogResource.php`

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class WorkoutLogResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'               => $this->id,
            'routine_id'       => $this->routine_id,
            'routine_name'     => $this->name ?? $this->routine?->name,
            'started_at'       => $this->started_at?->toISOString(),
            'completed_at'     => $this->completed_at?->toISOString(),
            'duration_seconds' => $this->duration_seconds,
            'duration_display' => $this->formatDuration(),
            'total_volume'     => (float) $this->total_volume,
            'notes'            => $this->notes,
            'sets'             => SetDataResource::collection($this->whenLoaded('sets')),
            'sets_count'       => $this->whenCounted('sets'),
            'created_at'       => $this->created_at?->toISOString(),
        ];
    }

    private function formatDuration(): string
    {
        $minutes = intdiv($this->duration_seconds, 60);
        $hours = intdiv($minutes, 60);
        $mins = $minutes % 60;

        return $hours > 0 ? "{$hours}h {$mins}m" : "{$mins}m";
    }
}
```

> **`$this->routine?->name`** — the `?` is the **nullsafe operator**. If `routine` is `null` (deleted routine), it doesn't crash — it just returns `null`. 
>
> **`$this->whenCounted('sets')`** — only includes a count if we loaded it with `->withCount('sets')`. Useful for list views where you want "12 sets" but don't need all the set data.

### RoutineResource
**File:** `app/Http/Resources/RoutineResource.php`

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class RoutineResource extends JsonResource
{
    public function toArray(Request $request): array
    {
        return [
            'id'         => $this->id,
            'name'       => $this->name,
            'notes'      => $this->notes,
            'exercises'  => $this->whenLoaded('exercises', function () {
                return $this->exercises->map(fn ($exercise) => [
                    'id'           => $exercise->id,
                    'name'         => $exercise->name,
                    'muscle_group' => $exercise->muscle_group,
                    'equipment'    => $exercise->equipment,
                    'gif_url'      => $exercise->gif_url,
                    'sort_order'   => $exercise->pivot->sort_order,
                    'target_sets'  => $exercise->pivot->target_sets,
                    'target_reps'  => $exercise->pivot->target_reps,
                ]);
            }),
            'created_at' => $this->created_at?->toISOString(),
        ];
    }
}
```

> **`$exercise->pivot->sort_order`** — remember the pivot table `routine_exercise`? When we load exercises through the `BelongsToMany` relationship with `->withPivot(...)`, those extra columns are available via `->pivot`.

## Step 4.3 — Form Requests for Core Features

### StoreRoutineRequest
**File:** `app/Http/Requests/StoreRoutineRequest.php`

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreRoutineRequest extends FormRequest
{
    public function authorize(): bool { return true; }

    public function rules(): array
    {
        return [
            'name'                       => 'required|string|max:100',
            'notes'                      => 'nullable|string|max:1000',
            'exercises'                  => 'required|array|min:1',
            'exercises.*.id'             => 'required|exists:exercises,id',
            'exercises.*.sort_order'     => 'required|integer|min:0',
            'exercises.*.target_sets'    => 'sometimes|integer|min:1|max:20',
            'exercises.*.target_reps'    => 'sometimes|integer|min:1|max:100',
        ];
    }
}
```

> **`exercises.*.id`** — the `*` means "every item in the array." So if you send 5 exercises, Laravel checks each one exists in the `exercises` table. If exercise #3 has an invalid ID → instant 422 error telling you exactly which one failed.
>
> **`sometimes`** — only validate this field IF it's present. If the frontend doesn't send `target_sets`, that's fine, we'll use the default (3).

### StoreWorkoutLogRequest
**File:** `app/Http/Requests/StoreWorkoutLogRequest.php`

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StoreWorkoutLogRequest extends FormRequest
{
    public function authorize(): bool { return true; }

    public function rules(): array
    {
        return [
            'routine_id'         => 'nullable|exists:routines,id',
            'name'               => 'nullable|string|max:100',
            'started_at'         => 'required|date',
            'completed_at'       => 'nullable|date|after:started_at',
            'duration_seconds'   => 'required|integer|min:0',
            'notes'              => 'nullable|string|max:1000',
            'sets'               => 'required|array|min:1',
            'sets.*.exercise_id' => 'required|exists:exercises,id',
            'sets.*.set_number'  => 'required|integer|min:1',
            'sets.*.set_type'    => 'required|in:working,warmup,dropset,failure',
            'sets.*.weight_kg'   => 'required|numeric|min:0|max:9999',
            'sets.*.reps'        => 'required|integer|min:0|max:999',
            'sets.*.rir'         => 'nullable|integer|min:0|max:10',
            'sets.*.rpe'         => 'nullable|integer|min:1|max:10',
        ];
    }
}
```

> **`in:working,warmup,dropset,failure`** — the `set_type` must be EXACTLY one of these four values. Anything else → rejected. This prevents bad data from ever entering your database.
>
> **`after:started_at`** — the `completed_at` must be AFTER `started_at`. You can't finish a workout before you start it! 😄

### AskAIRequest
**File:** `app/Http/Requests/AskAIRequest.php`

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class AskAIRequest extends FormRequest
{
    public function authorize(): bool { return true; }

    public function rules(): array
    {
        return [
            'message'                 => 'required|string|max:1000',
            'context'                 => 'sometimes|array',
            'context.exercise'        => 'sometimes|string|max:100',
            'context.weight'          => 'sometimes|numeric|min:0|max:999',
            'context.reps'            => 'sometimes|integer|min:0|max:100',
            'context.rir'             => 'sometimes|integer|min:0|max:10',
            'context.history_summary' => 'sometimes|string|max:2000',
        ];
    }
}
```

## Step 4.4 — ExerciseController

**File:** `app/Http/Controllers/ExerciseController.php`

Replace the entire file:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Resources\ExerciseResource;
use App\Models\Exercise;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class ExerciseController extends Controller
{
    /**
     * GET /api/exercises
     * List all exercises available to the current user.
     * Supports: ?search=, ?muscle_group=, ?equipment=, ?page=
     */
    public function index(Request $request): JsonResponse
    {
        $query = Exercise::availableTo($request->user()->id);

        // Search by name
        if ($search = $request->input('search')) {
            $query->where('name', 'like', "%{$search}%");
        }

        // Filter by muscle group
        if ($muscle = $request->input('muscle_group')) {
            $query->where('muscle_group', $muscle);
        }

        // Filter by equipment
        if ($equipment = $request->input('equipment')) {
            $query->where('equipment', $equipment);
        }

        // Order alphabetically, paginate 20 per page
        $exercises = $query->orderBy('name')->paginate(20);

        return ExerciseResource::collection($exercises)->response();
    }

    /**
     * POST /api/exercises
     * Create a custom exercise for the current user.
     */
    public function store(Request $request): JsonResponse
    {
        $validated = $request->validate([
            'name'         => 'required|string|max:150',
            'muscle_group' => 'required|string|max:50',
            'equipment'    => 'nullable|string|max:50',
            'instructions' => 'nullable|string',
        ]);

        $exercise = Exercise::create(array_merge($validated, [
            'is_custom' => true,
            'user_id'   => $request->user()->id,
        ]));

        return response()->json([
            'data' => new ExerciseResource($exercise),
        ], 201);
    }
}
```

> **`Exercise::availableTo($userId)`** — remember the scope we defined in the model? It gives us global exercises + this user's custom exercises. `$request->user()->id` gets the current user's ID from the Sanctum token.
>
> **`paginate(20)`** — returns 20 exercises per page, and automatically adds pagination metadata (`current_page`, `last_page`, `total`, `next_page_url`, etc.) to the response. The frontend can call `/api/exercises?page=2` to get the next page.
>
> **`array_merge`** — combines the validated user input with our forced values (`is_custom = true`). A user can't set `is_custom = false` to add global exercises — our code forces it to `true`.

## Step 4.5 — RoutineController

**File:** `app/Http/Controllers/RoutineController.php`

Replace the entire file:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\StoreRoutineRequest;
use App\Http\Requests\UpdateRoutineRequest;
use App\Http\Resources\RoutineResource;
use App\Http\Resources\WorkoutLogResource;
use App\Models\Routine;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;

class RoutineController extends Controller
{
    /**
     * GET /api/routines
     * List all routines for the current user.
     */
    public function index(Request $request): JsonResponse
    {
        $routines = $request->user()
            ->routines()
            ->with('exercises') // Eager load exercises to avoid N+1
            ->orderByDesc('updated_at')
            ->get();

        return response()->json([
            'data' => RoutineResource::collection($routines),
        ]);
    }

    /**
     * POST /api/routines
     * Create a new routine with exercises.
     */
    public function store(StoreRoutineRequest $request): JsonResponse
    {
        $validated = $request->validated();

        // Step 1: Create the routine
        $routine = $request->user()->routines()->create([
            'name'  => $validated['name'],
            'notes' => $validated['notes'] ?? null,
        ]);

        // Step 2: Attach exercises via pivot table
        $this->syncExercises($routine, $validated['exercises']);

        // Step 3: Load the exercises and return
        $routine->load('exercises');

        return response()->json([
            'data' => new RoutineResource($routine),
        ], 201);
    }

    /**
     * GET /api/routines/{routine}
     * Show a single routine with its exercises.
     */
    public function show(Request $request, Routine $routine): JsonResponse
    {
        // Make sure this routine belongs to the current user
        if ($routine->user_id !== $request->user()->id) {
            abort(403, 'This routine does not belong to you.');
        }

        $routine->load('exercises');

        return response()->json([
            'data' => new RoutineResource($routine),
        ]);
    }

    /**
     * PUT /api/routines/{routine}
     * Update a routine and its exercises.
     */
    public function update(UpdateRoutineRequest $request, Routine $routine): JsonResponse
    {
        if ($routine->user_id !== $request->user()->id) {
            abort(403, 'This routine does not belong to you.');
        }

        $validated = $request->validated();

        $routine->update([
            'name'  => $validated['name'],
            'notes' => $validated['notes'] ?? $routine->notes,
        ]);

        if (isset($validated['exercises'])) {
            $this->syncExercises($routine, $validated['exercises']);
        }

        $routine->load('exercises');

        return response()->json([
            'data' => new RoutineResource($routine),
        ]);
    }

    /**
     * DELETE /api/routines/{routine}
     */
    public function destroy(Request $request, Routine $routine): JsonResponse
    {
        if ($routine->user_id !== $request->user()->id) {
            abort(403, 'This routine does not belong to you.');
        }

        $routine->delete();

        return response()->json(null, 204); // 204 = No Content
    }

    /**
     * GET /api/routines/{routine}/last-log
     * Get the most recent workout log for a routine (for auto-filling previous weights).
     */
    public function lastLog(Request $request, Routine $routine): JsonResponse
    {
        if ($routine->user_id !== $request->user()->id) {
            abort(403);
        }

        $lastLog = $routine->workoutLogs()
            ->with('sets.exercise')
            ->orderByDesc('started_at')
            ->first();

        if (! $lastLog) {
            return response()->json(['data' => null]);
        }

        return response()->json([
            'data' => new WorkoutLogResource($lastLog),
        ]);
    }

    /**
     * Helper: sync exercises to a routine via the pivot table.
     */
    private function syncExercises(Routine $routine, array $exercises): void
    {
        // Build the pivot data array
        $syncData = [];
        foreach ($exercises as $exercise) {
            $syncData[$exercise['id']] = [
                'sort_order'   => $exercise['sort_order'] ?? 0,
                'target_sets'  => $exercise['target_sets'] ?? 3,
                'target_reps'  => $exercise['target_reps'] ?? 10,
            ];
        }

        // sync() removes old exercises and adds new ones in one operation
        $routine->exercises()->sync($syncData);
    }
}
```

> **`$request->user()->routines()->create(...)`** — this is beautiful. It creates a routine AND automatically sets `user_id` to the current user. We never manually set it — the relationship handles it.
>
> **`->sync($syncData)`** — this is the pivot table magic. It takes an array where keys are exercise IDs and values are the pivot data. It removes any exercises NOT in the array and adds new ones. One method call replaces: delete all, then insert all. Efficient!
>
> **`abort(403)`** — immediately stops execution and returns `{"message": "This routine does not belong to you."}` with a 403 Forbidden status. Security!
>
> **`lastLog`** — this is the KILLER feature for progressive overload. When you start a workout from a routine, the frontend calls this to get your last weights so it can auto-fill them. "Last time you did 80kg × 8, try 82.5kg × 8?" 💪

## Step 4.6 — WorkoutLogController (The Brain 🧠)

This is the most complex controller. It handles saving workouts, detecting PRs, calculating stats, and fetching exercise history for progress charts.

**File:** `app/Http/Controllers/WorkoutLogController.php`

Replace the entire file:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\StoreWorkoutLogRequest;
use App\Http\Resources\WorkoutLogResource;
use App\Models\SetData;
use App\Models\WorkoutLog;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;

class WorkoutLogController extends Controller
{
    /**
     * POST /api/workouts
     * Save a completed workout with all sets. Auto-detects PRs.
     */
    public function store(StoreWorkoutLogRequest $request): JsonResponse
    {
        $validated = $request->validated();

        // We wrap everything in a DB transaction.
        // If ANYTHING fails, ALL changes are rolled back.
        // This prevents "half-saved" workouts.
        $workoutLog = DB::transaction(function () use ($validated, $request) {

            // Step 1: Create the workout log
            $log = WorkoutLog::create([
                'user_id'          => $request->user()->id,
                'routine_id'       => $validated['routine_id'] ?? null,
                'name'             => $validated['name'] ?? null,
                'started_at'       => $validated['started_at'],
                'completed_at'     => $validated['completed_at'] ?? now(),
                'duration_seconds' => $validated['duration_seconds'],
                'notes'            => $validated['notes'] ?? null,
            ]);

            // Step 2: Save each set and check for PRs
            foreach ($validated['sets'] as $setData) {
                $set = $log->sets()->create($setData);

                // PR Detection: Is this the highest volume ever for this exercise?
                if ($set->set_type === 'working') {
                    $previousBest = SetData::where('exercise_id', $set->exercise_id)
                        ->where('id', '!=', $set->id) // Exclude the set we just created
                        ->where('set_type', 'working')
                        ->selectRaw('MAX(weight_kg * reps) as max_volume')
                        ->value('max_volume') ?? 0;

                    $currentVolume = $set->weight_kg * $set->reps;

                    if ($currentVolume > $previousBest) {
                        $set->update(['is_pr' => true]);
                    }
                }
            }

            // Step 3: Calculate and store total volume
            $log->update([
                'total_volume' => $log->calculateTotalVolume(),
            ]);

            return $log->load('sets.exercise');
        });

        return response()->json([
            'data' => new WorkoutLogResource($workoutLog),
        ], 201);
    }

    /**
     * GET /api/workouts/history
     * Get paginated workout history for the current user.
     */
    public function history(Request $request): JsonResponse
    {
        $logs = $request->user()
            ->workoutLogs()
            ->with('sets.exercise', 'routine')
            ->withCount('sets')
            ->orderByDesc('started_at')
            ->paginate($request->integer('per_page', 15));

        return WorkoutLogResource::collection($logs)->response();
    }

    /**
     * GET /api/workouts/{workoutLog}
     * Show a single workout with all sets.
     */
    public function show(Request $request, WorkoutLog $workoutLog): JsonResponse
    {
        if ($workoutLog->user_id !== $request->user()->id) {
            abort(403);
        }

        $workoutLog->load('sets.exercise', 'routine');

        return response()->json([
            'data' => new WorkoutLogResource($workoutLog),
        ]);
    }

    /**
     * DELETE /api/workouts/{workoutLog}
     */
    public function destroy(Request $request, WorkoutLog $workoutLog): JsonResponse
    {
        if ($workoutLog->user_id !== $request->user()->id) {
            abort(403);
        }

        $workoutLog->delete(); // Cascade deletes all related set_data too!

        return response()->json(null, 204);
    }

    /**
     * GET /api/workouts/stats
     * Dashboard statistics for a time period.
     */
    public function stats(Request $request): JsonResponse
    {
        $period = $request->input('period', 'week');
        $days = match ($period) {
            'week'  => 7,
            'month' => 30,
            'year'  => 365,
            default => 7,
        };

        $logs = $request->user()
            ->workoutLogs()
            ->where('started_at', '>=', now()->subDays($days))
            ->with('sets.exercise')
            ->get();

        // Count sets per muscle group for the radar chart
        $muscleDistribution = $logs
            ->flatMap->sets                              // Flatten all sets from all workouts
            ->groupBy(fn ($set) => $set->exercise->muscle_group) // Group by muscle
            ->map->count();                              // Count sets per group

        return response()->json([
            'data' => [
                'total_workouts'      => $logs->count(),
                'total_volume'        => (float) $logs->sum('total_volume'),
                'avg_duration'        => (int) $logs->avg('duration_seconds'),
                'muscle_distribution' => $muscleDistribution,
            ],
        ]);
    }

    /**
     * GET /api/workouts/exercise/{exercise}/history
     * Get set history for a specific exercise (for progress charts).
     * This is the key query for progressive overload tracking!
     */
    public function exerciseHistory(Request $request, int $exerciseId): JsonResponse
    {
        $limit = $request->integer('limit', 20);

        $sets = SetData::where('exercise_id', $exerciseId)
            ->whereHas('workoutLog', function ($q) use ($request) {
                $q->where('user_id', $request->user()->id);
            })
            ->with('workoutLog:id,started_at')
            ->orderByDesc('created_at')
            ->limit($limit * 5) // 5 sets per workout × 20 workouts
            ->get()
            ->groupBy(fn ($s) => $s->workoutLog->started_at->toDateString());

        return response()->json(['data' => $sets]);
    }
}
```

> **🧠 Let me explain the PR Detection logic step by step:**
>
> ```
> Scenario: User just did Bench Press → 85kg × 8 reps = 680 volume
>
> Step 1: Look at ALL previous working sets for Bench Press
> Step 2: Find the MAX volume ever recorded: maybe it was 80kg × 8 = 640
> Step 3: Is 680 > 640? YES! → Mark this set as is_pr = true 🏆
> ```
>
> The frontend will see `is_pr: true` in the response and show a gold PR banner with confetti! 🎉
>
> **🧠 The `exerciseHistory` query explained:**
>
> This powers the progress charts. The frontend asks: "Show me my Bench Press history."
>
> 1. Find all `SetData` where `exercise_id` = Bench Press
> 2. But ONLY if the parent `WorkoutLog` belongs to THIS user (`whereHas`)
> 3. Load just `id` and `started_at` from the workout (we don't need everything)
> 4. Get the most recent sets (limited)
> 5. Group them by date → `{"2026-02-28": [set1, set2, set3], "2026-02-25": [set1, set2]}`
>
> Now the frontend can plot: "Feb 25: 75kg, Feb 28: 80kg, Mar 1: 85kg" → beautiful upward trend line! 📈

### UpdateRoutineRequest
**File:** `app/Http/Requests/UpdateRoutineRequest.php`

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class UpdateRoutineRequest extends FormRequest
{
    public function authorize(): bool { return true; }

    public function rules(): array
    {
        return [
            'name'                       => 'required|string|max:100',
            'notes'                      => 'nullable|string|max:1000',
            'exercises'                  => 'sometimes|array|min:1',
            'exercises.*.id'             => 'required|exists:exercises,id',
            'exercises.*.sort_order'     => 'required|integer|min:0',
            'exercises.*.target_sets'    => 'sometimes|integer|min:1|max:20',
            'exercises.*.target_reps'    => 'sometimes|integer|min:1|max:100',
        ];
    }
}
```

---

## 🎉 Phase 3 & 4 Complete!

Here's what you just built:

| ✅ Done | What |
|---|---|
| ForceJsonResponse middleware | All errors return clean JSON |
| CORS configuration | React frontend can talk to the API |
| Auth validation | RegisterRequest + LoginRequest |
| Auth controllers | Register (with token), Login (with token), Logout (deletes token) |
| API routes | 15+ endpoints, all properly grouped and protected |
| 4 API Resources | ExerciseResource, SetDataResource, WorkoutLogResource, RoutineResource |
| 3 Form Requests | StoreRoutineRequest, StoreWorkoutLogRequest, AskAIRequest |
| ExerciseController | Search, filter, paginate, create custom exercises |
| RoutineController | Full CRUD + lastLog for auto-filling previous weights |
| WorkoutLogController | Save workouts with **automatic PR detection**, history with pagination, stats with muscle distribution, exercise history for progress charts |

---

## ✋ Checkpoint — Test Your API!

Start the Laravel server:

```bash
php artisan serve
```

Open a **new terminal** and test with curl:

### Test 1: Register a user
```bash
curl -X POST http://localhost:8000/api/register \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -d '{"name":"Test User","email":"test@musclo.app","password":"password123","password_confirmation":"password123"}'
```

> You should get back a JSON with `user` and `token`. **Copy the token!** You'll need it for the next tests.

### Test 2: Get exercises (using your token)
```bash
curl http://localhost:8000/api/exercises \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

> You should see your 10 seeded exercises!

### Test 3: Create a routine
```bash
curl -X POST http://localhost:8000/api/routines \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -d '{"name":"Push Day","exercises":[{"id":1,"sort_order":0,"target_sets":4,"target_reps":8},{"id":5,"sort_order":1,"target_sets":3,"target_reps":10}]}'
```

> You should see the routine with Bench Press and Overhead Press attached!

**When all 3 tests pass**, tell me and I'll give you **Phase 5 (DeepSeek AI Integration)** — the final phase! 🚀
