# Musclo — Master Blueprint

> **Version:** 1.0 · **Date:** 2026-03-01
> **Stack:** React 19 + TypeScript 5 + Vite 7 + Tailwind CSS v3.4 + Framer Motion 11 + Zustand 5 | Laravel 12 + Sanctum + MySQL 8 | DeepSeek R1 API

---

## Table of Contents

1. [Product Vision & Premium UI/UX Design System](#1-product-vision--premium-uiux-design-system)
2. [Exact File & Folder Architecture](#2-exact-file--folder-architecture)
3. [Backend Blueprint (Laravel & MySQL)](#3-backend-blueprint-laravel--mysql)
4. [Frontend Blueprint (React & TypeScript)](#4-frontend-blueprint-react--typescript)
5. [The AI Agent Execution Sequence](#5-the-ai-agent-execution-sequence)

---

## 1. Product Vision & Premium UI/UX Design System

### 1.1 Product Vision

Musclo is a hyper-polished fitness tracking and AI coaching web application. It must **surpass** Hevy and Lyfta in every dimension:

| Dimension | Hevy / Lyfta (Current) | Musclo (Target) |
|---|---|---|
| Workout Logging | Functional but flat UI | Animated slide-up logger with haptic-feel feedback, live volume counters |
| Progress Charts | Basic line charts | Animated Recharts with gradient fills, tooltips, muscle heatmaps |
| AI Integration | HevyGPT (basic) | Persistent AI Coach with contextual workout awareness, streaming responses |
| Design Language | Clean but generic | Glassmorphic dark theme, micro-animations on every interaction, premium typography |
| Offline Support | Limited | Full optimistic updates with Zustand persist + queue sync |

### 1.2 Color Palette

#### Dark Mode (Default)

| Token | HSL | Hex | Usage |
|---|---|---|---|
| `--bg-primary` | `240 10% 6%` | `#0e0e11` | App background |
| `--bg-secondary` | `240 8% 10%` | `#17171c` | Cards, panels |
| `--bg-tertiary` | `240 6% 14%` | `#202027` | Inputs, hover states |
| `--bg-elevated` | `240 6% 18%` | `#2a2a33` | Modals, dropdowns |
| `--border-subtle` | `240 6% 20%` | `#303038` | Dividers, outlines |
| `--border-focus` | `265 90% 60%` | `#8b3cf7` | Focus rings |
| `--text-primary` | `0 0% 96%` | `#f5f5f5` | Headings, body |
| `--text-secondary` | `240 5% 60%` | `#9393a0` | Subtext, labels |
| `--text-muted` | `240 4% 40%` | `#62626c` | Placeholders |
| `--accent-primary` | `265 90% 60%` | `#8b3cf7` | Buttons, links, active nav |
| `--accent-primary-hover` | `265 90% 55%` | `#7c2cf0` | Button hover |
| `--accent-secondary` | `180 70% 50%` | `#26c6c6` | Charts accent, badges |
| `--success` | `145 65% 50%` | `#2db870` | PR badges, success toast |
| `--warning` | `40 95% 55%` | `#f0a820` | RIR warnings |
| `--danger` | `0 75% 55%` | `#d94040` | Destructive actions |
| `--gradient-hero` | — | `linear-gradient(135deg, #8b3cf7 0%, #26c6c6 100%)` | Hero sections, CTA buttons |

#### Light Mode

| Token | Hex | Usage |
|---|---|---|
| `--bg-primary` | `#fafafa` | App background |
| `--bg-secondary` | `#ffffff` | Cards |
| `--bg-tertiary` | `#f0f0f3` | Inputs |
| `--text-primary` | `#111118` | Body text |
| `--text-secondary` | `#6b6b78` | Labels |
| `--accent-primary` | `#7028e4` | Buttons (slightly deeper for contrast) |

### 1.3 Typography

```
Font Family:  "Inter" (Google Fonts), fallback: system-ui, -apple-system, sans-serif
Font Weights: 400 (body), 500 (labels), 600 (subheadings), 700 (headings), 800 (display)

Scale (rem):
  --text-xs:    0.75rem / 1rem       (12px) — Badges, timestamps
  --text-sm:    0.875rem / 1.25rem   (14px) — Table cells, helper text
  --text-base:  1rem / 1.5rem        (16px) — Body text
  --text-lg:    1.125rem / 1.75rem   (18px) — Card titles
  --text-xl:    1.25rem / 1.75rem    (20px) — Section headers
  --text-2xl:   1.5rem / 2rem        (24px) — Page titles
  --text-3xl:   1.875rem / 2.25rem   (30px) — Dashboard hero stats
  --text-4xl:   2.25rem / 2.5rem     (36px) — Onboarding display
```

### 1.4 Spacing & Layout

```
Spacing Scale (px): 4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80
Border Radius:
  --radius-sm:   6px   (Badges, tags)
  --radius-md:   10px  (Buttons, inputs)
  --radius-lg:   16px  (Cards)
  --radius-xl:   24px  (Modals, bottom sheets)
  --radius-full: 9999px (Avatars, pills)
Container:       max-w-[1280px] mx-auto px-4 md:px-6 lg:px-8
Sidebar Width:   280px (desktop), hidden (mobile, replaced by bottom-nav)
```

### 1.5 Framer Motion — Animation Tokens

Every animation in the app MUST use these named presets for consistency:

```typescript
// src/lib/motion.ts

export const MOTION = {
  // Shared spring config
  spring: { type: 'spring', stiffness: 300, damping: 26, mass: 0.8 },
  springGentle: { type: 'spring', stiffness: 200, damping: 24, mass: 1 },
  springSnappy: { type: 'spring', stiffness: 400, damping: 28, mass: 0.6 },

  // Page transitions
  pageEnter: {
    initial: { opacity: 0, y: 16 },
    animate: { opacity: 1, y: 0 },
    exit: { opacity: 0, y: -8 },
    transition: { duration: 0.3, ease: [0.25, 0.1, 0.25, 1] },
  },

  // Slide-up (workout logger, bottom sheets)
  slideUp: {
    initial: { y: '100%' },
    animate: { y: 0 },
    exit: { y: '100%' },
    transition: { type: 'spring', stiffness: 300, damping: 30 },
  },

  // Modal (AI Chat, confirmations)
  modalOverlay: {
    initial: { opacity: 0 },
    animate: { opacity: 1 },
    exit: { opacity: 0 },
    transition: { duration: 0.2 },
  },
  modalContent: {
    initial: { opacity: 0, scale: 0.95, y: 10 },
    animate: { opacity: 1, scale: 1, y: 0 },
    exit: { opacity: 0, scale: 0.97, y: 5 },
    transition: { type: 'spring', stiffness: 350, damping: 28 },
  },

  // Card / list items (stagger children)
  staggerContainer: {
    animate: { transition: { staggerChildren: 0.04, delayChildren: 0.06 } },
  },
  staggerItem: {
    initial: { opacity: 0, y: 12 },
    animate: { opacity: 1, y: 0 },
    transition: { duration: 0.25, ease: 'easeOut' },
  },

  // Micro-interactions
  tapScale: { whileTap: { scale: 0.97 }, transition: { duration: 0.1 } },
  hoverLift: { whileHover: { y: -2, boxShadow: '0 8px 30px rgba(139,60,247,0.15)' } },
  successPulse: {
    animate: { scale: [1, 1.15, 1], opacity: [1, 0.8, 1] },
    transition: { duration: 0.4 },
  },
  numberTick: {
    initial: { opacity: 0, y: -10 },
    animate: { opacity: 1, y: 0 },
    transition: { type: 'spring', stiffness: 500, damping: 25 },
  },
} as const;
```

### 1.6 Specific UI Interactions

| Interaction | Animation | Details |
|---|---|---|
| **Workout Logger opens** | `slideUp` | Full-height bottom sheet slides from below; overlay `modalOverlay` fades in behind |
| **Adding a set row** | `staggerItem` + `layoutId` | New row slides down; existing rows reflow with Framer `layout` prop |
| **Completing a set** | `successPulse` on checkmark + `numberTick` on volume counter | Checkmark icon scales up with green glow, volume counter animates numerically |
| **PR Banner** | `slideUp` + confetti burst | Gold banner with "🏆 New Personal Record!" slides up from bottom for 3 seconds |
| **AI Chat Modal** | `modalOverlay` + `modalContent` | Backdrop blurs in, chat panel scales from 95% to 100% |
| **AI Streaming Response** | Typing indicator → character-by-character fade | Each token fades in via `motion.span` with 30ms stagger |
| **Navigation (page)** | `pageEnter` with `AnimatePresence` `mode="wait"` | Current page fades/slides out, new page fades/slides in |
| **Delete Swipe** | `drag="x"` + threshold at -80px | Card translates with red delete zone revealed behind; spring back or remove with `exit={{ x: -400, opacity: 0 }}` |
| **Tab switching** | `layoutId="tab-indicator"` | Underline pill morphs between tabs using shared layout animation |
| **Button press** | `tapScale` | Subtle scale-down on press gives physical feedback |
| **Card hover (desktop)** | `hoverLift` | Card lifts 2px with subtle purple shadow |
| **Toast notification** | Slide in from top-right, auto-dismiss after 4s | `initial={{ x: 100, opacity: 0 }}` → `animate={{ x: 0, opacity: 1 }}` |
| **Skeleton loaders** | Shimmer gradient animation | `animate={{ backgroundPosition: ['200% 0', '-200% 0'] }}` over 1.5s loop |

### 1.7 UI Component Library

Use **Radix UI Primitives** (headless, unstyled) + custom Tailwind styling. Do NOT use shadcn/ui CLI; instead manually build each component for full control.

#### Core Components to Build

| Component | Source | Notes |
|---|---|---|
| `Button` | Custom | Variants: `primary`, `secondary`, `ghost`, `danger`. Sizes: `sm`, `md`, `lg`. Includes `tapScale` animation. |
| `Input` | Custom | With floating label, error state, focus ring using `--border-focus` |
| `Select` | Radix `Select` | Styled dropdown for exercise picker, muscle group filter |
| `Dialog` / `Modal` | Radix `Dialog` | Wrapped with Framer `modalOverlay` + `modalContent` |
| `Sheet` | Radix `Dialog` (side) | For workout logger (bottom sheet on mobile) |
| `Tabs` | Radix `Tabs` | With `layoutId` animated indicator |
| `Toast` | Radix `Toast` | Animated slide-in, stacked, auto-dismiss |
| `Tooltip` | Radix `Tooltip` | For icon buttons, chart data points |
| `Avatar` | Custom | Gradient fallback with user initials |
| `Badge` | Custom | Variants: `success`, `warning`, `info`, `pr` (gold) |
| `Card` | Custom | With `hoverLift`, glassmorphic variant for dashboard stats |
| `Skeleton` | Custom | Shimmer animation component |
| `Chart` | Recharts wrapper | `AreaChart`, `BarChart`, `RadarChart` pre-styled |
| `NumberTicker` | Custom + Framer | Animates numeric value changes (volume, PR) |
| `BottomNav` | Custom | Mobile only; icons with active indicator pill |
| `Sidebar` | Custom | Desktop only; collapsible with logo, nav links, user avatar |
| `EmptyState` | Custom | Illustration + CTA for "No workouts yet" scenarios |
| `ConfirmDialog` | Radix `AlertDialog` | "Are you sure?" destructive confirmation |

### 1.8 Logo Assets

The Musclo logo features a **flexed bicep silhouette** in a bold, minimal style.

| File | Location | Usage |
|---|---|---|
| `logo.png` | `public/logo.png` | Full horizontal logo (bicep icon + "MUSCLO" wordmark) — used in Sidebar header, login/register pages |
| `logo-icon.png` | `public/logo-icon.png` | Icon-only square (bicep silhouette) — used in BottomNav brand slot, browser favicon, PWA icon |

- On **dark mode**: Logo renders white (apply CSS `filter: invert(1)` or serve a white variant)
- On **light mode**: Logo renders black (default)
- Sidebar displays `logo.png` at 140×40px with `object-contain`
- Favicon: Convert `logo-icon.png` to `favicon.svg` (or `.ico`) during build

---

## 2. Exact File & Folder Architecture

### 2.1 Backend — `musclo-api/` (Laravel 12)

```
musclo-api/
├── app/
│   ├── Http/
│   │   ├── Controllers/
│   │   │   ├── Auth/
│   │   │   │   ├── RegisterController.php
│   │   │   │   ├── LoginController.php
│   │   │   │   └── LogoutController.php
│   │   │   ├── RoutineController.php
│   │   │   ├── ExerciseController.php
│   │   │   ├── WorkoutLogController.php
│   │   │   └── AICoachController.php
│   │   ├── Requests/
│   │   │   ├── Auth/
│   │   │   │   ├── RegisterRequest.php
│   │   │   │   └── LoginRequest.php
│   │   │   ├── StoreRoutineRequest.php
│   │   │   ├── UpdateRoutineRequest.php
│   │   │   ├── StoreWorkoutLogRequest.php
│   │   │   └── AskAIRequest.php
│   │   ├── Resources/
│   │   │   ├── UserResource.php
│   │   │   ├── RoutineResource.php
│   │   │   ├── ExerciseResource.php
│   │   │   ├── WorkoutLogResource.php
│   │   │   ├── SetDataResource.php
│   │   │   └── AIResponseResource.php
│   │   └── Middleware/
│   │       └── ForceJsonResponse.php
│   ├── Models/
│   │   ├── User.php
│   │   ├── Routine.php
│   │   ├── Exercise.php
│   │   ├── WorkoutLog.php
│   │   ├── SetData.php
│   │   └── RoutineExercise.php
│   ├── Console/
│   │   └── Commands/
│   │       └── ImportExercisesCommand.php    # Fetches 1500+ exercises from ExerciseDB API
│   └── Services/
│       ├── DeepSeekService.php
│       └── ExerciseDBService.php             # ExerciseDB API client
├── config/
│   └── deepseek.php
├── database/
│   ├── migrations/
│   │   ├── 0001_01_01_000000_create_users_table.php        (default)
│   │   ├── 2026_03_02_000001_create_routines_table.php
│   │   ├── 2026_03_02_000002_create_exercises_table.php
│   │   ├── 2026_03_02_000003_create_routine_exercise_table.php
│   │   ├── 2026_03_02_000004_create_workout_logs_table.php
│   │   └── 2026_03_02_000005_create_set_data_table.php
│   ├── seeders/
│   │   ├── DatabaseSeeder.php
│   │   └── ExerciseSeeder.php
│   └── factories/
│       ├── RoutineFactory.php
│       ├── WorkoutLogFactory.php
│       └── SetDataFactory.php
├── routes/
│   └── api.php
├── .env
└── composer.json
```

### 2.2 Frontend — `musclo-spa/` (React + Vite + TypeScript)

```
musclo-spa/
├── public/
│   └── favicon.svg
├── src/
│   ├── main.tsx
│   ├── App.tsx
│   ├── index.css                          # Tailwind directives + CSS custom properties
│   ├── api/
│   │   ├── axios.ts                       # Axios instance + interceptors
│   │   ├── auth.ts                        # login, register, logout, getUser
│   │   ├── routines.ts                    # CRUD routines
│   │   ├── exercises.ts                   # list, search exercises
│   │   ├── workouts.ts                    # save, history, lastLog
│   │   └── ai.ts                          # sendMessage (streaming)
│   ├── stores/
│   │   ├── useAuthStore.ts
│   │   ├── useWorkoutStore.ts             # Active workout session state
│   │   ├── useRoutineStore.ts
│   │   ├── useAIChatStore.ts
│   │   └── useThemeStore.ts
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   ├── useExercises.ts
│   │   ├── useWorkoutHistory.ts
│   │   └── useMediaQuery.ts
│   ├── lib/
│   │   ├── motion.ts                      # MOTION animation presets
│   │   ├── utils.ts                       # cn(), formatWeight(), formatDate()
│   │   └── constants.ts                   # API_URL, MUSCLE_GROUPS enum, Exercise type (with gifUrl), etc.
│   ├── components/
│   │   ├── ui/
│   │   │   ├── Button.tsx
│   │   │   ├── Input.tsx
│   │   │   ├── Select.tsx
│   │   │   ├── Modal.tsx
│   │   │   ├── Sheet.tsx
│   │   │   ├── Tabs.tsx
│   │   │   ├── Toast.tsx
│   │   │   ├── Tooltip.tsx
│   │   │   ├── Avatar.tsx
│   │   │   ├── Badge.tsx
│   │   │   ├── Card.tsx
│   │   │   ├── Skeleton.tsx
│   │   │   ├── NumberTicker.tsx
│   │   │   ├── EmptyState.tsx
│   │   │   └── ConfirmDialog.tsx
│   │   ├── layout/
│   │   │   ├── AppLayout.tsx              # Sidebar + main + bottom nav
│   │   │   ├── Sidebar.tsx
│   │   │   ├── BottomNav.tsx
│   │   │   ├── TopBar.tsx
│   │   │   └── PageTransition.tsx         # AnimatePresence wrapper
│   │   ├── auth/
│   │   │   ├── LoginForm.tsx
│   │   │   ├── RegisterForm.tsx
│   │   │   └── ProtectedRoute.tsx
│   │   ├── dashboard/
│   │   │   ├── DashboardPage.tsx
│   │   │   ├── StatsCard.tsx              # Glassmorphic stat cards
│   │   │   ├── RecentWorkouts.tsx
│   │   │   ├── WeeklyVolumeChart.tsx
│   │   │   └── MuscleHeatmap.tsx
│   │   ├── routines/
│   │   │   ├── RoutinesPage.tsx
│   │   │   ├── RoutineCard.tsx
│   │   │   ├── RoutineBuilder.tsx         # Create / edit routine
│   │   │   └── ExercisePicker.tsx         # Search + add exercises
│   │   ├── workout/
│   │   │   ├── ActiveWorkout.tsx          # Full-screen workout logger
│   │   │   ├── SetRow.tsx                 # Individual set input row
│   │   │   ├── ExerciseBlock.tsx          # Groups sets by exercise
│   │   │   ├── RestTimer.tsx
│   │   │   ├── WorkoutSummary.tsx         # Post-workout stats
│   │   │   └── PRBanner.tsx              # Personal record celebration
│   │   ├── history/
│   │   │   ├── HistoryPage.tsx
│   │   │   ├── WorkoutHistoryCard.tsx
│   │   │   └── ProgressCharts.tsx         # Recharts implementations
│   │   ├── exercises/
│   │   │   ├── ExercisesPage.tsx
│   │   │   └── ExerciseDetailModal.tsx
│   │   └── ai/
│   │       ├── AIChatModal.tsx            # Floating chat panel
│   │       ├── ChatMessage.tsx
│   │       ├── StreamingText.tsx          # Token-by-token render
│   │       └── AIChatFAB.tsx             # Floating action button
│   ├── pages/
│   │   ├── LoginPage.tsx
│   │   ├── RegisterPage.tsx
│   │   ├── DashboardPage.tsx
│   │   ├── RoutinesPage.tsx
│   │   ├── WorkoutPage.tsx
│   │   ├── HistoryPage.tsx
│   │   ├── ExercisesPage.tsx
│   │   └── NotFoundPage.tsx
│   └── router/
│       └── index.tsx                      # React Router v6 config
├── tailwind.config.ts
├── postcss.config.js
├── tsconfig.json
├── vite.config.ts
├── package.json
└── index.html
```

---

## 3. Backend Blueprint (Laravel & MySQL)

### 3.1 Artisan Generation Commands

Execute these in strict order inside `musclo-api/`:

```bash
# Models + Migrations
php artisan make:model Routine -mf
php artisan make:model Exercise -mf
php artisan make:model RoutineExercise -m
php artisan make:model WorkoutLog -mf
php artisan make:model SetData -mf

# Controllers
php artisan make:controller Auth/RegisterController
php artisan make:controller Auth/LoginController
php artisan make:controller Auth/LogoutController
php artisan make:controller RoutineController --api
php artisan make:controller ExerciseController --api
php artisan make:controller WorkoutLogController --api
php artisan make:controller AICoachController

# Form Requests
php artisan make:request Auth/RegisterRequest
php artisan make:request Auth/LoginRequest
php artisan make:request StoreRoutineRequest
php artisan make:request UpdateRoutineRequest
php artisan make:request StoreWorkoutLogRequest
php artisan make:request AskAIRequest

# API Resources
php artisan make:resource UserResource
php artisan make:resource RoutineResource
php artisan make:resource ExerciseResource
php artisan make:resource WorkoutLogResource
php artisan make:resource SetDataResource
php artisan make:resource AIResponseResource

# Seeder
php artisan make:seeder ExerciseSeeder

# Middleware
php artisan make:middleware ForceJsonResponse

# Config
# Manually create config/deepseek.php
```

### 3.2 Database Migrations (Production-Ready)

#### `create_routines_table.php`

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

#### `create_exercises_table.php`

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
            $table->string('exercisedb_id', 20)->nullable()->unique(); // ExerciseDB API ID
            $table->string('name', 150);
            $table->string('muscle_group', 50);          // e.g. "Chest", "Back" (from targetMuscles)
            $table->string('secondary_muscles', 150)->nullable(); // e.g. "Triceps, Shoulders"
            $table->string('equipment', 50)->nullable();  // e.g. "Barbell", "Dumbbell"
            $table->string('body_part', 50)->nullable();  // e.g. "back", "upper arms" (ExerciseDB bodyParts)
            $table->string('category', 30)->default('strength'); // strength, cardio, stretch
            $table->text('instructions')->nullable();
            $table->string('gif_url', 500)->nullable();   // ExerciseDB GIF CDN URL
            $table->boolean('is_custom')->default(false);
            $table->foreignId('user_id')->nullable()->constrained()->onDelete('cascade'); // null = global
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

#### `create_routine_exercise_table.php` (Pivot)

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

#### `create_workout_logs_table.php`

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
            $table->string('name', 100)->nullable();       // Snapshot of routine name
            $table->dateTime('started_at');
            $table->dateTime('completed_at')->nullable();
            $table->unsignedInteger('duration_seconds')->default(0);
            $table->decimal('total_volume', 12, 2)->default(0); // sum(weight * reps)
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

#### `create_set_data_table.php`

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
            $table->string('set_type', 20)->default('working'); // working, warmup, dropset, failure
            $table->decimal('weight_kg', 8, 2);
            $table->unsignedSmallInteger('reps');
            $table->unsignedTinyInteger('rir')->nullable();      // Reps in Reserve (0-5)
            $table->unsignedTinyInteger('rpe')->nullable();      // Rate of Perceived Exertion (1-10)
            $table->boolean('is_pr')->default(false);
            $table->timestamps();

            $table->index(['workout_log_id', 'exercise_id']);
            $table->index(['exercise_id', 'created_at']);        // For PR lookups
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('set_data');
    }
};
```

### 3.3 Eloquent Model Relationships

#### `User.php` (Additions)

```php
// In User model:
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

#### `Routine.php`

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

#### `Exercise.php`

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

    // Scope: get global + user-specific exercises
    public function scopeAvailableTo($query, int $userId)
    {
        return $query->where(function ($q) use ($userId) {
            $q->where('is_custom', false)
              ->orWhere('user_id', $userId);
        });
    }
}
```

#### `WorkoutLog.php`

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

    // Compute total volume from sets
    public function calculateTotalVolume(): float
    {
        return $this->sets->sum(fn (SetData $set) =>
            $set->set_type !== 'warmup' ? $set->weight_kg * $set->reps : 0
        );
    }
}
```

#### `SetData.php`

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

    // Volume for this single set
    public function getVolumeAttribute(): float
    {
        return (float) $this->weight_kg * $this->reps;
    }
}
```

### 3.4 API Endpoint Documentation

#### Authentication Endpoints (Public)

| Method | URI | Request Body | Response | Notes |
|--------|-----|-------------|----------|-------|
| `POST` | `/api/register` | `{ name, email, password, password_confirmation }` | `{ user: UserResource, token: string }` | Creates user + issues Sanctum token |
| `POST` | `/api/login` | `{ email, password }` | `{ user: UserResource, token: string }` | Returns plaintext token |
| `POST` | `/api/logout` | — | `{ message: "Logged out" }` | Revokes current token |
| `GET` | `/sanctum/csrf-cookie` | — | Sets XSRF-TOKEN cookie | Required before SPA auth calls |

#### Routine Endpoints (Auth Required)

| Method | URI | Request Body | Response |
|--------|-----|-------------|----------|
| `GET` | `/api/routines` | — | `{ data: RoutineResource[] }` |
| `POST` | `/api/routines` | `{ name, notes?, exercises: [{ id, sort_order, target_sets, target_reps }] }` | `{ data: RoutineResource }` |
| `GET` | `/api/routines/{id}` | — | `{ data: RoutineResource }` (with exercises) |
| `PUT` | `/api/routines/{id}` | Same as POST | `{ data: RoutineResource }` |
| `DELETE` | `/api/routines/{id}` | — | `204 No Content` |
| `GET` | `/api/routines/{id}/last-log` | — | `{ data: WorkoutLogResource }` (with sets grouped by exercise) |

#### Exercise Endpoints (Auth Required)

| Method | URI | Query Params | Response |
|--------|-----|-------------|----------|
| `GET` | `/api/exercises` | `?search=&muscle_group=&equipment=&page=` | `{ data: ExerciseResource[], meta: pagination }` |
| `POST` | `/api/exercises` | `{ name, muscle_group, equipment?, instructions? }` | `{ data: ExerciseResource }` (custom) |

#### Workout Endpoints (Auth Required)

| Method | URI | Request Body | Response |
|--------|-----|-------------|----------|
| `POST` | `/api/workouts` | See payload below | `{ data: WorkoutLogResource }` |
| `GET` | `/api/workouts/history` | `?page=&per_page=15` | `{ data: WorkoutLogResource[], meta: pagination }` |
| `GET` | `/api/workouts/{id}` | — | `{ data: WorkoutLogResource }` (with sets) |
| `DELETE` | `/api/workouts/{id}` | — | `204 No Content` |
| `GET` | `/api/workouts/stats` | `?period=week\|month\|year` | `{ total_workouts, total_volume, avg_duration, muscle_distribution: {} }` |
| `GET` | `/api/workouts/exercise/{exerciseId}/history` | `?limit=20` | `{ data: [{ date, sets: SetDataResource[] }] }` |

**POST `/api/workouts` — Full Payload:**

```json
{
  "routine_id": 1,
  "name": "Push Day",
  "started_at": "2026-03-01T14:00:00Z",
  "completed_at": "2026-03-01T15:15:00Z",
  "duration_seconds": 4500,
  "notes": "Felt strong today",
  "sets": [
    {
      "exercise_id": 12,
      "set_number": 1,
      "set_type": "warmup",
      "weight_kg": 40.00,
      "reps": 12,
      "rir": null,
      "rpe": null
    },
    {
      "exercise_id": 12,
      "set_number": 2,
      "set_type": "working",
      "weight_kg": 80.00,
      "reps": 8,
      "rir": 2,
      "rpe": 8
    }
  ]
}
```

#### AI Chat Endpoint (Auth Required)

| Method | URI | Request Body | Response |
|--------|-----|-------------|----------|
| `POST` | `/api/chat` | `{ message, context: { exercise?, weight?, reps?, rir?, history_summary? } }` | SSE stream or `{ data: { advice: string, suggestions: string[] } }` |

### 3.5 Laravel Routes (`routes/api.php`)

```php
<?php

use App\Http\Controllers\Auth\RegisterController;
use App\Http\Controllers\Auth\LoginController;
use App\Http\Controllers\Auth\LogoutController;
use App\Http\Controllers\RoutineController;
use App\Http\Controllers\ExerciseController;
use App\Http\Controllers\WorkoutLogController;
use App\Http\Controllers\AICoachController;
use Illuminate\Support\Facades\Route;

// Public
Route::post('/register', [RegisterController::class, 'store']);
Route::post('/login', [LoginController::class, 'store']);

// Protected
Route::middleware('auth:sanctum')->group(function () {
    // Auth
    Route::post('/logout', [LogoutController::class, 'destroy']);
    Route::get('/user', fn (Request $request) => new \App\Http\Resources\UserResource($request->user()));

    // Routines
    Route::apiResource('routines', RoutineController::class);
    Route::get('/routines/{routine}/last-log', [RoutineController::class, 'lastLog']);

    // Exercises
    Route::get('/exercises', [ExerciseController::class, 'index']);
    Route::post('/exercises', [ExerciseController::class, 'store']);

    // Workouts
    Route::post('/workouts', [WorkoutLogController::class, 'store']);
    Route::get('/workouts/history', [WorkoutLogController::class, 'history']);
    Route::get('/workouts/stats', [WorkoutLogController::class, 'stats']);
    Route::get('/workouts/exercise/{exercise}/history', [WorkoutLogController::class, 'exerciseHistory']);
    Route::get('/workouts/{workoutLog}', [WorkoutLogController::class, 'show']);
    Route::delete('/workouts/{workoutLog}', [WorkoutLogController::class, 'destroy']);

    // AI Coach
    Route::post('/chat', [AICoachController::class, 'ask']);
});
```

### 3.6 DeepSeek Service (`app/Services/DeepSeekService.php`)

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
        $this->apiKey = config('deepseek.api_key');
        $this->baseUrl = config('deepseek.base_url', 'https://api.deepseek.com');
        $this->model = config('deepseek.model', 'deepseek-reasoner');
    }

    /**
     * Build the system prompt with user workout context injected.
     */
    private function buildSystemPrompt(array $context): string
    {
        $base = "You are Musclo AI Coach, an expert personal trainer and exercise scientist. "
            . "You provide advice grounded in progressive overload, periodization, and exercise science. "
            . "Be concise, actionable, and encouraging. Use data provided to give specific recommendations. "
            . "Never recommend dangerous overloads. Always prioritize form and safety.";

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
     * Send a message to DeepSeek and return the response.
     * Returns ['advice' => string, 'suggestions' => string[]] on success.
     * Returns ['error' => string] on failure.
     */
    public function ask(string $message, array $context = []): array
    {
        $systemPrompt = $this->buildSystemPrompt($context);

        try {
            $response = Http::withToken($this->apiKey)
                ->timeout(30)
                ->retry(2, 1000)
                ->post("{$this->baseUrl}/v1/chat/completions", [
                    'model' => $this->model,
                    'messages' => [
                        ['role' => 'system', 'content' => $systemPrompt],
                        ['role' => 'user', 'content' => $message],
                    ],
                    'temperature' => 0.7,
                    'max_tokens' => 800,
                ]);

            if ($response->successful()) {
                $content = $response->json('choices.0.message.content', '');
                return [
                    'advice' => $content,
                    'suggestions' => $this->extractSuggestions($content),
                ];
            }

            Log::error('DeepSeek API error', [
                'status' => $response->status(),
                'body' => $response->body(),
            ]);

            return ['error' => 'AI service returned an error. Please try again.'];

        } catch (\Illuminate\Http\Client\ConnectionException $e) {
            Log::error('DeepSeek connection timeout', ['exception' => $e->getMessage()]);
            return ['error' => 'AI service is temporarily unavailable. Please try again in a moment.'];
        }
    }

    /**
     * Extract follow-up suggestions from AI response.
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

### 3.7 Config File (`config/deepseek.php`)

```php
<?php

return [
    'api_key' => env('DEEPSEEK_API_KEY', ''),
    'base_url' => env('DEEPSEEK_BASE_URL', 'https://api.deepseek.com'),
    'model' => env('DEEPSEEK_MODEL', 'deepseek-reasoner'),
];
```

### 3.8 AICoachController

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\AskAIRequest;
use App\Services\DeepSeekService;
use Illuminate\Http\JsonResponse;

class AICoachController extends Controller
{
    public function __construct(private DeepSeekService $deepSeek) {}

    public function ask(AskAIRequest $request): JsonResponse
    {
        $result = $this->deepSeek->ask(
            $request->validated('message'),
            $request->validated('context', [])
        );

        if (isset($result['error'])) {
            return response()->json(['error' => $result['error']], 503);
        }

        return response()->json(['data' => $result]);
س    }
}
```

### 3.9 Key Form Request Validations

#### `AskAIRequest.php`

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
            'message' => 'required|string|max:1000',
            'context' => 'sometimes|array',
            'context.exercise' => 'sometimes|string|max:100',
            'context.weight' => 'sometimes|numeric|min:0|max:999',
            'context.reps' => 'sometimes|integer|min:0|max:100',
            'context.rir' => 'sometimes|integer|min:0|max:10',
            'context.history_summary' => 'sometimes|string|max:2000',
        ];
    }
}
```

#### `StoreWorkoutLogRequest.php`

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
            'routine_id' => 'nullable|exists:routines,id',
            'name' => 'nullable|string|max:100',
            'started_at' => 'required|date',
            'completed_at' => 'nullable|date|after:started_at',
            'duration_seconds' => 'required|integer|min:0',
            'notes' => 'nullable|string|max:1000',
            'sets' => 'required|array|min:1',
            'sets.*.exercise_id' => 'required|exists:exercises,id',
            'sets.*.set_number' => 'required|integer|min:1',
            'sets.*.set_type' => 'required|in:working,warmup,dropset,failure',
            'sets.*.weight_kg' => 'required|numeric|min:0|max:9999',
            'sets.*.reps' => 'required|integer|min:0|max:999',
            'sets.*.rir' => 'nullable|integer|min:0|max:10',
            'sets.*.rpe' => 'nullable|integer|min:1|max:10',
        ];
    }
}
```

### 3.10 WorkoutLogController (Core Logic)

```php
<?php

namespace App\Http\Controllers;

use App\Http\Requests\StoreWorkoutLogRequest;
use App\Http\Resources\WorkoutLogResource;
use App\Models\WorkoutLog;
use App\Models\SetData;
use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\DB;

class WorkoutLogController extends Controller
{
    public function store(StoreWorkoutLogRequest $request): JsonResponse
    {
        $validated = $request->validated();

        $workoutLog = DB::transaction(function () use ($validated, $request) {
            $log = WorkoutLog::create([
                'user_id' => $request->user()->id,
                'routine_id' => $validated['routine_id'] ?? null,
                'name' => $validated['name'] ?? null,
                'started_at' => $validated['started_at'],
                'completed_at' => $validated['completed_at'] ?? now(),
                'duration_seconds' => $validated['duration_seconds'],
                'notes' => $validated['notes'] ?? null,
            ]);

            foreach ($validated['sets'] as $setData) {
                $set = $log->sets()->create($setData);

                // Check for PR: is this the highest volume (weight × reps) for this exercise?
                $previousBest = SetData::where('exercise_id', $set->exercise_id)
                    ->where('id', '!=', $set->id)
                    ->where('set_type', 'working')
                    ->selectRaw('MAX(weight_kg * reps) as max_volume')
                    ->value('max_volume') ?? 0;

                $currentVolume = $set->weight_kg * $set->reps;
                if ($set->set_type === 'working' && $currentVolume > $previousBest) {
                    $set->update(['is_pr' => true]);
                }
            }

            // Calculate and store total volume
            $log->update(['total_volume' => $log->calculateTotalVolume()]);

            return $log->load('sets.exercise');
        });

        return response()->json(
            ['data' => new WorkoutLogResource($workoutLog)],
            201
        );
    }

    public function history(Request $request): JsonResponse
    {
        $logs = $request->user()
            ->workoutLogs()
            ->with('sets.exercise', 'routine')
            ->orderByDesc('started_at')
            ->paginate($request->integer('per_page', 15));

        return WorkoutLogResource::collection($logs)->response();
    }

    public function show(WorkoutLog $workoutLog): JsonResponse
    {
        $this->authorize('view', $workoutLog);
        $workoutLog->load('sets.exercise', 'routine');
        return response()->json(['data' => new WorkoutLogResource($workoutLog)]);
    }

    public function destroy(WorkoutLog $workoutLog): JsonResponse
    {
        $this->authorize('delete', $workoutLog);
        $workoutLog->delete();
        return response()->json(null, 204);
    }

    public function stats(Request $request): JsonResponse
    {
        $period = $request->input('period', 'week');
        $days = match($period) {
            'week' => 7, 'month' => 30, 'year' => 365, default => 7,
        };

        $query = $request->user()->workoutLogs()
            ->where('started_at', '>=', now()->subDays($days));

        $logs = $query->with('sets.exercise')->get();

        $muscleDistribution = $logs->flatMap->sets
            ->groupBy(fn ($set) => $set->exercise->muscle_group)
            ->map->count();

        return response()->json([
            'data' => [
                'total_workouts' => $logs->count(),
                'total_volume' => $logs->sum('total_volume'),
                'avg_duration' => (int) $logs->avg('duration_seconds'),
                'muscle_distribution' => $muscleDistribution,
            ],
        ]);
    }

    public function exerciseHistory(Request $request, int $exerciseId): JsonResponse
    {
        $limit = $request->integer('limit', 20);

        $sets = SetData::where('exercise_id', $exerciseId)
            ->whereHas('workoutLog', fn ($q) => $q->where('user_id', $request->user()->id))
            ->with('workoutLog:id,started_at')
            ->orderByDesc('created_at')
            ->limit($limit * 5)
            ->get()
            ->groupBy(fn ($s) => $s->workoutLog->started_at->toDateString());

        return response()->json(['data' => $sets]);
    }
}
```

### 3.11 ForceJsonResponse Middleware

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class ForceJsonResponse
{
    public function handle(Request $request, Closure $next)
    {
        $request->headers->set('Accept', 'application/json');
        return $next($request);
    }
}
```

Register in `bootstrap/app.php`:

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->api(prepend: [
        \App\Http\Middleware\ForceJsonResponse::class,
    ]);
    $middleware->statefulApi();
})
```

### 3.12 CORS Configuration

In `config/cors.php`:

```php
'paths' => ['api/*', 'sanctum/csrf-cookie'],
'allowed_origins' => [env('FRONTEND_URL', 'http://localhost:5173')],
'supports_credentials' => true,
```

In `.env`:

```env
FRONTEND_URL=http://localhost:5173
SANCTUM_STATEFUL_DOMAINS=localhost:5173
SESSION_DOMAIN=localhost
```

---

### 3.13 ExerciseDB API Service (`app/Services/ExerciseDBService.php`)

This service fetches exercises from the free, open-source [ExerciseDB API v1](https://www.exercisedb.dev). It provides 1,500+ exercises with animated GIF demonstrations.

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
     * Yields arrays of exercises for memory-efficient processing.
     */
    public function fetchAllExercises(): \Generator
    {
        $offset = 0;
        $totalPages = 1;

        do {
            $response = Http::timeout(30)
                ->retry(3, 2000)
                ->get("{$this->baseUrl}/exercises", [
                    'limit' => $this->limit,
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
            $totalPages = $json['metadata']['totalPages'] ?? 1;
            $currentPage = $json['metadata']['currentPage'] ?? 1;

            yield $json['data'] ?? [];

            $offset += $this->limit;
        } while ($currentPage < $totalPages);
    }

    /**
     * Fetch a single exercise by ExerciseDB ID.
     */
    public function fetchExercise(string $exercisedbId): ?array
    {
        $response = Http::timeout(15)
            ->retry(2, 1000)
            ->get("{$this->baseUrl}/exercises/{$exercisedbId}");

        if ($response->successful()) {
            return $response->json('data');
        }

        return null;
    }

    /**
     * Fetch available muscle groups from ExerciseDB.
     */
    public function fetchMuscles(): array
    {
        $response = Http::timeout(15)
            ->get("{$this->baseUrl}/muscles");

        return $response->successful() ? $response->json('data', []) : [];
    }

    /**
     * Fetch available equipment types from ExerciseDB.
     */
    public function fetchEquipments(): array
    {
        $response = Http::timeout(15)
            ->get("{$this->baseUrl}/equipments");

        return $response->successful() ? $response->json('data', []) : [];
    }

    /**
     * Fetch available body parts from ExerciseDB.
     */
    public function fetchBodyParts(): array
    {
        $response = Http::timeout(15)
            ->get("{$this->baseUrl}/bodyparts");

        return $response->successful() ? $response->json('data', []) : [];
    }
}
```

### 3.14 Import Exercises Command (`app/Console/Commands/ImportExercisesCommand.php`)

This replaces the static `ExerciseSeeder` — it pulls all 1,500+ exercises from ExerciseDB including animated GIF URLs.

```php
<?php

namespace App\Console\Commands;

use App\Models\Exercise;
use App\Services\ExerciseDBService;
use Illuminate\Console\Command;

class ImportExercisesCommand extends Command
{
    protected $signature = 'exercises:import {--fresh : Delete all non-custom exercises before importing}';
    protected $description = 'Import exercises from ExerciseDB API (1500+ exercises with GIFs)';

    public function handle(ExerciseDBService $service): int
    {
        if ($this->option('fresh')) {
            $deleted = Exercise::where('is_custom', false)->delete();
            $this->info("Deleted {$deleted} existing global exercises.");
        }

        $this->info('Fetching exercises from ExerciseDB API...');
        $bar = $this->output->createProgressBar();
        $bar->start();

        $imported = 0;
        $updated = 0;

        foreach ($service->fetchAllExercises() as $batch) {
            foreach ($batch as $exerciseData) {
                $result = Exercise::updateOrCreate(
                    ['exercisedb_id' => $exerciseData['exerciseId']],
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
        $this->newLine();
        $this->info("✅ Done! Imported: {$imported}, Updated: {$updated}");

        return Command::SUCCESS;
    }
}
```

**Usage:**

```bash
# First-time import
php artisan exercises:import

# Re-import (updates existing, adds new)
php artisan exercises:import

# Fresh import (deletes all non-custom exercises first)
php artisan exercises:import --fresh
```

### 3.15 ExerciseResource (Updated)

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

---

## 4. Frontend Blueprint (React & TypeScript)

### 4.1 Zustand Store Structures

#### `useAuthStore.ts`

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import { apiClient } from '../api/axios';

interface User {
  id: number;
  name: string;
  email: string;
}

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  isLoading: boolean;

  login: (email: string, password: string) => Promise<void>;
  register: (name: string, email: string, password: string, password_confirmation: string) => Promise<void>;
  logout: () => Promise<void>;
  fetchUser: () => Promise<void>;
  setToken: (token: string) => void;
  reset: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set, get) => ({
      user: null,
      token: null,
      isAuthenticated: false,
      isLoading: false,

      login: async (email, password) => {
        set({ isLoading: true });
        const { data } = await apiClient.post('/login', { email, password });
        set({ user: data.user, token: data.token, isAuthenticated: true, isLoading: false });
      },

      register: async (name, email, password, password_confirmation) => {
        set({ isLoading: true });
        const { data } = await apiClient.post('/register', { name, email, password, password_confirmation });
        set({ user: data.user, token: data.token, isAuthenticated: true, isLoading: false });
      },

      logout: async () => {
        try { await apiClient.post('/logout'); } catch {}
        set({ user: null, token: null, isAuthenticated: false });
      },

      fetchUser: async () => {
        try {
          const { data } = await apiClient.get('/user');
          set({ user: data.data, isAuthenticated: true });
        } catch {
          get().reset();
        }
      },

      setToken: (token) => set({ token }),
      reset: () => set({ user: null, token: null, isAuthenticated: false }),
    }),
    { name: 'musclo-auth', partialize: (state) => ({ token: state.token }) }
  )
);
```

#### `useWorkoutStore.ts` — Active Workout Session

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface ActiveSet {
  id: string;                // client-side UUID
  exerciseId: number;
  exerciseName: string;
  setNumber: number;
  setType: 'working' | 'warmup' | 'dropset' | 'failure';
  weightKg: number | null;
  reps: number | null;
  rir: number | null;
  rpe: number | null;
  isCompleted: boolean;
  isPR: boolean;
  previousWeight: number | null;  // Auto-filled from last log
  previousReps: number | null;
}

interface ActiveExercise {
  exerciseId: number;
  exerciseName: string;
  muscleGroup: string;
  sets: ActiveSet[];
}

interface WorkoutState {
  // Session state
  isActive: boolean;
  routineId: number | null;
  routineName: string | null;
  startedAt: string | null;
  exercises: ActiveExercise[];
  notes: string;
  restTimerSeconds: number;
  restTimerRunning: boolean;
  restTimerEnd: number | null;

  // Actions
  startWorkout: (routineId: number | null, routineName: string | null) => void;
  addExercise: (exercise: { id: number; name: string; muscleGroup: string }) => void;
  removeExercise: (exerciseId: number) => void;
  addSet: (exerciseId: number) => void;
  removeSet: (exerciseId: number, setId: string) => void;
  updateSet: (exerciseId: number, setId: string, field: string, value: any) => void;
  completeSet: (exerciseId: number, setId: string) => void;
  populatePreviousData: (exerciseId: number, previousSets: { weight: number; reps: number }[]) => void;
  setNotes: (notes: string) => void;
  startRestTimer: (seconds: number) => void;
  stopRestTimer: () => void;
  cancelWorkout: () => void;
  finishWorkout: () => {
    routineId: number | null;
    name: string | null;
    startedAt: string;
    completedAt: string;
    durationSeconds: number;
    notes: string;
    sets: any[];
  } | null;

  // Computed
  totalVolume: () => number;
  completedSetsCount: () => number;
  totalSetsCount: () => number;
}

export const useWorkoutStore = create<WorkoutState>()(
  persist(
    (set, get) => ({
      isActive: false,
      routineId: null,
      routineName: null,
      startedAt: null,
      exercises: [],
      notes: '',
      restTimerSeconds: 90,
      restTimerRunning: false,
      restTimerEnd: null,

      startWorkout: (routineId, routineName) => {
        set({
          isActive: true,
          routineId,
          routineName,
          startedAt: new Date().toISOString(),
          exercises: [],
          notes: '',
        });
      },

      addExercise: (exercise) => {
        const exercises = get().exercises;
        if (exercises.find((e) => e.exerciseId === exercise.id)) return;
        set({
          exercises: [...exercises, {
            exerciseId: exercise.id,
            exerciseName: exercise.name,
            muscleGroup: exercise.muscleGroup,
            sets: [{
              id: crypto.randomUUID(),
              exerciseId: exercise.id,
              exerciseName: exercise.name,
              setNumber: 1,
              setType: 'working',
              weightKg: null,
              reps: null,
              rir: null,
              rpe: null,
              isCompleted: false,
              isPR: false,
              previousWeight: null,
              previousReps: null,
            }],
          }],
        });
      },

      removeExercise: (exerciseId) => {
        set({ exercises: get().exercises.filter((e) => e.exerciseId !== exerciseId) });
      },

      addSet: (exerciseId) => {
        set({
          exercises: get().exercises.map((e) => {
            if (e.exerciseId !== exerciseId) return e;
            const lastSet = e.sets[e.sets.length - 1];
            return {
              ...e,
              sets: [...e.sets, {
                id: crypto.randomUUID(),
                exerciseId,
                exerciseName: e.exerciseName,
                setNumber: e.sets.length + 1,
                setType: 'working',
                weightKg: lastSet?.weightKg ?? null,
                reps: lastSet?.reps ?? null,
                rir: null,
                rpe: null,
                isCompleted: false,
                isPR: false,
                previousWeight: null,
                previousReps: null,
              }],
            };
          }),
        });
      },

      removeSet: (exerciseId, setId) => {
        set({
          exercises: get().exercises.map((e) => {
            if (e.exerciseId !== exerciseId) return e;
            return { ...e, sets: e.sets.filter((s) => s.id !== setId) };
          }).filter((e) => e.sets.length > 0),
        });
      },

      updateSet: (exerciseId, setId, field, value) => {
        set({
          exercises: get().exercises.map((e) => {
            if (e.exerciseId !== exerciseId) return e;
            return {
              ...e,
              sets: e.sets.map((s) => s.id === setId ? { ...s, [field]: value } : s),
            };
          }),
        });
      },

      completeSet: (exerciseId, setId) => {
        set({
          exercises: get().exercises.map((e) => {
            if (e.exerciseId !== exerciseId) return e;
            return {
              ...e,
              sets: e.sets.map((s) => s.id === setId ? { ...s, isCompleted: !s.isCompleted } : s),
            };
          }),
        });
      },

      populatePreviousData: (exerciseId, previousSets) => {
        set({
          exercises: get().exercises.map((e) => {
            if (e.exerciseId !== exerciseId) return e;
            return {
              ...e,
              sets: e.sets.map((s, i) => ({
                ...s,
                previousWeight: previousSets[i]?.weight ?? null,
                previousReps: previousSets[i]?.reps ?? null,
                weightKg: s.weightKg ?? previousSets[i]?.weight ?? null,
                reps: s.reps ?? previousSets[i]?.reps ?? null,
              })),
            };
          }),
        });
      },

      setNotes: (notes) => set({ notes }),

      startRestTimer: (seconds) => set({
        restTimerRunning: true,
        restTimerSeconds: seconds,
        restTimerEnd: Date.now() + seconds * 1000,
      }),

      stopRestTimer: () => set({ restTimerRunning: false, restTimerEnd: null }),

      cancelWorkout: () => set({
        isActive: false, routineId: null, routineName: null,
        startedAt: null, exercises: [], notes: '',
      }),

      finishWorkout: () => {
        const state = get();
        if (!state.isActive || !state.startedAt) return null;
        const completedAt = new Date().toISOString();
        const durationSeconds = Math.floor(
          (new Date(completedAt).getTime() - new Date(state.startedAt).getTime()) / 1000
        );
        const payload = {
          routineId: state.routineId,
          name: state.routineName,
          startedAt: state.startedAt,
          completedAt,
          durationSeconds,
          notes: state.notes,
          sets: state.exercises.flatMap((e) =>
            e.sets.filter((s) => s.isCompleted).map((s) => ({
              exercise_id: s.exerciseId,
              set_number: s.setNumber,
              set_type: s.setType,
              weight_kg: s.weightKg ?? 0,
              reps: s.reps ?? 0,
              rir: s.rir,
              rpe: s.rpe,
            }))
          ),
        };
        // Reset state
        set({
          isActive: false, routineId: null, routineName: null,
          startedAt: null, exercises: [], notes: '',
        });
        return payload;
      },

      totalVolume: () => {
        return get().exercises.reduce((total, e) =>
          total + e.sets.reduce((sum, s) =>
            sum + (s.isCompleted && s.setType !== 'warmup' ? (s.weightKg ?? 0) * (s.reps ?? 0) : 0), 0), 0);
      },

      completedSetsCount: () => {
        return get().exercises.reduce((t, e) => t + e.sets.filter((s) => s.isCompleted).length, 0);
      },

      totalSetsCount: () => {
        return get().exercises.reduce((t, e) => t + e.sets.length, 0);
      },
    }),
    {
      name: 'musclo-active-workout',
      // Persist workout to survive page refresh / accidental close
    }
  )
);
```

#### `useAIChatStore.ts`

```typescript
import { create } from 'zustand';

interface ChatMessage {
  id: string;
  role: 'user' | 'assistant';
  content: string;
  timestamp: string;
  isStreaming?: boolean;
}

interface AIChatState {
  isOpen: boolean;
  messages: ChatMessage[];
  isLoading: boolean;
  error: string | null;

  openChat: () => void;
  closeChat: () => void;
  toggleChat: () => void;
  addUserMessage: (content: string) => void;
  addAssistantMessage: (content: string) => void;
  setStreaming: (id: string, content: string) => void;
  finalizeStreaming: (id: string) => void;
  setLoading: (loading: boolean) => void;
  setError: (error: string | null) => void;
  clearMessages: () => void;
}

export const useAIChatStore = create<AIChatState>()((set, get) => ({
  isOpen: false,
  messages: [],
  isLoading: false,
  error: null,

  openChat: () => set({ isOpen: true }),
  closeChat: () => set({ isOpen: false }),
  toggleChat: () => set({ isOpen: !get().isOpen }),

  addUserMessage: (content) => {
    set({
      messages: [...get().messages, {
        id: crypto.randomUUID(), role: 'user', content, timestamp: new Date().toISOString(),
      }],
    });
  },

  addAssistantMessage: (content) => {
    const id = crypto.randomUUID();
    set({
      messages: [...get().messages, {
        id, role: 'assistant', content, timestamp: new Date().toISOString(), isStreaming: true,
      }],
    });
    return id;
  },

  setStreaming: (id, content) => {
    set({
      messages: get().messages.map((m) => m.id === id ? { ...m, content } : m),
    });
  },

  finalizeStreaming: (id) => {
    set({
      messages: get().messages.map((m) => m.id === id ? { ...m, isStreaming: false } : m),
    });
  },

  setLoading: (loading) => set({ isLoading: loading }),
  setError: (error) => set({ error }),
  clearMessages: () => set({ messages: [] }),
}));
```

#### `useThemeStore.ts`

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

interface ThemeState {
  theme: 'dark' | 'light';
  toggleTheme: () => void;
  setTheme: (theme: 'dark' | 'light') => void;
}

export const useThemeStore = create<ThemeState>()(
  persist(
    (set, get) => ({
      theme: 'dark',
      toggleTheme: () => {
        const next = get().theme === 'dark' ? 'light' : 'dark';
        document.documentElement.classList.toggle('dark', next === 'dark');
        set({ theme: next });
      },
      setTheme: (theme) => {
        document.documentElement.classList.toggle('dark', theme === 'dark');
        set({ theme });
      },
    }),
    { name: 'musclo-theme' }
  )
);
```

### 4.2 Axios Instance & Interceptors (`src/api/axios.ts`)

```typescript
import axios, { AxiosError, InternalAxiosRequestConfig } from 'axios';
import { useAuthStore } from '../stores/useAuthStore';

const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:8000';

export const apiClient = axios.create({
  baseURL: `${API_URL}/api`,
  withCredentials: true,
  withXSRFToken: true,
  headers: {
    Accept: 'application/json',
    'Content-Type': 'application/json',
  },
});

// Request interceptor: attach Bearer token
apiClient.interceptors.request.use((config: InternalAxiosRequestConfig) => {
  const token = useAuthStore.getState().token;
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Response interceptor: handle 401 (expired token)
apiClient.interceptors.response.use(
  (response) => response,
  (error: AxiosError) => {
    if (error.response?.status === 401) {
      useAuthStore.getState().reset();
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

// CSRF cookie fetcher (call before login/register)
export const getCsrfCookie = () =>
  axios.get(`${API_URL}/sanctum/csrf-cookie`, { withCredentials: true });
```

### 4.3 React Router v6 Configuration (`src/router/index.tsx`)

```tsx
import { createBrowserRouter, Navigate } from 'react-router-dom';
import AppLayout from '../components/layout/AppLayout';
import ProtectedRoute from '../components/auth/ProtectedRoute';
import LoginPage from '../pages/LoginPage';
import RegisterPage from '../pages/RegisterPage';
import DashboardPage from '../pages/DashboardPage';
import RoutinesPage from '../pages/RoutinesPage';
import WorkoutPage from '../pages/WorkoutPage';
import HistoryPage from '../pages/HistoryPage';
import ExercisesPage from '../pages/ExercisesPage';
import NotFoundPage from '../pages/NotFoundPage';

export const router = createBrowserRouter([
  // Public routes
  { path: '/login', element: <LoginPage /> },
  { path: '/register', element: <RegisterPage /> },

  // Protected routes
  {
    path: '/',
    element: (
      <ProtectedRoute>
        <AppLayout />
      </ProtectedRoute>
    ),
    children: [
      { index: true, element: <Navigate to="/dashboard" replace /> },
      { path: 'dashboard', element: <DashboardPage /> },
      { path: 'routines', element: <RoutinesPage /> },
      { path: 'workout', element: <WorkoutPage /> },
      { path: 'workout/:routineId', element: <WorkoutPage /> },
      { path: 'history', element: <HistoryPage /> },
      { path: 'exercises', element: <ExercisesPage /> },
    ],
  },

  // 404
  { path: '*', element: <NotFoundPage /> },
]);
```

### 4.4 ProtectedRoute Component

```tsx
import { Navigate, useLocation } from 'react-router-dom';
import { useAuthStore } from '../../stores/useAuthStore';

export default function ProtectedRoute({ children }: { children: React.ReactNode }) {
  const isAuthenticated = useAuthStore((s) => s.isAuthenticated);
  const location = useLocation();

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  return <>{children}</>;
}
```

### 4.5 Tailwind Config (`tailwind.config.ts`)

```typescript
import type { Config } from 'tailwindcss';

export default {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  darkMode: 'class',
  theme: {
    extend: {
      colors: {
        bg: {
          primary: 'var(--bg-primary)',
          secondary: 'var(--bg-secondary)',
          tertiary: 'var(--bg-tertiary)',
          elevated: 'var(--bg-elevated)',
        },
        border: {
          subtle: 'var(--border-subtle)',
          focus: 'var(--border-focus)',
        },
        text: {
          primary: 'var(--text-primary)',
          secondary: 'var(--text-secondary)',
          muted: 'var(--text-muted)',
        },
        accent: {
          primary: 'var(--accent-primary)',
          'primary-hover': 'var(--accent-primary-hover)',
          secondary: 'var(--accent-secondary)',
        },
        success: 'var(--success)',
        warning: 'var(--warning)',
        danger: 'var(--danger)',
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', '-apple-system', 'sans-serif'],
      },
      borderRadius: {
        sm: '6px',
        md: '10px',
        lg: '16px',
        xl: '24px',
      },
      animation: {
        shimmer: 'shimmer 1.5s infinite linear',
      },
      keyframes: {
        shimmer: {
          '0%': { backgroundPosition: '200% 0' },
          '100%': { backgroundPosition: '-200% 0' },
        },
      },
    },
  },
  plugins: [],
} satisfies Config;
```

### 4.6 CSS Custom Properties (`src/index.css`)

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');

@layer base {
  :root {
    --bg-primary: #fafafa;
    --bg-secondary: #ffffff;
    --bg-tertiary: #f0f0f3;
    --bg-elevated: #e8e8ec;
    --border-subtle: #d4d4dc;
    --border-focus: #7028e4;
    --text-primary: #111118;
    --text-secondary: #6b6b78;
    --text-muted: #9b9ba8;
    --accent-primary: #7028e4;
    --accent-primary-hover: #6020d0;
    --accent-secondary: #1fa8a8;
    --success: #2db870;
    --warning: #f0a820;
    --danger: #d94040;
  }

  .dark {
    --bg-primary: #0e0e11;
    --bg-secondary: #17171c;
    --bg-tertiary: #202027;
    --bg-elevated: #2a2a33;
    --border-subtle: #303038;
    --border-focus: #8b3cf7;
    --text-primary: #f5f5f5;
    --text-secondary: #9393a0;
    --text-muted: #62626c;
    --accent-primary: #8b3cf7;
    --accent-primary-hover: #7c2cf0;
    --accent-secondary: #26c6c6;
    --success: #2db870;
    --warning: #f0a820;
    --danger: #d94040;
  }

  body {
    @apply bg-bg-primary text-text-primary font-sans antialiased;
  }

  * {
    @apply border-border-subtle;
  }
}

/* Glassmorphism utility */
@layer utilities {
  .glass {
    @apply bg-bg-secondary/60 backdrop-blur-xl border border-border-subtle;
  }
}
```

---

## 5. The AI Agent Execution Sequence

Execute these steps in **strict order**. Each step must complete before the next begins.

### Phase 0: Environment Prerequisite Checks

```bash
# Verify prerequisites — abort if any are missing
node -v        # Expect v18+ or v20+
npm -v         # Expect 9+
php -v         # Expect 8.2+
composer -V    # Expect 2.x
mysql --version # Expect 8.x
```

### Phase 1: Backend Initialization

```bash
# Step 1.1: Create Laravel project
composer create-project laravel/laravel musclo-api
cd musclo-api

# Step 1.2: Install Sanctum (included by default in Laravel 12, but ensure)
php artisan install:api

# Step 1.3: Configure .env
# Set DB_DATABASE=musclo_db, DB_USERNAME, DB_PASSWORD
# Add FRONTEND_URL=http://localhost:5173
# Add SANCTUM_STATEFUL_DOMAINS=localhost:5173
# Add SESSION_DOMAIN=localhost
# Add DEEPSEEK_API_KEY=your_key_here
# Add DEEPSEEK_MODEL=deepseek-reasoner

# Step 1.4: Create MySQL database
mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS musclo_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"

# Step 1.5: Generate all Models, Migrations, Controllers, Requests, Resources (see Section 3.1)
php artisan make:model Routine -mf
php artisan make:model Exercise -mf
php artisan make:model RoutineExercise -m
php artisan make:model WorkoutLog -mf
php artisan make:model SetData -mf
php artisan make:controller Auth/RegisterController
php artisan make:controller Auth/LoginController
php artisan make:controller Auth/LogoutController
php artisan make:controller RoutineController --api
php artisan make:controller ExerciseController --api
php artisan make:controller WorkoutLogController --api
php artisan make:controller AICoachController
php artisan make:request Auth/RegisterRequest
php artisan make:request Auth/LoginRequest
php artisan make:request StoreRoutineRequest
php artisan make:request UpdateRoutineRequest
php artisan make:request StoreWorkoutLogRequest
php artisan make:request AskAIRequest
php artisan make:resource UserResource
php artisan make:resource RoutineResource
php artisan make:resource ExerciseResource
php artisan make:resource WorkoutLogResource
php artisan make:resource SetDataResource
php artisan make:resource AIResponseResource
php artisan make:seeder ExerciseSeeder
php artisan make:middleware ForceJsonResponse

# Step 1.6: Replace migration file contents with code from Section 3.2
# Step 1.7: Replace model file contents with code from Section 3.3
# Step 1.8: Create app/Services/DeepSeekService.php with code from Section 3.6
# Step 1.8b: Create app/Services/ExerciseDBService.php with code from Section 3.13
# Step 1.9: Create config/deepseek.php with code from Section 3.7
# Step 1.9b: Create app/Console/Commands/ImportExercisesCommand.php with code from Section 3.14
# Step 1.10: Replace controller contents with code from Sections 3.8, 3.10
# Step 1.11: Replace form request contents with code from Section 3.9
# Step 1.11b: Create ExerciseResource with code from Section 3.15
# Step 1.12: Set up routes/api.php with code from Section 3.5
# Step 1.13: Register ForceJsonResponse middleware per Section 3.11
# Step 1.14: Update config/cors.php per Section 3.12

# Step 1.15: Run migrations
php artisan migrate

# Step 1.16: Import exercises from ExerciseDB API (replaces static seeder)
# This fetches 1500+ exercises with animated GIFs from https://www.exercisedb.dev
php artisan exercises:import

# Step 1.17: Verify backend
php artisan serve
# Test: curl http://localhost:8000/api/exercises should return 401 (auth required)
```

### Phase 2: Frontend Initialization

```bash
# Step 2.1: Create Vite React project
npm create vite@latest musclo-spa -- --template react-ts
cd musclo-spa
npm install

# Step 2.2: Install dependencies
npm install axios zustand react-router-dom framer-motion recharts
npm install @radix-ui/react-dialog @radix-ui/react-tabs @radix-ui/react-toast @radix-ui/react-tooltip @radix-ui/react-select @radix-ui/react-alert-dialog
npm install clsx tailwind-merge lucide-react
npm install -D tailwindcss postcss autoprefixer @types/node

# Step 2.3: Initialize Tailwind
npx tailwindcss init -p

# Step 2.4: Replace tailwind.config.ts with code from Section 4.5
# Step 2.5: Replace src/index.css with code from Section 4.6
# Step 2.6: Create .env file
echo "VITE_API_URL=http://localhost:8000" > .env

# Step 2.7: Create directory structure
mkdir -p src/{api,stores,hooks,lib,components/{ui,layout,auth,dashboard,routines,workout,history,exercises,ai},pages,router}

# Step 2.8: Create src/lib/utils.ts
# Implement cn() using clsx + tailwind-merge, formatWeight(), formatDate(), formatDuration()

# Step 2.9: Create src/lib/motion.ts with code from Section 1.5
# Step 2.10: Create src/lib/constants.ts with MUSCLE_GROUPS enum, SET_TYPES array, and Exercise interface:
#   interface Exercise { id, exercisedbId, name, muscleGroup, secondaryMuscles, equipment, bodyPart, category, instructions, gifUrl, isCustom }

# Step 2.11: Create src/api/axios.ts with code from Section 4.2
# Step 2.12: Create all Zustand stores with code from Section 4.1

# Step 2.13: Create src/router/index.tsx with code from Section 4.3
# Step 2.14: Update src/App.tsx to use RouterProvider
# Step 2.15: Update src/main.tsx to render App with theme initialization
```

### Phase 3: Build UI Components

Build in this order (dependencies first):

```
Step 3.1:  src/components/ui/Button.tsx
Step 3.2:  src/components/ui/Input.tsx
Step 3.3:  src/components/ui/Card.tsx
Step 3.4:  src/components/ui/Badge.tsx
Step 3.5:  src/components/ui/Avatar.tsx
Step 3.6:  src/components/ui/Skeleton.tsx
Step 3.7:  src/components/ui/Modal.tsx (Radix Dialog + Framer Motion)
Step 3.8:  src/components/ui/Sheet.tsx
Step 3.9:  src/components/ui/Toast.tsx (Radix Toast + animation)
Step 3.10: src/components/ui/Tabs.tsx (Radix Tabs + layoutId)
Step 3.11: src/components/ui/Select.tsx (Radix Select)
Step 3.12: src/components/ui/Tooltip.tsx
Step 3.13: src/components/ui/NumberTicker.tsx
Step 3.14: src/components/ui/EmptyState.tsx
Step 3.15: src/components/ui/ConfirmDialog.tsx
```

### Phase 4: Build Layout & Auth

```
Step 4.1:  src/components/layout/Sidebar.tsx — Desktop nav with Musclo logo (`logo.png`), nav links, user avatar, theme toggle
Step 4.2:  src/components/layout/BottomNav.tsx — Mobile nav with 5 icons (Dashboard, Routines, Workout, History, Exercises)
Step 4.3:  src/components/layout/TopBar.tsx — Mobile header with page title, AI chat button
Step 4.4:  src/components/layout/PageTransition.tsx — AnimatePresence wrapper using MOTION.pageEnter
Step 4.5:  src/components/layout/AppLayout.tsx — Combines Sidebar, TopBar, BottomNav, Outlet with PageTransition
Step 4.6:  src/components/auth/LoginForm.tsx — Email/password form, getCsrfCookie() before submit
Step 4.7:  src/components/auth/RegisterForm.tsx — Name/email/password/confirm form
Step 4.8:  src/components/auth/ProtectedRoute.tsx (Section 4.4)
Step 4.9:  src/pages/LoginPage.tsx — Full-page with gradient background, centered LoginForm
Step 4.10: src/pages/RegisterPage.tsx — Matching design
```

### Phase 5: Build Feature Pages

```
Step 5.1:  Dashboard — StatsCard (glassmorphic), RecentWorkouts, WeeklyVolumeChart (Recharts AreaChart), MuscleHeatmap (RadarChart)
Step 5.2:  Routines — RoutineCard with hoverLift, RoutineBuilder modal, ExercisePicker with search + GIF thumbnails (48×48, lazy-loaded) from ExerciseDB
Step 5.3:  Active Workout — Full-screen logger, ExerciseBlock (with small 32×32 GIF icon next to exercise name), SetRow (weight/reps/rir inputs), RestTimer (circular progress), live NumberTicker for volume
Step 5.4:  Workout Summary — Post-workout modal with animated stats, PR celebrations with PRBanner
Step 5.5:  History — WorkoutHistoryCard list (with tiny GIF avatars for each exercise), ProgressCharts (Recharts) with exercise-specific plots
Step 5.6:  Exercises — Searchable/filterable exercise library with animated GIF previews from ExerciseDB (play on hover desktop, always visible mobile). ExerciseDetailModal shows full-width animated GIF hero with name, muscles, equipment, and step-by-step instructions below.
Step 5.7:  AI Chat — AIChatFAB (floating button), AIChatModal with StreamingText, context auto-injection from useWorkoutStore
```

### Phase 6: Edge Case Handling

| Scenario | Implementation |
|----------|---------------|
| **DeepSeek API timeout** | `DeepSeekService` has 30s timeout + 2 retries. On failure, frontend shows: "AI Coach is temporarily unavailable. Try again shortly." with retry button. Chat stays open. |
| **Optimistic UI updates** | When completing a set, update `useWorkoutStore` immediately. On API save failure, show toast error but keep local state (user can retry). |
| **Active workout persistence** | `useWorkoutStore` uses Zustand `persist` middleware → survives page refresh, browser crash, accidental close. Show "Resume Workout" banner on Dashboard if `isActive === true`. |
| **401 Unauthorized** | Axios response interceptor clears auth state and redirects to `/login`. |
| **Empty states** | Every list view (routines, history, exercises) shows `EmptyState` component with illustration and CTA button when data is empty. |
| **Form validation** | Frontend validates all inputs before submit. Laravel `FormRequest` classes provide server-side validation. Display errors inline under inputs with red text + shake animation. |
| **Responsive design** | Every component must work on mobile (< 768px) and desktop (≥ 1024px). Sidebar hidden on mobile, BottomNav hidden on desktop. Workout logger is full-screen on mobile, side panel on desktop. |

### Phase 7: Final Verification

```bash
# Backend
cd musclo-api
php artisan test    # Ensure no errors (write Feature tests for auth, workout CRUD)
php artisan serve   # Start on port 8000

# Frontend
cd musclo-spa
npm run build       # Ensure zero TypeScript errors, zero build warnings
npm run dev         # Start on port 5173

# Manual Smoke Test Checklist:
# [ ] Register new user → redirects to dashboard
# [ ] Login → redirects to dashboard
# [ ] Dashboard shows empty state
# [ ] Create routine with 3 exercises
# [ ] Start workout from routine → logger opens with exercises pre-loaded
# [ ] Previous set data auto-fills from last workout
# [ ] Complete sets → volume counter animates
# [ ] Finish workout → summary shows with PR banners if applicable
# [ ] View history → new workout appears
# [ ] Progress charts render with real data
# [ ] AI Chat → send message → receive streaming response
# [ ] Theme toggle (dark/light) works
# [ ] Mobile responsive layout works
# [ ] Page refresh during workout → workout state persists
# [ ] Logout → redirects to login, clears state


---

> **End of Master Blueprint.** Every section above is designed to be executed sequentially by an AI coding agent with zero ambiguity. The design system, database schema, API contracts, state management, and edge cases are fully specified.
