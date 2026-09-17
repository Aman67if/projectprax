# 🚀 Praxes — EdTech Assessment Platform

An Under development, full-stack educational testing and assessment platform. **Praxes** empowers educational institutions to manage structured curriculums, author complex STEM questions with LaTeX and diagrams, facilitate dynamic test-taking with deterministic evaluation, and monetize via subscription plans.

---

## 🧭 Explore Sub-Projects

Detailed documentation, architecture notes, and API references are available in each sub-project:

| Component | Description | Tech Stack | Documentation |
| :--- | :--- | :--- | :--- |
| **Frontend** | Modern role-based web app with LaTeX rendering & live test engine | Next.js 15, React 19, Tailwind v4, MathJax | [📖 Frontend README](./Frontend/README.md) |
| **Backend** | High-performance REST API with caching, ORM, and payment processing | Express, TypeScript, PostgreSQL, Prisma, Redis | [📖 Backend README](./Backend/README.md) |

---

## 🏛️ System Architecture

```
                                  ┌────────────────────────┐
                                  │   Next.js 15 Frontend  │
                                  │ (Port 3000: App Router)│
                                  └───────────┬────────────┘
                                              │ HTTP / JSON
                                              │ (with JWT HttpOnly cookie)
                                              ▼
                                  ┌────────────────────────┐
                                  │   Express TypeScript   │
                                  │      REST API          │
                                  │      (Port 5000)       │
                                  └─┬─────┬──────┬───────┬─┘
                                    │     │      │       │
              ┌─────────────────────┘     │      │       └─────────────────────┐
              ▼                           ▼      ▼                             ▼
    ┌──────────────────┐        ┌──────────────┐ ┌─────────────┐     ┌──────────────────┐
    │ PostgreSQL DB    │        │  Redis Cache │ │ AWS S3      │     │  Paytm Gateway   │
    │ (Prisma ORM)     │        │  (Port 6379) │ │ (Images)    │     │  (Subscriptions) │
    └──────────────────┘        └──────────────┘ └─────────────┘     └──────────────────┘
```

---

## 🔄 Core Application Flows

### 1. User Roles & Access Hierarchy

The application enforces strict Role-Based Access Control (RBAC) at both backend middleware (`protect`, `restrictTo`) and frontend layout guards (`useAuth`):

```
       ┌──────────┐
       │   User   │
       └────┬─────┘
            │ Logs In
            ▼
    ┌───────────────────────────────────────────────┐
    │ Check Role in JWT Cookie                      │
    └───┬───────────────────┬───────────────────┬───┘
        │                   │                   │
        ▼                   ▼                   ▼
  ┌───────────┐       ┌───────────┐       ┌───────────┐
  │  STUDENT  │       │ EMPLOYEE  │       │   ADMIN   │
  └─────┬─────┘       └─────┬─────┘       └─────┬─────┘
        │                   │                   │
        ├─ Browse tests     ├─ Question entry   ├─ User & employee CRUD
        ├─ Attempt tests    ├─ LaTeX editing    ├─ Question review & approval
        ├─ View results     ├─ Image upload     ├─ Test group management
        ├─ View analytics   ├─ Fix corrections  ├─ Subscription plans
        └─ Buy subscription └─ View rejections  └─ Platform metrics
```

| Role | Access Scope | Landing Dashboard |
| :--- | :--- | :--- |
| **Student** | Attempt tests, view scores/history, track analytics, subscribe to plans | `/student` |
| **Employee** | Create questions (LaTeX + S3 images), manage edits, fix flagged questions | `/employee` |
| **Admin** | Full system oversight: approve/reject questions, manage users & plans, view metrics | `/admin` |

---

### 2. Question Lifecycle & Review Pipeline

Every question submitted by an employee undergoes an editorial workflow before becoming available in student tests:

```
[ Employee ] ──▶ Submits Question ──▶ Status: PENDING
                                            │
                                            ▼
                                    [ Admin Review ]
                                     /      │       \
                                    /       │        \
                        [ APPROVE ]    [ REJECT ]   [ REQUEST CORRECTION ]
                             │              │                  │
                             ▼              ▼                  ▼
                         APPROVED       REJECTED       NEEDS_CORRECTION
                      (Live in Tests)   (Archived)             │
                                                               ▼
                                                       [ Employee Edits ]
                                                               │
                                                               └──▶ Resubmitted as PENDING
```

---

### 3. Test-Taking & Deterministic Evaluation

To guarantee fairness and anti-cheating protection, tests operate on an immutable snapshot model:

```
1. Student Initiates Test
   │
   ├──▶ Validates active subscription (for premium tests)
   ├──▶ Selects required number of APPROVED questions
   ├──▶ Shuffles MCQ options ONCE and saves a frozen test snapshot (JSONB)
   └──▶ Correct answers are STRIPPED before returning payload to student

2. Live Test Session
   │
   ├──▶ Full-screen UI with countdown timer
   ├──▶ Option selection saved locally & synced
   └──▶ Auto-submits on timer expiry or manual submission

3. Deterministic Evaluation
   │
   ├──▶ Evaluated strictly against the stored JSONB snapshot (immune to question edits)
   ├──▶ Calculates score, accuracy, subject-wise breakdown, and time spent
   └──▶ Reveals full answer keys and LaTeX explanations post-submission
```

---

### 4. Subscription & Monetization Flow

```
[ Student ] ──▶ Views Plans ──▶ Clicks "Subscribe"
                                       │
                                       ▼
                       Backend generates Paytm Txn Token
                                       │
                                       ▼
                         Paytm Payment Gateway Screen
                                  /         \
                         (Success)           (Failure)
                            │                    │
                            ▼                    ▼
                   Paytm Webhook Callback    Redirects to
                 (Verified via Checksum)    Retry Screen
                            │
                            ▼
              Subscription Activated (Valid for N days)
              Premium Test Groups Unlocked
```

---

## 📊 Current Status of Working Model

> [!NOTE]
> **Development Phase:** Praxes is actively **under development**. The table below reflects the actual working state of each module in the current local development build:

| Module / Feature | Current Status | Working State & Details |
| :--- | :---: | :--- |
| **Authentication & RBAC** | ✅ Working (Local) | JWT via HttpOnly cookies, bcrypt hashing, role guards (`STUDENT`, `EMPLOYEE`, `ADMIN`). |
| **Curriculum Hierarchy** | ✅ Working (Local) | 4-level tree (`Class` → `Subject` → `Chapter` → `Topic`) via DB seed and cascading UI dropdowns. |
| **Question Authoring** | ✅ Working (Local) | MCQ entry with real-time MathJax LaTeX preview, answer options, and topic selection. |
| **Question Moderation** | ✅ Working (Local) | Admin review queue: Approve, Reject, and Request Correction; Employee revision and resubmit. |
| **Test Engine & Session** | ✅ Working (Local) | Immutable JSONB test snapshots, single-instance option shuffle, full-screen timer, and auto-submit. |
| **Evaluation & Scoring** | ✅ Working (Local) | Instant evaluation against snapshot, score breakdown, timing metrics, and post-test review with LaTeX. |
| **Student Analytics** | ✅ Working (Local) | Test attempt history, result breakdowns, and basic performance review. |
| **Admin Control Panel** | ✅ Working (Local) | Employee user creation/management, test group creation, and overview platform metrics. |
| **Redis Caching Layer** | ⚙️ Config-Dependent | Works when Redis is active (port 6379); backend gracefully falls back with a warning if offline. |
| **Image Uploads (AWS S3)**| ⚙️ Config-Dependent | S3 upload client implemented; requires valid AWS credentials in `.env` to store images. |
| **Paytm Payment Gateway** | 🚧 In Development | Processed with test credentials provided by Paytm via [ngrok](https://ngrok.com/) (required because Paytm servers cannot access local instances for webhook callbacks). |

---

## 📁 Repository Structure

```
projectprax/
├── .gitmodules               # Git submodule definitions for Frontend & Backend
├── .gitignore                # Root git ignore rules (excludes submodules & temp files)
├── Frontend/                 # Next.js 15 Web Application (Git Submodule)
│   ├── app/                  # App Router: /admin, /employee, /student, /auth
│   ├── components/           # UI components (MathRenderer, Card, Modal, Sidebar)
│   ├── hooks/                # React hooks (useAuth, useCurriculum)
│   ├── lib/                  # Fetch wrapper with auto-auth and ngrok headers
│   └── README.md             # Detailed Frontend documentation
│
├── Backend/                  # Express.js REST API (Git Submodule)
│   ├── prisma/               # Schema, migrations, and database seed script
│   ├── src/
│   │   ├── config/           # Prisma, Redis, S3, Paytm, and Logger singletons
│   │   ├── controllers/      # Handlers for Auth, Questions, Tests, Admin, Payments
│   │   ├── middleware/       # Auth guards, error handler, input validators
│   │   ├── routes/           # Express router endpoints
│   │   ├── services/         # Business logic (TestEngine, Cache, S3, Payments)
│   │   └── utils/            # JWT, AppError, shuffleOptions utilities
│   └── README.md             # Detailed Backend documentation
│
└── README.md                 # Root documentation (this file)
```

---

## 🏁 Quick Setup Guide

Follow these steps to run the entire platform on your local machine:

### Prerequisites

- **Git** (with submodule support)
- **Node.js** (v18+) or **Bun**
- **PostgreSQL** (v14+)
- **Redis** (local service, Docker, or WSL)

---

### Step 1: Clone Repository with Submodules

`Frontend` and `Backend` are maintained in independent repositories and linked to this root project as Git submodules.

#### Option A: Clone with submodules directly (Recommended)

Clone the root repository and automatically download all submodules:

```bash
git clone --recursive <ROOT_REPO_URL>
cd projectprax
```

#### Option B: If you already ran a standard `git clone`

If you ran `git clone <ROOT_REPO_URL>` without `--recursive`, the submodule folders will be empty initially. Initialize and fetch them using:

```bash
git submodule update --init --recursive
```

#### Keeping Submodules Updated

To pull the latest changes across all submodules from their respective remotes at any time:

```bash
git submodule update --remote --recursive
```

---

### Step 2: Start the Backend

```bash
cd Backend

# 1. Environment variables
cp .env.example .env
# Fill in your DATABASE_URL, REDIS_PORT, and JWT_SECRET

# 2. Install dependencies (choose your package manager)
npm install        # or: bun install / pnpm install / yarn install

# 3. Apply database migrations & seed initial accounts
npx prisma migrate dev
npx prisma generate
npx ts-node prisma/seed.ts

# 4. Start Redis (e.g. using Docker)
docker run -d --name edtech-redis -p 6379:6379 redis:alpine

# 5. Start development server
npm run dev        # or: bun run dev / pnpm dev / yarn dev
```

*The Backend API runs at `http://localhost:5000`.*

---

### Step 3: Start the Frontend

In a new terminal window:

```bash
cd Frontend

# 1. Environment variables
cp .env.example .env.local
# Ensure NEXT_PUBLIC_API_URL=http://localhost:5000/api

# 2. Install dependencies (choose your package manager)
bun install        # or: npm install / pnpm install / yarn install

# 3. Start development server
bun --bun run dev  # or: npm run dev / pnpm dev / yarn dev
```

*The Frontend application runs at `http://localhost:3000`.*

---

### Step 4: Default Credentials (from Seed)

Once seeded, you can sign in with any of the default test accounts:

| Role | Email | Password |
| :--- | :--- | :--- |
| **Admin** | `admin@praxes.com` | `Admin@123` |
| **Employee** | `employee@praxes.com` | `Employee@123` |
| **Student** | `student@praxes.com` | `Student@123` |

---

## 🔗 Sub-Project Documentation Links

- [Frontend Documentation (Pages, Components, Package Managers, MathJax)](./Frontend/README.md)
- [Backend Documentation (Architecture, Prisma Schema, Redis, APIs, Payments)](./Backend/README.md)
