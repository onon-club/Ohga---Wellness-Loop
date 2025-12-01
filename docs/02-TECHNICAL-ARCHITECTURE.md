# 🏗️ Technical Architecture Documentation

## System Overview

Wellness Loop follows a **clean architecture** pattern with clear separation of concerns across frontend, backend, and infrastructure layers. The system is designed for **offline-first** operation with intelligent sync capabilities.

---

## 📱 Frontend Architecture (Flutter)

### Technology Stack

| Component | Technology | Version |
|-----------|------------|---------|
| Framework | Flutter | 3.16+ |
| Language | Dart | 3.8+ |
| State Management | Riverpod | 3.x |
| Local Database | Drift (SQLite) | 2.x |
| Routing | GoRouter | 14.x |
| HTTP Client | Dio | 5.x |
| Secure Storage | flutter_secure_storage | 9.x |
| Charts | fl_chart | 0.68+ |
| Animations | flutter_animate | 4.x |
| Error Tracking | Sentry | 8.x |

### Directory Structure

```
lib/
├── app/                    # Application configuration
│   ├── config/             # Environment, constants
│   └── theme/              # Design system, colors, typography
│
├── core/                   # Shared infrastructure
│   ├── auth/               # Authentication providers
│   ├── network/            # API client, interceptors
│   ├── services/           # Cross-cutting services
│   └── utils/              # Utilities, extensions
│
├── features/               # Feature modules
│   ├── ai_coach/           # AI chat and recommendations
│   │   ├── models/
│   │   ├── pages/
│   │   ├── providers/
│   │   ├── services/
│   │   └── widgets/
│   │
│   ├── exercises/          # Exercise library (476+ exercises)
│   ├── gamification/       # XP, levels, achievements
│   ├── goals/              # Goal setting and tracking
│   ├── home/               # Dashboard and navigation
│   ├── nutrition/          # Food logging and analysis
│   ├── profile/            # User profile and settings
│   ├── streaks/            # Streak tracking
│   └── workouts/           # Workout creation and tracking
│
├── routes/                 # GoRouter configuration
├── shared/                 # Shared widgets and utilities
└── main.dart               # Application entry point
```

### State Management (Riverpod)

**24 Providers** organized by feature:

```dart
// Authentication
final authProvider = NotifierProvider<AuthNotifier, AuthState>(...);

// Profile
final profileProvider = NotifierProvider<ProfileNotifier, ProfileState>(...);
final profileStatsProvider = FutureProvider<ProfileStats>(...);

// Workouts
final workoutSessionProvider = NotifierProvider<WorkoutSessionNotifier, WorkoutSessionState>(...);
final workoutHistoryProvider = NotifierProvider<WorkoutHistoryNotifier, List<Workout>>(...);

// Exercises
final exerciseProvider = NotifierProvider<ExerciseNotifier, ExerciseState>(...);
final exerciseSearchProvider = StateNotifierProvider<ExerciseSearchNotifier, SearchState>(...);
final exerciseFavoritesProvider = NotifierProvider<FavoritesNotifier, List<Exercise>>(...);

// Nutrition
final nutritionProvider = NotifierProvider<NutritionNotifier, NutritionState>(...);
final nutritionDashboardProvider = NotifierProvider<DashboardNotifier, DashboardState>(...);

// Gamification
final achievementProvider = NotifierProvider<AchievementNotifier, List<Achievement>>(...);
final streakProvider = NotifierProvider<StreakNotifier, StreakState>(...);
final goalProvider = NotifierProvider<GoalNotifier, List<Goal>>(...);

// AI Coach
final chatProvider = NotifierProvider<ChatNotifier, ChatState>(...);

// Infrastructure
final themeProvider = NotifierProvider<ThemeNotifier, ThemeState>(...);
final connectivityProvider = StreamProvider<ConnectivityStatus>(...);
```

### Local Database (Drift)

**10 Tables** with corresponding DAOs:

```dart
// Database tables
@DataClassName('ProfileEntity')
class Profiles extends Table {
  TextColumn get id => text()();
  TextColumn get email => text().nullable()();
  TextColumn get displayName => text().nullable()();
  IntColumn get currentLevel => integer().withDefault(const Constant(1))();
  IntColumn get totalXp => integer().withDefault(const Constant(0))();
  TextColumn get preferencesJson => text().nullable()();
  DateTimeColumn get lastSyncedAt => dateTime().nullable()();
}

// Similar tables for:
// - WorkoutSessions
// - Exercises
// - NutritionEntries
// - Goals
// - StreakEntries
// - Quests
// - ChatMessages
// - SyncQueue
// - ExerciseUserData
```

### Offline-First Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      User Action                             │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Service Layer                             │
│  1. Save to local Drift database (immediate)                │
│  2. Update Riverpod state (UI refresh)                      │
│  3. Queue for backend sync (SyncQueue table)                │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    Sync Service                              │
│  - Monitors connectivity status                             │
│  - Processes sync queue when online                         │
│  - Handles conflict resolution                              │
│  - Retry with exponential backoff                           │
└─────────────────────────────────────────────────────────────┘
```

---

## ⚙️ Backend Architecture (Go)

### Technology Stack

| Component | Technology | Version |
|-----------|------------|---------|
| Runtime | Go | 1.24+ |
| Web Framework | Fiber | v2 |
| Database Driver | pgx | v5 |
| Cache | go-redis | v9 |
| Message Queue | NATS | 2.x |
| Auth | WorkOS SDK | 5.x |
| Logging | Zap | 1.x |
| Metrics | Prometheus | 0.5+ |
| Tracing | OpenTelemetry | 1.x |

### Directory Structure

```
backend-go/
├── cmd/
│   ├── api/                # Main API server entry point
│   ├── data-generator/     # Test data generation tool
│   └── delete-workos-users/# User cleanup utility
│
├── internal/
│   ├── auth/               # Authentication middleware
│   ├── cache/              # Redis caching layer
│   ├── clients/            # External API clients (Perplexity, WorkOS)
│   ├── config/             # Configuration management
│   ├── database/           # PostgreSQL connection
│   ├── handlers/           # HTTP request handlers (14 handlers)
│   ├── logger/             # Structured logging
│   ├── metrics/            # Prometheus metrics
│   ├── middleware/         # HTTP middleware (auth, rate limit, CORS)
│   ├── models/             # Data models (28 models)
│   ├── queue/              # NATS message queue
│   ├── repository/         # Database access (12 repos)
│   ├── services/           # Business logic (14 services)
│   ├── telemetry/          # OpenTelemetry setup
│   └── utils/              # Shared utilities
│
├── docker/
│   └── postgres/           # Database initialization scripts
│
├── tests/
│   ├── e2e/                # End-to-end tests
│   ├── integration/        # Integration tests
│   ├── load/               # k6 load tests
│   ├── mocks/              # Mock implementations
│   ├── testutil/           # Test utilities
│   └── unit/               # Unit tests (27 test files)
│
└── infra/                  # Terraform and monitoring configs
```

### API Endpoints

**14 Handlers** with 40+ endpoints:

```go
// Authentication
POST   /api/v1/auth/validate       // Validate JWT token
POST   /api/v1/auth/magic/send-code // Send magic link
POST   /api/v1/auth/magic/verify    // Verify magic link code

// User Profile
GET    /api/v1/user/profile         // Get user profile
PUT    /api/v1/user/profile         // Update profile
GET    /api/v1/user/stats           // Get user statistics
GET    /api/v1/user/preferences     // Get preferences
PUT    /api/v1/user/preferences     // Update preferences
POST   /api/v1/user/sync            // Sync user data

// Workouts
GET    /api/v1/workout/logs         // Get workout history
POST   /api/v1/workout/log          // Log completed workout
GET    /api/v1/workout/analytics    // Get workout analytics

// Exercises
GET    /api/v1/exercise/user-data   // Get user exercise data
PUT    /api/v1/exercise/user-data   // Update exercise data (favorites, etc.)
GET    /api/v1/exercise/search      // Search exercises

// Gamification
POST   /api/v1/xp/award             // Award XP to user
GET    /api/v1/xp/history           // Get XP history
GET    /api/v1/quests               // Get available quests
POST   /api/v1/quests/:id/claim     // Claim quest reward

// Streaks
GET    /api/v1/streak/current/:type // Get current streak
POST   /api/v1/streak/log           // Log streak activity
POST   /api/v1/streak/batch-sync    // Batch sync streaks
GET    /api/v1/streak/history/:type // Get streak history
GET    /api/v1/streak/achievements  // Get streak achievements
GET    /api/v1/streak/analytics/:type // Get streak analytics

// Achievements
GET    /api/v1/achievements         // Get all achievements
GET    /api/v1/user/achievements    // Get user's unlocked achievements

// Goals
GET    /api/v1/goals                // Get user goals
POST   /api/v1/goals                // Create goal
PUT    /api/v1/goals/:id            // Update goal
DELETE /api/v1/goals/:id            // Delete goal

// Nutrition
POST   /api/v1/nutrition/analyze-photo  // AI food analysis
POST   /api/v1/nutrition/meal-planner   // Generate meal plan

// AI Coach
POST   /api/v1/ai/chat              // Send message to AI coach
GET    /api/v1/chat/history         // Get chat history
POST   /api/v1/chat/message         // Save chat message

// Health
GET    /api/v1/health               // Basic health check
GET    /api/v1/health/detailed      // Detailed health check
```

### Database Schema (PostgreSQL)

**14+ Tables:**

```sql
-- Core tables
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    display_name VARCHAR(100),
    avatar_url TEXT,
    current_level INTEGER DEFAULT 1,
    total_xp INTEGER DEFAULT 0,
    profile_data JSONB,
    preferences JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE workouts (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    name VARCHAR(200),
    exercises JSONB,
    duration_seconds INTEGER,
    total_volume DECIMAL,
    notes TEXT,
    completed_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE streak_entries (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    streak_type VARCHAR(50),
    date DATE,
    value INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT NOW(),
    UNIQUE(user_id, streak_type, date)
);

-- Additional tables:
-- user_quests, goals, nutrition_entries, nutrition_goals,
-- streak_achievements, user_achievements, xp_events,
-- user_counters, chat_messages, chat_conversations,
-- exercise_user_data, content_analytics
```

### AI Integration

**Smart Exercise Selection Pipeline:**

```go
func (s *AIService) GetChatResponse(ctx context.Context, userID string, message string) (*AIResponse, error) {
    // 1. Analyze query for intent
    intent := s.queryAnalyzer.Analyze(message)
    
    // 2. Get user context (workout history, favorites, fitness level)
    userContext, _ := s.userRepo.GetContext(ctx, userID)
    
    // 3. If exercise-related, get relevant exercises
    var exercises []Exercise
    if intent.IsExerciseQuery {
        exercises = s.exerciseSelector.Select(ctx, SelectCriteria{
            Query:        intent.ExerciseQuery,
            MuscleGroups: intent.MuscleGroups,
            Equipment:    userContext.AvailableEquipment,
            Level:        userContext.FitnessLevel,
            Limit:        15,
        })
    }
    
    // 4. Build AI prompt with context
    prompt := s.promptBuilder.Build(PromptContext{
        UserMessage:    message,
        UserContext:    userContext,
        Exercises:      exercises,
        ChatHistory:    s.getRecentHistory(ctx, userID),
    })
    
    // 5. Call AI service
    response, err := s.perplexityClient.Chat(ctx, prompt)
    if err != nil {
        return nil, err
    }
    
    // 6. Cache response
    s.cache.Set(ctx, cacheKey, response, 6*time.Hour)
    
    return response, nil
}
```

---

## ☁️ AWS Infrastructure

### Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                            INTERNET                                  │
│                                │                                     │
│                    ┌───────────▼───────────┐                        │
│                    │       Route 53         │                        │
│                    │     api.ohga.app       │                        │
│                    └───────────┬───────────┘                        │
└────────────────────────────────│────────────────────────────────────┘
                                 │
┌────────────────────────────────│────────────────────────────────────┐
│                         VPC (10.0.0.0/16)                           │
│                                │                                     │
│  ┌─────────────────────────────▼─────────────────────────────────┐  │
│  │                    PUBLIC SUBNETS                              │  │
│  │  ┌─────────────────────────────────────────────────────────┐  │  │
│  │  │           Application Load Balancer (ALB)                │  │  │
│  │  │     - HTTPS (443) with ACM certificate                   │  │  │
│  │  │     - HTTP/2 enabled                                     │  │  │
│  │  │     - Health checks: /health                             │  │  │
│  │  └─────────────────────────┬───────────────────────────────┘  │  │
│  └────────────────────────────│──────────────────────────────────┘  │
│                               │                                      │
│  ┌────────────────────────────│──────────────────────────────────┐  │
│  │                    PRIVATE SUBNETS                             │  │
│  │                            │                                   │  │
│  │  ┌─────────────────────────▼───────────────────────────────┐  │  │
│  │  │              ECS Fargate Service                         │  │  │
│  │  │     - ARM64 (Graviton3) for cost efficiency             │  │  │
│  │  │     - 256 CPU / 512 MB RAM                              │  │  │
│  │  │     - Auto-scaling: 1-4 tasks                           │  │  │
│  │  └──────────┬─────────────────────────────┬────────────────┘  │  │
│  │             │                             │                    │  │
│  │  ┌──────────▼──────────┐    ┌────────────▼────────────────┐  │  │
│  │  │    RDS PostgreSQL   │    │    ElastiCache Redis        │  │  │
│  │  │    db.t4g.micro     │    │    cache.t4g.micro          │  │  │
│  │  │    Multi-AZ backup  │    │    1 node cluster           │  │  │
│  │  └─────────────────────┘    └─────────────────────────────┘  │  │
│  │                                                                │  │
│  │  ┌────────────────────────────────────────────────────────┐   │  │
│  │  │               Secrets Manager                           │   │  │
│  │  │    - Database credentials                               │   │  │
│  │  │    - API keys (WorkOS, Perplexity)                     │   │  │
│  │  │    - JWT secrets                                        │   │  │
│  │  └────────────────────────────────────────────────────────┘   │  │
│  └───────────────────────────────────────────────────────────────┘  │
│                                                                      │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                       MONITORING                               │  │
│  │    - CloudWatch Logs (/ecs/dev/wellness-loop-api)             │  │
│  │    - CloudWatch Alarms (CPU, Memory, Error Rate)              │  │
│  │    - Prometheus metrics + Grafana dashboards                   │  │
│  └───────────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### Terraform Configuration

**55 Terraform files** organized by concern:

```
infra/terraform/
├── bootstrap/              # Initial setup (ECR, S3 state)
├── modules/
│   ├── vpc/                # Network infrastructure
│   ├── alb/                # Load balancer
│   ├── ecs/                # Container service
│   ├── rds/                # PostgreSQL database
│   ├── elasticache/        # Redis cache
│   └── monitoring/         # CloudWatch, alarms
└── ohga-deploy/            # Main deployment configuration
```

### Cost Optimization

| Service | Configuration | Monthly Cost |
|---------|---------------|--------------|
| ECS Fargate | ARM64 (Graviton3) | ~$10 |
| RDS PostgreSQL | db.t4g.micro | ~$12 |
| ElastiCache Redis | cache.t4g.micro | ~$12 |
| ALB | Base + requests | ~$16 |
| Route 53 | 1 hosted zone | ~$1 |
| CloudWatch | Logs + metrics | ~$1 |
| Data Transfer | ~10 GB | ~$1 |
| **Total** | | **~$53/month** |

---

## 🔄 Data Flow

### Workout Completion Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   User      │────▶│   Flutter   │────▶│   Service   │
│ Completes   │     │   UI        │     │   Layer     │
│ Workout     │     │             │     │             │
└─────────────┘     └─────────────┘     └──────┬──────┘
                                               │
                    ┌──────────────────────────┼──────────────────────────┐
                    │                          │                          │
                    ▼                          ▼                          ▼
           ┌──────────────┐           ┌──────────────┐           ┌──────────────┐
           │ Save to Drift│           │ Award XP     │           │ Update       │
           │ (Local DB)   │           │ via Provider │           │ Streaks      │
           └──────────────┘           └──────────────┘           └──────────────┘
                    │                          │                          │
                    └──────────────────────────┼──────────────────────────┘
                                               │
                                               ▼
                                      ┌──────────────┐
                                      │  SyncQueue   │
                                      │  (Pending)   │
                                      └──────┬───────┘
                                             │
                              ┌──────────────┴──────────────┐
                              │ When Online                  │
                              ▼                              │
                    ┌──────────────┐                        │
                    │  API Client  │                        │
                    │  (POST /api  │                        │
                    │   /workout)  │                        │
                    └──────┬───────┘                        │
                           │                                │
                           ▼                                │
              ┌──────────────────────┐                     │
              │   Go Backend         │                     │
              │ - Validate           │                     │
              │ - Calculate XP       │                     │
              │ - Update stats       │                     │
              │ - Check achievements │                     │
              └──────────┬───────────┘                     │
                         │                                 │
                         ▼                                 │
              ┌──────────────────────┐                     │
              │   PostgreSQL         │                     │
              │   + Redis Cache      │                     │
              └──────────────────────┘                     │
```

---

## 🔐 Security

### Authentication Flow

```
1. User enters email
2. Backend sends magic link via WorkOS
3. User clicks link / enters code
4. Backend validates with WorkOS
5. Backend issues JWT (24h expiry)
6. Flutter stores JWT securely
7. All API requests include Bearer token
8. Backend validates JWT on each request
```

### Security Measures

| Layer | Protection |
|-------|------------|
| Transport | HTTPS with TLS 1.3 |
| Authentication | JWT with RS256 signing |
| Authorization | Role-based access control |
| API | Rate limiting (100 req/min) |
| Database | VPC-only access, encrypted at rest |
| Secrets | AWS Secrets Manager |
| Logging | No PII in logs |

---

## 📈 Performance

### Frontend Metrics

| Metric | Target | Actual |
|--------|--------|--------|
| Cold Start | < 3s | 1.8s |
| Warm Start | < 1s | 0.6s |
| App Size | < 60MB | 48MB |
| Memory Usage | < 150MB | 118MB |
| Frame Rate | 60 fps | 60 fps |

### Backend Metrics

| Metric | Target | Actual |
|--------|--------|--------|
| P50 Latency | < 100ms | 45ms |
| P99 Latency | < 500ms | 180ms |
| Throughput | 1000 rps | 1200 rps |
| Error Rate | < 0.1% | 0.02% |

---

**Next: [Amazon Kiro Usage Proof](./03-AMAZON-KIRO-USAGE.md)** →

