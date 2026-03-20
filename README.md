# NeuroFocus – AI-Based Dopamine Reallocation System

> An offline Android application built with **Kotlin + Jetpack Compose** that detects addictive smartphone usage patterns, blocks distracting apps behind an **XP paywall**, and redirects users toward productive goals using a **rule-based behavioral scoring engine** (Explainable AI).

---

## Table of Contents

- [Problem Statement](#problem-statement)
- [How It Works](#how-it-works)
- [AI Methodology](#ai-methodology)
- [App Blocking – Earn Your Time](#app-blocking--earn-your-time)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Features](#features)
- [Screenshots & Flow](#screenshots--flow)
- [Setup & Running](#setup--running)
- [Permissions Required](#permissions-required)
- [Project Structure](#project-structure)
- [Configuration](#configuration)

---

## Problem Statement

Digital addiction is a growing concern, especially among students. Excessive social media usage, late-night phone use, and rapid app switching are indicators of **dopamine-seeking behavior** that leads to reduced focus, disrupted sleep, and decreased productivity.

**NeuroFocus** addresses this by:

1. **Monitoring** real-time smartphone usage patterns via Android's `UsageStatsManager`
2. **Scoring** behavior using a weighted rule-based AI engine (Explainable AI)
3. **Blocking** distracting apps (social media + games) behind an XP paywall
4. **Intervening** when addictive patterns are detected with contextual suggestions
5. **Redirecting** users toward productive activities via gamified goals and focus sessions

---

## How It Works

```
                    ┌─────────────────────────────┐
                    │     User opens Instagram     │
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │   AccessibilityService       │
                    │   detects foreground app      │
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │   Is app in blocked list?     │
                    │   Is session bypassed?        │
                    └──────┬──────────────┬───────┘
                           │              │
                      NOT bypassed    Bypassed
                           │              │
              ┌────────────▼────────┐     │ ──► App opens normally
              │  InterceptActivity  │
              │  "Pay 50 XP to Open"│
              └────────┬──────┬─────┘
                       │      │
                  Has XP   No XP
                       │      │
          ┌────────────▼──┐   └──► "Earn more XP via Focus Sessions"
          │ Deduct 50 XP  │
          │ Set bypass flag│
          │ Close overlay  │
          └───────┬───────┘
                  │
     User uses app freely until they switch away
     Bypass clears → must pay again next time
```

---

## AI Methodology

### Why This Qualifies as AI

This system implements a **Rule-Based Expert System**, a well-established branch of Artificial Intelligence dating back to the 1970s (MYCIN, DENDRAL). It is a form of **"Good Old-Fashioned AI" (GOFAI)** that uses human-codified knowledge to make autonomous decisions.

### Dopamine Scoring Formula

The `DopamineScoreEngine` computes a weighted behavioral score from 4 real-time signals:

```
dopamineScore = (socialMediaOpenCount × socialMediaWeight)
              + (lateNightUsageMinutes × lateNightWeight)
              + (appSwitchFrequency × appSwitchWeight)
              + (totalScreenTimeMinutes × screenTimeWeight)
```

| Signal | Default Weight | Rationale |
|--------|:---:|-----------|
| Social Media Opens | **2.0** | Each open = dopamine-seeking "check" behavior |
| Late-Night Usage (min) | **0.5** | Correlates with compulsive use & sleep disruption |
| App Switching Frequency | **1.5** | Rapid switching indicates shortened attention span |
| Total Screen Time (min) | **0.2** | Broad indicator; lower weight since some usage is productive |

### AI Characteristics

| Property | Implementation |
|----------|---------------|
| **Pattern Recognition** | Analyzes 4 behavioral signals simultaneously to identify addictive usage |
| **Autonomous Decision Making** | Triggers interventions when combined score ≥ threshold (default: 50) |
| **Explainability (XAI)** | Every score is broken down into component contributions with exact weights |
| **Configurable Knowledge Base** | Weights are tunable via the Settings screen without code changes |
| **Anomaly Detection** | 7-day moving average + standard deviation flags unusual spikes (score > μ + 1σ) |

### Anomaly Detection Algorithm

```kotlin
val mean = last7DayScores.average()
val stdDev = sqrt(scores.map { (it - mean).pow(2) }.average())
val isAnomaly = todayScore > (mean + stdDev)  // Beyond 1 standard deviation
```

---

## App Blocking – Earn Your Time

NeuroFocus implements an **XP-gated app locking system** using Android's `AccessibilityService`.

### How Session Bypass Works

1. **Detection**: The `AppInterceptorService` listens for `TYPE_WINDOW_STATE_CHANGED` events
2. **Interception**: When a blocked app comes to the foreground, `InterceptActivity` is launched
3. **Payment**: User pays **50 XP** to unlock the app for the current session
4. **Bypass**: An in-memory `@Volatile` flag stores the currently bypassed package
5. **Clearing**: The bypass is cleared the moment the user switches to any other app or the home screen
6. **Re-entry**: Opening the same app again requires another 50 XP payment

### Why In-Memory (Not Database)

- Bypass is inherently **transient** — only valid while the user stays in the app
- No persistence needed; service restart = bypass cleared (correct behavior)
- Avoids async DB queries on every accessibility event (hundreds per minute)

### Blocked Apps (36 Total)

<details>
<summary><b>Social Media (12 apps)</b></summary>

| App | Package |
|-----|---------|
| Instagram | `com.instagram.android` |
| Facebook | `com.facebook.katana` |
| Messenger | `com.facebook.orca` |
| Twitter/X | `com.twitter.android` |
| Snapchat | `com.snapchat.android` |
| TikTok | `com.zhiliaoapp.musically` |
| Reddit | `com.reddit.frontpage` |
| YouTube | `com.google.android.youtube` |
| Pinterest | `com.pinterest` |
| Discord | `com.discord` |
| Tumblr | `com.tumblr` |
| Twitter Lite | `com.twitter.android.lite` |

</details>

<details>
<summary><b>Games (24 apps)</b></summary>

| App | Package |
|-----|---------|
| Free Fire | `com.dts.freefireth` |
| Free Fire MAX | `com.dts.freefiremax` |
| Stick Cricket 2 | `com.sticksports.stickcricket2` |
| Stick Cricket Super | `com.sticksports.stickcricketsuper` |
| Dream League Soccer | `com.firsttouchgames.dls7` |
| Score! Match | `com.firsttouchgames.smp` |
| BGMI | `com.pubg.imobile` |
| PUBG Mobile | `com.tencent.ig` |
| Clash of Clans | `com.supercell.clashofclans` |
| Clash Royale | `com.supercell.clashroyale` |
| Subway Surfers | `com.kiloo.subwaysurf` |
| Candy Crush Saga | `com.king.candycrushsaga` |
| 8 Ball Pool | `com.miniclip.eightballpool` |
| Minecraft | `com.mojang.minecraftpe` |
| Call of Duty Mobile | `com.activision.callofduty.shooter` |
| Archery Battle 3D | `com.grey.archer` |
| Ludo King | `com.ludo.king` |
| Among Us | `com.innersloth.spacemafia` |
| Hill Climb Racing | `com.fingersoft.hillclimb` |
| Temple Run 2 | `com.imangi.templerun2` |
| Asphalt 9 | `com.gameloft.android.ANMP.GlsAsphalt9` |
| Genshin Impact | `com.miHoYo.GenshinImpact` |
| Roblox | `com.roblox.client` |
| Stumble Guys | `com.scopely.stumbleguys` |

</details>

---

## Architecture

**Clean Architecture (MVVM)** with strict layer separation:

```
┌──────────────────────────────────────────────────────────┐
│                   Presentation Layer                      │
│  Compose Screens → ViewModels → StateFlow                │
│  InterceptActivity, BlockActivity, AppLockedActivity     │
├──────────────────────────────────────────────────────────┤
│                     Service Layer                         │
│  AppInterceptorService (AccessibilityService)            │
│  UsageTrackingWorker (WorkManager)                       │
├──────────────────────────────────────────────────────────┤
│                     Domain Layer                          │
│  Use Cases (5) → DopamineScoreEngine → AnomalyDetector   │
│  Repository Interfaces (6)                               │
├──────────────────────────────────────────────────────────┤
│                      Data Layer                           │
│  Room DB (7 entities, 7 DAOs) → Repository Impls (6)     │
│  UsageStatsManager Integration                           │
├──────────────────────────────────────────────────────────┤
│                 Dependency Injection                      │
│  Hilt Modules: DatabaseModule, RepositoryModule          │
└──────────────────────────────────────────────────────────┘
```

### Key Components

| Component | File | Purpose |
|-----------|------|---------|
| **DopamineScoreEngine** | `domain/engine/` | Core AI – computes weighted score from 4 behavioral signals |
| **AnomalyDetector** | `domain/engine/` | Statistical anomaly detection (7-day moving average + σ) |
| **AppInterceptorService** | `service/` | AccessibilityService – detects blocked app launches, manages session bypass |
| **InterceptActivity** | `presentation/intercept/` | "Pay 50 XP to Open" full-screen overlay |
| **UsageStatsTracker** | `data/usage/` | Queries Android's UsageStatsManager for real-time data |
| **UsageTrackingWorker** | `data/usage/` | Background periodic data collection via WorkManager |
| **NeuroFocusDatabase** | `data/local/` | Room database with 7 entities (v2, destructive migration) |

---

## Tech Stack

| Technology | Purpose |
|------------|---------|
| **Kotlin** | Programming language |
| **Jetpack Compose** | Declarative UI toolkit |
| **Material 3** | Design system with dark/light theme |
| **Room** | Local SQLite database (7 entities) |
| **Hilt** | Dependency injection framework |
| **WorkManager** | Background periodic usage data collection |
| **UsageStatsManager** | Android system API for app usage statistics |
| **AccessibilityService** | Real-time foreground app detection for blocking |
| **Compose Canvas** | Custom chart drawing (score gauge, bar charts, line charts) |
| **StateFlow** | Reactive state management in ViewModels |
| **Navigation Compose** | Bottom navigation + nested routes |

---

## Features

### 🧠 Dopamine Score Dashboard
- Animated score gauge with color-coded status (Green → Yellow → Orange → Red)
- Real-time score breakdown showing each factor's contribution
- Total screen time and social media open count display

### 🚫 XP-Gated App Blocking
- 36 blocked apps (social media + games) intercepted via AccessibilityService
- "Pay 50 XP to Open" overlay with XP balance display
- In-memory session bypass — app stays unlocked until user switches away
- Insufficient XP warning with guidance to earn more

### 📊 7-Day Analytics
- Score trend line chart (custom Canvas)
- Daily screen time bar chart
- Intervention frequency bar chart
- Summary statistics and averages

### 🎯 Goals & Gamification
- Create custom goals (e.g., "Practice LeetCode", "Read for 30 min")
- Focus timer with animated circular countdown ring
- XP rewards on session completion (base + bonus for longer sessions)
- Level system: Level = floor(totalXp / 500) + 1
- Streak tracking with longest streak record

### ⚙️ Settings & Configuration
- Adjustable scoring weights via sliders
- Configurable intervention threshold
- Sample data generator for demos
- About section with app info

### 🔍 Anomaly Detection
- 7-day moving average comparison
- Standard deviation-based spike detection
- Warning banner on the dashboard when anomaly detected

### 🔄 Data Architecture
- Init-only data fetch (loads once on app restart — no battery drain)
- Fully offline — all data stored locally via Room
- Periodic background collection via WorkManager

---

## Screenshots & Flow

### User Flow

```
App Launch
    │
    ├── Dashboard ──── Score Gauge + Breakdown + Anomaly Warning
    │
    ├── Analytics ──── 7-Day Charts (Score, Screen Time, Interventions)
    │
    ├── Goals ──────── Goal Cards → Start Focus Session → Earn XP
    │                                    │
    │                              FocusTimerScreen
    │                              (Countdown + XP reward)
    │
    └── Settings ───── Weight Sliders + Threshold + Sample Data

Background: AppInterceptorService
    │
    └── User opens blocked app → InterceptActivity
                                      │
                                 "Pay 50 XP"
                                      │
                              ┌───────┴───────┐
                          Success          Not enough
                              │                │
                        App unlocked     "Earn more XP"
                     (session bypass)
```

---

## Setup & Running

### Prerequisites
- **Android Studio** Hedgehog (2023.1.1) or newer
- **JDK 17** or newer
- Android device/emulator running **API 26+** (Android 8.0+)

### Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/Bhavesh1506/NeuroFocus.git
   cd NeuroFocus
   ```

2. **Open in Android Studio** → Sync Gradle dependencies

3. **Build & Run** on emulator or physical device

4. **Grant Permissions** (prompted on first launch):
   - **Usage Stats Access** – Required for tracking screen time

5. **Enable Accessibility Service** (for app blocking):
   - Go to **Settings → Accessibility → NeuroFocus → Enable**

6. **Generate Demo Data**:
   - Open the **Settings** tab → Tap **Generate Sample Data**
   - This creates usage records, scores, and XP for demonstration

---

## Permissions Required

| Permission | Purpose | Type |
|------------|---------|------|
| `PACKAGE_USAGE_STATS` | Query app usage data from UsageStatsManager | Special (manual grant) |
| `BIND_ACCESSIBILITY_SERVICE` | Detect foreground app for blocking | Special (manual enable) |
| `SYSTEM_ALERT_WINDOW` | Fallback overlay permission | Special |
| `RECEIVE_BOOT_COMPLETED` | Restart WorkManager after device reboot | Normal |
| `FOREGROUND_SERVICE` | Focus timer foreground service | Normal |
| `POST_NOTIFICATIONS` | Notifications on Android 13+ | Runtime |

---

## Project Structure

```
app/src/main/java/com/neurofocus/app/
├── data/
│   ├── local/
│   │   ├── dao/               # Room DAOs (7)
│   │   │   ├── UsageDao.kt
│   │   │   ├── DopamineScoreDao.kt
│   │   │   ├── InterventionDao.kt
│   │   │   ├── GoalDao.kt
│   │   │   ├── FocusSessionDao.kt
│   │   │   ├── UserProgressDao.kt
│   │   │   └── AppUnlockDao.kt
│   │   ├── entity/            # Room entities (7)
│   │   │   ├── UsageRecord.kt
│   │   │   ├── DopamineScoreRecord.kt
│   │   │   ├── InterventionRecord.kt
│   │   │   ├── Goal.kt
│   │   │   ├── FocusSession.kt
│   │   │   ├── UserProgress.kt
│   │   │   └── AppUnlock.kt
│   │   └── NeuroFocusDatabase.kt
│   ├── repository/            # Repository implementations (6)
│   └── usage/                 # UsageStatsTracker, WorkManager worker
├── di/                        # Hilt modules
│   ├── DatabaseModule.kt      # Room DB + DAOs + WorkManager
│   └── RepositoryModule.kt    # Interface → Impl bindings
├── domain/
│   ├── engine/                # AI engines
│   │   ├── DopamineScoreEngine.kt
│   │   └── AnomalyDetector.kt
│   ├── model/                 # Data models
│   │   ├── ScoreResult.kt
│   │   └── ScoringWeights.kt
│   ├── repository/            # Repository interfaces (6)
│   └── usecase/               # Use cases (5)
│       ├── CalculateDopamineScoreUseCase.kt
│       ├── GetAnalyticsDataUseCase.kt
│       ├── ManageGoalsUseCase.kt
│       ├── RecordFocusSessionUseCase.kt
│       └── TriggerInterventionUseCase.kt
├── service/
│   └── AppInterceptorService.kt  # AccessibilityService (app blocking)
├── presentation/
│   ├── analytics/             # AnalyticsScreen + ViewModel
│   ├── applock/               # AppLockedActivity + ViewModel (XP timer)
│   ├── block/                 # BlockActivity (daily limit)
│   ├── dashboard/             # DashboardScreen + ViewModel
│   ├── goals/                 # GoalsScreen, FocusTimerScreen + ViewModel
│   ├── intercept/             # InterceptActivity + ViewModel (XP session)
│   ├── intervention/          # InterventionScreen
│   ├── navigation/            # NavGraph + Bottom Navigation
│   ├── settings/              # SettingsScreen + ViewModel
│   └── theme/                 # Color.kt, Theme.kt (Material 3)
├── MainActivity.kt            # Entry point (@AndroidEntryPoint)
└── NeuroFocusApp.kt           # Application class (@HiltAndroidApp)

app/src/main/res/
├── xml/
│   └── accessibility_service_config.xml  # AccessibilityService metadata
└── values/
    └── strings.xml            # App strings + service description
```

---

## Configuration

### Modifying Blocked Apps

Edit the `blockedPackages` set in `AppInterceptorService.kt`:

```kotlin
private val blockedPackages = setOf(
    "com.instagram.android",
    "com.your.custom.app",  // Add any package here
    // ...
)
```

### Adjusting XP Cost

Edit `InterceptViewModel.kt`:

```kotlin
data class InterceptState(
    val xpCost: Int = 50,  // Change the XP cost per unlock
    // ...
)
```

### Tuning the Scoring Engine

Use the **Settings screen** in the app to adjust weights and threshold in real-time, or modify defaults in `ScoringWeights.kt`:

```kotlin
data class ScoringWeights(
    val socialMediaWeight: Float = 2.0f,
    val lateNightWeight: Float = 0.5f,
    val appSwitchWeight: Float = 1.5f,
    val screenTimeWeight: Float = 0.2f,
    val interventionThreshold: Float = 50f
)
```

---

## License

This project is developed for educational purposes as part of a college project on AI-based digital wellbeing intervention systems.

---

*Built with ❤️ using Kotlin, Jetpack Compose, and Clean Architecture*
