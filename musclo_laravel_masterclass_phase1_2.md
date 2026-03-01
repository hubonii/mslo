# 🏋️ Musclo — Laravel Backend Masterclass

## Welcome, Future Developer! 🎉

I've read your entire Master Blueprint. Here's what we're about to build together:

**Musclo's backend** is a rock-solid Laravel 12 REST API that will:
- 🔐 Authenticate users with Sanctum tokens
- 📋 Manage workout routines with exercises
- 📝 Log every rep, set, and weight in workouts
- 🏆 Automatically detect Personal Records (PRs)
- 🤖 Connect to DeepSeek R1 for AI coaching
- 📊 Serve clean JSON responses that a React frontend can consume perfectly

**My rules as your mentor:**
- ✅ Every file path is given explicitly
- ✅ Every terminal command is exact — just copy and paste
- ✅ Every concept is explained like you're 10 years old
- ✅ Nothing is simplified — this is enterprise-grade code

Let's go! 💪

---

# PHASE 1: Foundation & Setup

## Step 1.1 — Check Your Prerequisites

Before we write a single line of code, let's make sure your computer has everything installed. Open your terminal and run each of these one by one:

```bash
php -v
```
> **What to expect:** You should see `PHP 8.2` or higher (8.3 or 8.4 is even better). Laravel 12 requires PHP 8.2 minimum.
> 
> **If you don't have PHP:** Install it with `sudo apt install php php-cli php-mbstring php-xml php-curl php-mysql php-zip unzip` on Ubuntu/WSL.

```bash
composer -V
```
> **What to expect:** `Composer version 2.x.x`. Composer is PHP's package manager — it downloads Laravel and its dependencies for you, like `npm` does for JavaScript.
> 
> **If you don't have Composer:** Run:
> ```bash
> curl -sS https://getcomposer.org/installer | php
> sudo mv composer.phar /usr/local/bin/composer
> ```

```bash
mysql --version
```
> **What to expect:** `mysql Ver 8.x.x`. MySQL is our database — where all the workouts, users, and exercises will be stored.
>
> **If MySQL isn't running:** Start it with `sudo service mysql start`

```bash
node -v
```
> **What to expect:** `v20.x` or `v22.x`. We don't need Node for the backend, but it's good to confirm it's ready for the React frontend later.

## Step 1.2 — Create the Laravel Project

Now let's create our Laravel project! Navigate to wherever you keep your projects:

```bash
cd ~/devo
```

Now run this command. It tells Composer to download a fresh Laravel 12 project and put it in a folder called `musclo-api`:

```bash
composer create-project laravel/laravel musclo-api
```

> **What just happened?** Composer downloaded Laravel 12 and ALL its dependencies (hundreds of packages!) into a new `musclo-api/` folder. This is your entire backend project.

Now enter the project folder. **Every command from now on should be run from inside this folder:**

```bash
cd musclo-api
```

Let's verify Laravel is working:

```bash
php artisan --version
```
> You should see `Laravel Framework 12.x.x`. If you see this, you're golden! ✨

## Step 1.3 — Create the MySQL Database

Our app needs a database to store everything. Let's create it:

```bash
mysql -u root -p
```
> This opens MySQL's command line. It will ask for your password (the one you set when you installed MySQL). Type it and press Enter.

Once you see `mysql>`, type this:

```sql
CREATE DATABASE musclo_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

> **Why `utf8mb4`?** This lets our database store ALL characters — including emojis 💪🏆. The `unicode_ci` part means text comparisons are case-insensitive (so "Bench Press" and "bench press" are treated the same in searches).

Now exit MySQL:

```sql
EXIT;
```

## Step 1.4 — Configure the `.env` File

Laravel has a special file called `.env` (short for "environment") that stores secret configuration. It's like a secret settings file that NEVER gets uploaded to GitHub.

Open it:

```bash
nano .env
```

> **Or** open it in your code editor. The file is at: `musclo-api/.env`

Find these lines and change them to match your MySQL setup:

```env
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=musclo_db
DB_USERNAME=root
DB_PASSWORD=your_mysql_password_here
```

> **Replace `your_mysql_password_here`** with whatever password you use to log into MySQL.

Now add these lines at the **bottom** of the `.env` file (they don't exist yet, you're adding them new):

```env
FRONTEND_URL=http://localhost:5173
SANCTUM_STATEFUL_DOMAINS=localhost:5173
SESSION_DOMAIN=localhost
DEEPSEEK_API_KEY=your_deepseek_key_here
DEEPSEEK_MODEL=deepseek-reasoner
```

> **What are these?**
> - `FRONTEND_URL` — tells Laravel where the React frontend will run (Vite uses port 5173)
> - `SANCTUM_STATEFUL_DOMAINS` — tells Sanctum to trust requests from the frontend
> - `SESSION_DOMAIN` — the domain for session cookies
> - `DEEPSEEK_API_KEY` — your AI API key (we'll use it in Phase 5, but let's set it up now)

Save and close the file (`Ctrl+X`, then `Y`, then `Enter` if using nano).

## Step 1.5 — Install Sanctum (API Authentication)

Laravel 12 comes with Sanctum built-in, but we need to publish its configuration:

```bash
php artisan install:api
```

> **What just happened?** This command:
> 1. Created a `personal_access_tokens` migration (for storing login tokens)
> 2. Set up the routes for Sanctum
> 3. Added the necessary middleware
> 
> **What is Sanctum?** It's Laravel's way of handling login/logout for APIs. When a user logs in, they get a "token" (a long random string). They send this token with every request, and Laravel says "yep, that's a real user."

## Step 1.6 — Generate All Models, Migrations, and Controllers

Now the fun part! We're going to tell Laravel to create all the files we need. Each command creates multiple files at once.

> **What are Models, Migrations, and Controllers?**
> - **Model** = a PHP class that represents a database table. `User` model = `users` table.
> - **Migration** = a PHP file that describes what columns a table should have. It's like a blueprint for a table.
> - **Controller** = a PHP class that handles HTTP requests (like "give me all workouts" or "save this new routine").

Run these commands **one by one** inside `musclo-api/`:

### Models + Migrations + Factories

```bash
php artisan make:model Routine -mf
```
> This creates THREE files:
> - `app/Models/Routine.php` (the Model)
> - `database/migrations/202X_XX_XX_XXXXXX_create_routines_table.php` (the Migration)
> - `database/factories/RoutineFactory.php` (a Factory — for generating fake test data)

```bash
php artisan make:model Exercise -m
```

```bash
php artisan make:model RoutineExercise -m
```
> This is a **pivot table** — it connects Routines to Exercises (a routine can have many exercises, and an exercise can be in many routines).

```bash
php artisan make:model WorkoutLog -mf
```

```bash
php artisan make:model SetData -mf
```

### Controllers

```bash
php artisan make:controller Auth/RegisterController
```
```bash
php artisan make:controller Auth/LoginController
```
```bash
php artisan make:controller Auth/LogoutController
```

> **Why three separate controllers for auth?** This follows the **Single Responsibility Principle** — each controller does ONE thing. This is how enterprise code is structured. Hevy and Lyfta probably cram everything into one `AuthController`, but we're better than that 😎

```bash
php artisan make:controller RoutineController --api
```
> The `--api` flag creates a controller with `index`, `store`, `show`, `update`, and `destroy` methods already stubbed out — perfect for REST APIs.

```bash
php artisan make:controller ExerciseController --api
```
```bash
php artisan make:controller WorkoutLogController --api
```
```bash
php artisan make:controller AICoachController
```

### Form Requests (Validation)

```bash
php artisan make:request Auth/RegisterRequest
```
```bash
php artisan make:request Auth/LoginRequest
```
```bash
php artisan make:request StoreRoutineRequest
```
```bash
php artisan make:request UpdateRoutineRequest
```
```bash
php artisan make:request StoreWorkoutLogRequest
```
```bash
php artisan make:request AskAIRequest
```

> **What are Form Requests?** Instead of writing validation rules inside your controller (messy!), Laravel lets you create dedicated classes just for validation. When a request comes in, Laravel runs the validation BEFORE your controller code. If it fails, it automatically sends a 422 error with the exact validation errors. Professional! 🎩

### API Resources (JSON Response Formatters)

```bash
php artisan make:resource UserResource
```
```bash
php artisan make:resource RoutineResource
```
```bash
php artisan make:resource ExerciseResource
```
```bash
php artisan make:resource WorkoutLogResource
```
```bash
php artisan make:resource SetDataResource
```
```bash
php artisan make:resource AIResponseResource
```

> **What are API Resources?** They control EXACTLY what your API returns in JSON. Without them, you might accidentally expose sensitive data (like passwords). With them, you define a clean, predictable JSON structure that the React frontend can rely on.

### Other Files

```bash
php artisan make:middleware ForceJsonResponse
```
```bash
php artisan make:seeder ExerciseSeeder
```

> ✅ **Checkpoint!** You should now have a LOT of new files. Run `php artisan about` to see a summary of your app.

---

## Step 1.7 — Write the Migration Files

Now we need to tell each migration WHAT COLUMNS the table should have. Open each migration file and **replace the entire content** with the code I give you.

> **How do I find the migration files?** They're in `database/migrations/`. They have timestamps in their names like `2026_03_02_000001_create_routines_table.php`. The order matters!

### Migration 1: `create_routines_table.php`
**File:** `database/migrations/XXXX_XX_XX_XXXXXX_create_routines_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('routines', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->string('name', 100);
            $table->text('notes')->nullable();
            $table->timestamps();

            $table->index('user_id');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('routines');
    }
};
```

> **Let me explain every line:**
> 
> | Line | What it does |
> |---|---|
> | `$table->id()` | Creates an `id` column that auto-increments (1, 2, 3...). Every table needs this. |
> | `$table->foreignId('user_id')->constrained()->onDelete('cascade')` | Creates a `user_id` column that **references** the `users` table. `constrained()` means MySQL will enforce that any `user_id` must actually exist in the `users` table. `onDelete('cascade')` means: if a user is deleted, ALL their routines are automatically deleted too. |
> | `$table->string('name', 100)` | A text column, max 100 characters. |
> | `$table->text('notes')->nullable()` | A longer text column. `nullable()` means it can be empty — not every routine needs notes. |
> | `$table->timestamps()` | Adds `created_at` and `updated_at` columns. Laravel fills these automatically. |
> | `$table->index('user_id')` | Creates a database index for faster lookups. When we search "all routines for user 5", MySQL can find them instantly instead of scanning every row. |

---

### Migration 2: `create_exercises_table.php`
**File:** `database/migrations/XXXX_XX_XX_XXXXXX_create_exercises_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('exercises', function (Blueprint $table) {
            $table->id();
            $table->string('exercisedb_id', 20)->nullable()->unique();
            $table->string('name', 150);
            $table->string('muscle_group', 50);
            $table->string('secondary_muscles', 150)->nullable();
            $table->string('equipment', 50)->nullable();
            $table->string('body_part', 50)->nullable();
            $table->string('category', 30)->default('strength');
            $table->text('instructions')->nullable();
            $table->string('gif_url', 500)->nullable();
            $table->boolean('is_custom')->default(false);
            $table->foreignId('user_id')->nullable()->constrained()->onDelete('cascade');
            $table->timestamps();

            $table->index('muscle_group');
            $table->index('equipment');
            $table->index('body_part');
            $table->index('name');
            $table->index('user_id');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('exercises');
    }
};
```

> **Key things to understand:**
> 
> | Column | Purpose |
> |---|---|
> | `exercisedb_id` | Links to the ExerciseDB API. `unique()` means no two exercises can have the same ExerciseDB ID. |
> | `gif_url` | The URL to the animated GIF showing how to do the exercise (from ExerciseDB). |
> | `is_custom` | `false` = global exercise (from ExerciseDB). `true` = user-created custom exercise. |
> | `user_id` nullable | Global exercises have `null` user_id (they belong to everyone). Custom exercises have a user_id. |
> | `default('strength')` | If no category is specified, it defaults to "strength". |

---

### Migration 3: `create_routine_exercise_table.php` (Pivot Table)
**File:** `database/migrations/XXXX_XX_XX_XXXXXX_create_routine_exercise_table.php`

> **Wait — rename this file!** When you ran `make:model RoutineExercise -m`, Laravel may have named it `create_routine_exercises_table.php`. We need the table to be called `routine_exercise` (singular, no "s"). The easiest way is to just rename the migration file, OR make sure the `Schema::create` call inside uses the correct name. Let's use the correct name in our code:

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('routine_exercise', function (Blueprint $table) {
            $table->id();
            $table->foreignId('routine_id')->constrained()->onDelete('cascade');
            $table->foreignId('exercise_id')->constrained()->onDelete('cascade');
            $table->unsignedSmallInteger('sort_order')->default(0);
            $table->unsignedTinyInteger('target_sets')->default(3);
            $table->unsignedTinyInteger('target_reps')->default(10);
            $table->timestamps();

            $table->unique(['routine_id', 'exercise_id']);
            $table->index('routine_id');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('routine_exercise');
    }
};
```

> **What is a Pivot Table?** Imagine this: a "Push Day" routine has Bench Press, Overhead Press, and Tricep Dips. But Bench Press also appears in "Chest Day" routine. This is a **many-to-many** relationship — a routine has many exercises, and an exercise can be in many routines. The `routine_exercise` table sits IN BETWEEN and connects them.
>
> | Column | Purpose |
> |---|---|
> | `sort_order` | The order exercises appear in the routine (1st, 2nd, 3rd...) |
> | `target_sets` / `target_reps` | The default target for this exercise in this routine |
> | `unique(['routine_id', 'exercise_id'])` | Prevents adding the same exercise twice to the same routine |

---

### Migration 4: `create_workout_logs_table.php`
**File:** `database/migrations/XXXX_XX_XX_XXXXXX_create_workout_logs_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('workout_logs', function (Blueprint $table) {
            $table->id();
            $table->foreignId('user_id')->constrained()->onDelete('cascade');
            $table->foreignId('routine_id')->nullable()->constrained()->onDelete('set null');
            $table->string('name', 100)->nullable();
            $table->dateTime('started_at');
            $table->dateTime('completed_at')->nullable();
            $table->unsignedInteger('duration_seconds')->default(0);
            $table->decimal('total_volume', 12, 2)->default(0);
            $table->text('notes')->nullable();
            $table->timestamps();

            $table->index(['user_id', 'started_at']);
            $table->index('routine_id');
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('workout_logs');
    }
};
```

> **Interesting design decisions here:**
> 
> | Choice | Why? |
> |---|---|
> | `onDelete('set null')` for routine_id | If a routine is deleted, we DON'T want to lose the workout history! We just set `routine_id` to `null`. The workout log survives. |
> | `name` is nullable | It's a snapshot of the routine name at the time of the workout. If there's no routine, the user gave the workout a custom name (or none). |
> | `total_volume` | Calculated as `sum(weight × reps)` across all working sets. Stored so we don't recalculate it every time. |
> | `index(['user_id', 'started_at'])` | A **compound index** — makes "get all workouts for user X, sorted by date" lightning fast. |

---

### Migration 5: `create_set_data_table.php`
**File:** `database/migrations/XXXX_XX_XX_XXXXXX_create_set_data_table.php`

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('set_data', function (Blueprint $table) {
            $table->id();
            $table->foreignId('workout_log_id')->constrained()->onDelete('cascade');
            $table->foreignId('exercise_id')->constrained()->onDelete('restrict');
            $table->unsignedTinyInteger('set_number');
            $table->string('set_type', 20)->default('working');
            $table->decimal('weight_kg', 8, 2);
            $table->unsignedSmallInteger('reps');
            $table->unsignedTinyInteger('rir')->nullable();
            $table->unsignedTinyInteger('rpe')->nullable();
            $table->boolean('is_pr')->default(false);
            $table->timestamps();

            $table->index(['workout_log_id', 'exercise_id']);
            $table->index(['exercise_id', 'created_at']);
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('set_data');
    }
};
```

> **This is the heart of the app!** Every single rep you log is a row in this table.
> 
> | Column | Purpose |
> |---|---|
> | `set_type` | Is this a `working` set, `warmup`, `dropset`, or `failure` set? |
> | `weight_kg` | The weight used. `decimal(8, 2)` means up to 999,999.99 kg (more than enough!) |
> | `rir` | **Reps in Reserve** — how many more reps could you have done? (0 = complete failure, 3 = easy) |
> | `rpe` | **Rate of Perceived Exertion** — how hard was it on a 1-10 scale? |
> | `is_pr` | 🏆 Is this set a Personal Record? We calculate this automatically! |
> | `onDelete('restrict')` on exercise_id | We PREVENT deleting an exercise if any set data references it. Can't lose workout history! |
> | Index on `['exercise_id', 'created_at']` | Makes PR lookups fast — "what's the best set ever for Bench Press?" |

---

## Step 1.8 — Run the Migrations!

This is the big moment. This command reads all your migration files and creates the actual tables in MySQL:

```bash
php artisan migrate
```

> **What to expect:** You should see a list of migrations being run:
> ```
> Running migrations.
> 2026_xx_xx_000000_create_users_table .............. DONE
> 2026_xx_xx_000001_create_routines_table ........... DONE
> 2026_xx_xx_000002_create_exercises_table .......... DONE
> ...
> ```
> If you see errors, it usually means your `.env` database settings are wrong. Double-check `DB_DATABASE`, `DB_USERNAME`, and `DB_PASSWORD`.

> [!IMPORTANT]
> **If you get a migration order error** (like "exercise table doesn't exist when creating routine_exercise"), you need to make sure the migration filenames are in the right chronological order. The timestamps in the filenames determine the order. Routines and Exercises must be created BEFORE routine_exercise and set_data.

Let's verify the tables were created:

```bash
php artisan db:show
```

You should see all your tables listed: `users`, `routines`, `exercises`, `routine_exercise`, `workout_logs`, `set_data`, `personal_access_tokens`, etc.

---

# PHASE 2: Data Seeding & Relationships

Now that our tables exist, we need to:
1. Tell our Models HOW the tables are connected (Relationships)
2. Fill the `exercises` table with real data to test with (Seeders)

## Step 2.1 — Define Eloquent Relationships

> **What are Eloquent Relationships?** In real life, a User "has many" Routines. A Routine "belongs to" a User. In Laravel, we express these connections with special methods in our Models. Then we can do magical things like `$user->routines` to get all of a user's routines — Laravel writes the SQL for us!

### Model: `User.php`
**File:** `app/Models/User.php`

This file already exists. We just need to ADD the relationship methods. Open it and add these three methods inside the `User` class (after the existing code, before the closing `}`):

```php
// Add these USE statements at the top of the file (after the existing use statements):
use Illuminate\Database\Eloquent\Relations\HasMany;

// Add these methods inside the User class:

public function routines(): HasMany
{
    return $this->hasMany(Routine::class);
}

public function workoutLogs(): HasMany
{
    return $this->hasMany(WorkoutLog::class);
}

public function customExercises(): HasMany
{
    return $this->hasMany(Exercise::class)->where('is_custom', true);
}
```

> **Reading this out loud:** "A User *has many* Routines. A User *has many* WorkoutLogs. A User *has many* custom Exercises."
> 
> The `customExercises()` method is extra clever — it automatically filters to only return exercises where `is_custom` is `true`. So `$user->customExercises` gives you ONLY the exercises that user created themselves.

---

### Model: `Routine.php`
**File:** `app/Models/Routine.php`

**Replace the ENTIRE file** with:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Routine extends Model
{
    use HasFactory;

    protected $fillable = ['user_id', 'name', 'notes'];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    public function exercises(): BelongsToMany
    {
        return $this->belongsToMany(Exercise::class, 'routine_exercise')
            ->withPivot('sort_order', 'target_sets', 'target_reps')
            ->orderByPivot('sort_order');
    }

    public function workoutLogs(): HasMany
    {
        return $this->hasMany(WorkoutLog::class);
    }
}
```

> **New concept: `BelongsToMany`!** This is for the many-to-many relationship through the pivot table. Let me break down the `exercises()` method:
> 
> | Part | What it does |
> |---|---|
> | `belongsToMany(Exercise::class, 'routine_exercise')` | "This routine is connected to exercises via the `routine_exercise` pivot table" |
> | `->withPivot('sort_order', 'target_sets', 'target_reps')` | "When you fetch the exercises, ALSO include these extra columns from the pivot table" |
> | `->orderByPivot('sort_order')` | "Sort the exercises by their `sort_order` column in the pivot table" |
> 
> **What is `$fillable`?** It's a security feature. It lists WHICH columns can be mass-assigned (set all at once). Without it, a hacker could send `"is_admin": true` in a request and make themselves admin!

---

### Model: `Exercise.php`
**File:** `app/Models/Exercise.php`

**Replace the ENTIRE file** with:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Exercise extends Model
{
    use HasFactory;

    protected $fillable = [
        'exercisedb_id', 'name', 'muscle_group', 'secondary_muscles',
        'equipment', 'body_part', 'category', 'instructions',
        'gif_url', 'is_custom', 'user_id',
    ];

    protected $casts = [
        'is_custom' => 'boolean',
    ];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    public function routines(): BelongsToMany
    {
        return $this->belongsToMany(Routine::class, 'routine_exercise')
            ->withPivot('sort_order', 'target_sets', 'target_reps');
    }

    /**
     * Scope: get global exercises + exercises created by a specific user.
     * Usage: Exercise::availableTo($userId)->get();
     */
    public function scopeAvailableTo($query, int $userId)
    {
        return $query->where(function ($q) use ($userId) {
            $q->where('is_custom', false)
              ->orWhere('user_id', $userId);
        });
    }
}
```

> **New concept: `$casts`!** Database stores `is_custom` as `0` or `1` (integers). `$casts` tells Laravel to automatically convert it to `true`/`false` (boolean) in PHP. Much easier to work with!
> 
> **New concept: `scopeAvailableTo`!** This is a **query scope** — a reusable filter. When the frontend asks "show me all exercises", we don't want to show User A's custom exercises to User B. This scope says: "Give me all global exercises (is_custom = false) PLUS this specific user's custom ones."

---

### Model: `WorkoutLog.php`
**File:** `app/Models/WorkoutLog.php`

**Replace the ENTIRE file** with:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
use Illuminate\Database\Eloquent\Relations\HasMany;

class WorkoutLog extends Model
{
    use HasFactory;

    protected $fillable = [
        'user_id', 'routine_id', 'name', 'started_at',
        'completed_at', 'duration_seconds', 'total_volume', 'notes',
    ];

    protected $casts = [
        'started_at' => 'datetime',
        'completed_at' => 'datetime',
        'total_volume' => 'decimal:2',
    ];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    public function routine(): BelongsTo
    {
        return $this->belongsTo(Routine::class);
    }

    public function sets(): HasMany
    {
        return $this->hasMany(SetData::class);
    }

    /**
     * Calculate total volume from all working sets.
     * Volume = sum of (weight × reps) for non-warmup sets.
     */
    public function calculateTotalVolume(): float
    {
        return $this->sets->sum(fn (SetData $set) =>
            $set->set_type !== 'warmup' ? $set->weight_kg * $set->reps : 0
        );
    }
}
```

> **Why cast `started_at` to `datetime`?** Without this, `started_at` is just a string like `"2026-03-01 14:00:00"`. With the cast, it becomes a Carbon date object, so you can do `$workout->started_at->diffForHumans()` → "2 hours ago" 🕐
> 
> **The `calculateTotalVolume()` method** is the formula for progressive overload tracking: for each set that ISN'T a warmup, multiply weight × reps, then sum everything up.

---

### Model: `SetData.php`
**File:** `app/Models/SetData.php`

**Replace the ENTIRE file** with:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class SetData extends Model
{
    use HasFactory;

    protected $table = 'set_data';

    protected $fillable = [
        'workout_log_id', 'exercise_id', 'set_number', 'set_type',
        'weight_kg', 'reps', 'rir', 'rpe', 'is_pr',
    ];

    protected $casts = [
        'weight_kg' => 'decimal:2',
        'is_pr' => 'boolean',
    ];

    public function workoutLog(): BelongsTo
    {
        return $this->belongsTo(WorkoutLog::class);
    }

    public function exercise(): BelongsTo
    {
        return $this->belongsTo(Exercise::class);
    }

    /**
     * Volume for this single set.
     * This is an "accessor" — use it like $set->volume
     */
    public function getVolumeAttribute(): float
    {
        return (float) $this->weight_kg * $this->reps;
    }
}
```

> **Why `protected $table = 'set_data'`?** Laravel normally guesses the table name by making the model name plural: `SetData` → `set_datas`. But that's not a real word! We tell it explicitly: "the table is called `set_data`."
> 
> **New concept: Accessor!** The `getVolumeAttribute()` method is a virtual column. There's no `volume` column in the database, but you can access `$set->volume` and it returns `weight_kg × reps` on the fly. Magic! ✨

---

### Model: `RoutineExercise.php`
**File:** `app/Models/RoutineExercise.php`

**Replace the ENTIRE file** with:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class RoutineExercise extends Model
{
    protected $table = 'routine_exercise';

    protected $fillable = [
        'routine_id', 'exercise_id', 'sort_order',
        'target_sets', 'target_reps',
    ];

    public function routine(): BelongsTo
    {
        return $this->belongsTo(Routine::class);
    }

    public function exercise(): BelongsTo
    {
        return $this->belongsTo(Exercise::class);
    }
}
```

---

## Step 2.2 — Seed the Exercises Table

We need some exercises in the database to test with. Later, you'll run `php artisan exercises:import` to import 1,500+ from ExerciseDB, but for now let's add a few manually so you can test your API.

**File:** `database/seeders/ExerciseSeeder.php`

**Replace the ENTIRE file** with:

```php
<?php

namespace Database\Seeders;

use App\Models\Exercise;
use Illuminate\Database\Seeder;

class ExerciseSeeder extends Seeder
{
    public function run(): void
    {
        $exercises = [
            // CHEST
            [
                'name' => 'Barbell Bench Press',
                'muscle_group' => 'chest',
                'secondary_muscles' => 'triceps, shoulders',
                'equipment' => 'barbell',
                'body_part' => 'chest',
                'category' => 'strength',
                'instructions' => "Step 1: Lie on a flat bench.\nStep 2: Grip the barbell slightly wider than shoulder-width.\nStep 3: Lower the bar to your chest.\nStep 4: Press it back up to full extension.",
            ],
            [
                'name' => 'Incline Dumbbell Press',
                'muscle_group' => 'chest',
                'secondary_muscles' => 'triceps, shoulders',
                'equipment' => 'dumbbell',
                'body_part' => 'chest',
                'category' => 'strength',
                'instructions' => "Step 1: Set bench to 30-45 degrees.\nStep 2: Press dumbbells up from shoulder level.\nStep 3: Lower with control.",
            ],
            // BACK
            [
                'name' => 'Barbell Row',
                'muscle_group' => 'upper back',
                'secondary_muscles' => 'biceps, forearms',
                'equipment' => 'barbell',
                'body_part' => 'back',
                'category' => 'strength',
                'instructions' => "Step 1: Hinge at the hips with slight knee bend.\nStep 2: Pull barbell to lower chest.\nStep 3: Squeeze shoulder blades at the top.",
            ],
            [
                'name' => 'Pull-Up',
                'muscle_group' => 'lats',
                'secondary_muscles' => 'biceps, forearms',
                'equipment' => 'body weight',
                'body_part' => 'back',
                'category' => 'strength',
                'instructions' => "Step 1: Hang from bar with overhand grip.\nStep 2: Pull yourself up until chin is over bar.\nStep 3: Lower with control.",
            ],
            // SHOULDERS
            [
                'name' => 'Overhead Press',
                'muscle_group' => 'shoulders',
                'secondary_muscles' => 'triceps',
                'equipment' => 'barbell',
                'body_part' => 'shoulders',
                'category' => 'strength',
                'instructions' => "Step 1: Start with barbell at shoulder height.\nStep 2: Press overhead to full lockout.\nStep 3: Lower back to shoulders.",
            ],
            // LEGS
            [
                'name' => 'Barbell Squat',
                'muscle_group' => 'quads',
                'secondary_muscles' => 'glutes, hamstrings',
                'equipment' => 'barbell',
                'body_part' => 'upper legs',
                'category' => 'strength',
                'instructions' => "Step 1: Place barbell on upper back.\nStep 2: Squat down until thighs are parallel to floor.\nStep 3: Drive back up through your heels.",
            ],
            [
                'name' => 'Romanian Deadlift',
                'muscle_group' => 'hamstrings',
                'secondary_muscles' => 'glutes, lower back',
                'equipment' => 'barbell',
                'body_part' => 'upper legs',
                'category' => 'strength',
                'instructions' => "Step 1: Hold barbell at hip height.\nStep 2: Hinge at hips, pushing them back.\nStep 3: Lower until you feel a hamstring stretch.\nStep 4: Drive hips forward to return.",
            ],
            // ARMS
            [
                'name' => 'Barbell Curl',
                'muscle_group' => 'biceps',
                'secondary_muscles' => 'forearms',
                'equipment' => 'barbell',
                'body_part' => 'upper arms',
                'category' => 'strength',
                'instructions' => "Step 1: Stand with barbell, arms extended.\nStep 2: Curl the weight to shoulder height.\nStep 3: Lower with control.",
            ],
            [
                'name' => 'Tricep Pushdown',
                'muscle_group' => 'triceps',
                'secondary_muscles' => null,
                'equipment' => 'cable',
                'body_part' => 'upper arms',
                'category' => 'strength',
                'instructions' => "Step 1: Grip cable attachment at chest height.\nStep 2: Push down until arms are fully extended.\nStep 3: Return with control.",
            ],
            // ABS
            [
                'name' => 'Cable Crunch',
                'muscle_group' => 'abs',
                'secondary_muscles' => null,
                'equipment' => 'cable',
                'body_part' => 'waist',
                'category' => 'strength',
                'instructions' => "Step 1: Kneel in front of cable machine.\nStep 2: Hold rope behind head.\nStep 3: Crunch down, bringing elbows to knees.",
            ],
        ];

        foreach ($exercises as $exercise) {
            Exercise::create(array_merge($exercise, [
                'is_custom' => false,
                'user_id' => null,
            ]));
        }

        $this->command->info('✅ Seeded ' . count($exercises) . ' exercises!');
    }
}
```

---

## Step 2.3 — Register the Seeder

**File:** `database/seeders/DatabaseSeeder.php`

Open this file and update the `run` method:

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    public function run(): void
    {
        $this->call([
            ExerciseSeeder::class,
        ]);
    }
}
```

---

## Step 2.4 — Run the Seeder!

```bash
php artisan db:seed
```

> You should see: `✅ Seeded 10 exercises!`

Let's verify the data is in the database:

```bash
php artisan tinker
```

This opens an interactive PHP shell. Type:

```php
App\Models\Exercise::count();
```
> Should return `10`

```php
App\Models\Exercise::first()->toArray();
```
> Should show the Barbell Bench Press with all its data!

```php
App\Models\Exercise::where('muscle_group', 'chest')->pluck('name');
```
> Should return `["Barbell Bench Press", "Incline Dumbbell Press"]`

Type `exit` to leave Tinker.

---

## 🎉 Phase 1 & 2 Complete!

Here's what you've built so far:

| ✅ Done | What |
|---|---|
| Laravel 12 project | Installed and configured |
| MySQL database | `musclo_db` created |
| `.env` configuration | Database, Sanctum, DeepSeek settings |
| Sanctum | Installed for API auth |
| 6 tables | `users`, `routines`, `exercises`, `routine_exercise`, `workout_logs`, `set_data` |
| 6 Models | With full relationships, scopes, casts, and accessors |
| 10 seed exercises | Real exercises with muscle groups and instructions |
| All Controllers | Generated and ready to fill with logic |
| All Form Requests | Generated and ready for validation rules |
| All API Resources | Generated and ready for response formatting |

---

## ✋ Checkpoint

Before we move on to **Phase 3 (Sanctum Authentication)**, please confirm:

1. ✅ Did `php artisan migrate` run without errors?
2. ✅ Did `php artisan db:seed` show "Seeded 10 exercises"?
3. ✅ Did `php artisan tinker` → `Exercise::count()` return `10`?

**When you're ready**, tell me and I'll give you Phase 3 (Authentication) and Phase 4 (The Core Business Logic)! 🚀
