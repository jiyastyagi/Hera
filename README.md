# Hera

Full-stack women's wellness platform with AI-powered food scanning, menstrual cycle prediction via machine learning, mood and symptom tracking, and exercise logging.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [System Architecture](#system-architecture)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Environment Variables](#environment-variables)
- [API Reference](#api-reference)
- [ML Microservice](#ml-microservice)
- [Running Tests](#running-tests)
- [Deployment](#deployment)
- [Roadmap](#roadmap)
- [License](#license)

---

## Overview

Hera is a free, all-in-one wellness application built specifically for women. It combines period and ovulation tracking, daily mood and symptom logging, exercise and calorie management, and a smart food scanner powered by GPT-4o Vision. A separate Python microservice runs a scikit-learn model to predict upcoming period dates and fertile windows based on a user's historical cycle data.

The project is built on the MERN stack with a FastAPI microservice for machine learning, deployed on Render and Vercel with a GitHub Actions CI pipeline.

---

## Features

### Cycle Tracking
- Log period start and end dates
- View cycle history with a visual calendar
- Get predicted next period date and fertile window from the ML model
- See average cycle length and trend over time

### AI Food Scanner
- Photograph any meal and upload it directly in the app
- GPT-4o Vision identifies food items and estimates portion sizes
- Nutritionix API fetches accurate macros and calorie data per item
- All meals are saved to a daily food log with macro breakdown charts

### Mood and Symptom Logging
- Log daily mood on a five-point scale with optional notes
- Track physical symptoms such as cramps, bloating, fatigue, and headaches
- View 30-day mood trend charts on the dashboard

### Exercise Tracking
- Log workouts by type, duration, and intensity
- MET-based calorie burn calculation
- Weekly exercise summary with total active minutes and calories burned

### Analytics Dashboard
- Mood trend line chart (Recharts)
- Daily and weekly calorie bar charts
- Cycle history calendar with phase highlights
- Macro distribution pie chart per day

### Authentication and Security
- JWT-based authentication with HTTP-only refresh tokens
- Passwords hashed with bcrypt (12 salt rounds)
- Rate limiting on all auth routes (100 requests per 15 minutes)
- Helmet.js security headers on all responses
- Input validation with express-validator on every endpoint

### PWA Support
- Installable on Android and desktop
- Push notifications for upcoming period reminders
- Offline caching for the dashboard and recent logs

---

## Tech Stack

### Frontend
| Technology | Purpose |
|---|---|
| React 18 | UI framework |
| Vite | Build tool and dev server |
| React Router v6 | Client-side routing |
| Recharts | Dashboard data visualisation |
| Axios | HTTP client with JWT interceptor |
| vite-plugin-pwa | Service worker and push notifications |
| date-fns | Date formatting and cycle math |

### Backend
| Technology | Purpose |
|---|---|
| Node.js | Runtime |
| Express.js | Web framework |
| MongoDB Atlas | Database |
| Mongoose | ODM with schema validation |
| JSON Web Tokens | Authentication |
| bcryptjs | Password hashing |
| Multer | Food photo upload handling |
| Sharp | Image compression before AI call |
| Helmet.js | HTTP security headers |
| express-rate-limit | Brute-force protection |
| OpenAI SDK | GPT-4o Vision integration |
| Axios | Nutritionix API and ML service calls |

### ML Microservice
| Technology | Purpose |
|---|---|
| Python 3.11 | Runtime |
| FastAPI | Web framework |
| Uvicorn | ASGI server |
| scikit-learn | Cycle prediction model |
| pandas / numpy | Data processing |
| joblib | Model serialisation |
| Pydantic | Request and response validation |

### Infrastructure
| Technology | Purpose |
|---|---|
| MongoDB Atlas | Cloud database (free tier) |
| Vercel | React frontend deployment |
| Render | Node API and FastAPI deployment |
| GitHub Actions | CI pipeline (lint + test on every push) |
| Docker Compose | Local development orchestration |

---

## System Architecture

```
Browser (React PWA)
        |
        | HTTPS + JWT
        v
Node.js / Express API  (Render — port 5000)
        |
        |--- MongoDB Atlas (Mongoose ODM)
        |
        |--- OpenAI GPT-4o Vision API
        |         (food photo identification)
        |
        |--- Nutritionix API
        |         (calorie and macro lookup)
        |
        |--- FastAPI ML Service  (Render — port 8000)
                  (cycle length prediction model)
```

All log collections in MongoDB use a compound index on `{ userId: 1, date: -1 }` for fast date-range queries per user. The FastAPI service is internal-only — it is not exposed to the public internet and only accepts requests from the Node server.

---

## Project Structure

```
hera/
├── client/                          # React + Vite frontend
│   ├── src/
│   │   ├── components/
│   │   │   ├── cycle/               # CycleRing, CalendarWidget
│   │   │   ├── food/                # FoodCard, MacroChart
│   │   │   ├── mood/                # MoodSelector, SymptomPicker
│   │   │   └── shared/              # Navbar, PrivateRoute, Loader
│   │   ├── pages/                   # Dashboard, CycleTracker, FoodLog, etc.
│   │   ├── hooks/                   # useAuth, useCycle, useFoodLog, useMood
│   │   ├── context/                 # AuthContext (global JWT state)
│   │   ├── services/                # Axios API call functions
│   │   └── utils/                   # dateHelpers, constants
│   ├── sw.js                        # Service worker (PWA + push)
│   └── vite.config.js
│
├── server/                          # Node.js + Express API
│   ├── config/                      # DB connection, API keys
│   ├── controllers/                 # Route handler functions
│   ├── middleware/                  # Auth, rate limiter, upload, error handler
│   ├── models/                      # Mongoose schemas (User, CycleLog, etc.)
│   ├── routes/                      # Express routers
│   ├── services/                    # AI, nutrition, ML bridge logic
│   ├── utils/                       # JWT helpers, response helpers
│   ├── __tests__/                   # Jest + Supertest test suite
│   └── server.js                    # Entry point
│
├── ml/                              # Python FastAPI microservice
│   ├── app.py                       # FastAPI app entry point
│   ├── routers/                     # predict.py, health.py
│   ├── models/                      # cycle_model.pkl, Pydantic schemas
│   ├── training/                    # train.py, generate_data.py
│   └── requirements.txt
│
├── .github/workflows/ci.yml         # GitHub Actions CI
├── docker-compose.yml               # Local dev orchestration
├── .env.example                     # Environment variable template
└── README.md
```

---

## Getting Started

### Prerequisites

- Node.js 18 or higher
- Python 3.11 or higher
- MongoDB Atlas account (free tier is sufficient)
- OpenAI API key (GPT-4o access required)
- Nutritionix API credentials (free tier available)

### 1. Clone the repository

```bash
git clone https://github.com/yourusername/hera.git
cd hera
```

### 2. Set up environment variables

```bash
copy .env.example server/.env
```

Fill in all values in `server/.env` — see the [Environment Variables](#environment-variables) section below.

### 3. Install server dependencies

```bash
cd server
npm install
cd ..
```

### 4. Install client dependencies

```bash
cd client
npm install
cd ..
```

### 5. Set up the Python ML service

```powershell
cd ml
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

Train the initial model using synthetic data:

```bash
python training/generate_data.py
python training/train.py
```

This writes `ml/models/cycle_model.pkl`.

### 6. Run all three services

Option A — Docker (recommended):

```bash
docker-compose up
```

Option B — Three separate terminals:

```bash
# Terminal 1 — Node API
cd server && npm run dev

# Terminal 2 — React client
cd client && npm run dev

# Terminal 3 — FastAPI ML service
cd ml && venv\Scripts\activate && uvicorn app:app --reload --port 8000
```

The app is available at `http://localhost:5173`.

---

## Environment Variables

Create `server/.env` using `.env.example` as a template.

```env
# MongoDB
MONGO_URI=mongodb+srv://<user>:<password>@cluster.mongodb.net/hera

# JWT
JWT_SECRET=your_long_random_secret_here
JWT_REFRESH_SECRET=another_long_random_secret
JWT_EXPIRES_IN=15m
JWT_REFRESH_EXPIRES_IN=7d

# OpenAI
OPENAI_API_KEY=sk-...

# Nutritionix
NUTRITIONIX_APP_ID=your_app_id
NUTRITIONIX_API_KEY=your_api_key

# ML microservice
ML_SERVICE_URL=http://localhost:8000

# Web Push (VAPID)
VAPID_PUBLIC_KEY=your_vapid_public_key
VAPID_PRIVATE_KEY=your_vapid_private_key
VAPID_EMAIL=mailto:you@example.com

# Server
PORT=5000
NODE_ENV=development
```

Generate VAPID keys with:

```bash
npx web-push generate-vapid-keys
```

---

## API Reference

All endpoints are prefixed with `/api`. Protected routes require the header:

```
Authorization: Bearer <access_token>
```

### Auth

| Method | Endpoint | Description |
|---|---|---|
| POST | /api/auth/register | Create account |
| POST | /api/auth/login | Login, returns access + refresh token |
| POST | /api/auth/refresh | Refresh access token |
| POST | /api/auth/logout | Invalidate refresh token |

### Cycle Tracking

| Method | Endpoint | Description |
|---|---|---|
| POST | /api/cycles/log | Log a period entry |
| GET | /api/cycles/history | Get all past cycles |
| GET | /api/cycles/stats | Average length, last period date |
| GET | /api/cycles/predict | Get ML prediction for next period |

### Mood and Symptoms

| Method | Endpoint | Description |
|---|---|---|
| POST | /api/mood/log | Log today's mood and symptoms |
| GET | /api/mood/history | Get entries for a date range |
| GET | /api/mood/trend | 30-day aggregated mood trend |

### Food and Nutrition

| Method | Endpoint | Description |
|---|---|---|
| POST | /api/food/log | Manually log a meal |
| GET | /api/food/daily | Get today's food log with totals |
| GET | /api/food/history | Weekly calorie history |
| POST | /api/food/scan | Upload a food photo for AI analysis |

### Exercise

| Method | Endpoint | Description |
|---|---|---|
| POST | /api/exercise/log | Log a workout |
| GET | /api/exercise/weekly | Weekly summary with calories burned |
| GET | /api/exercise/history | Full exercise history |

---

## ML Microservice

The FastAPI service runs independently on port 8000. It exposes two endpoints:

### POST /predict

Accepts the user's last 6 cycle lengths and returns a prediction.

Request body:
```json
{
  "cycle_history": [28, 30, 27, 29, 31, 28]
}
```

Response:
```json
{
  "next_period_date": "2025-02-14",
  "fertile_window_start": "2025-01-31",
  "fertile_window_end": "2025-02-04",
  "predicted_cycle_length": 29,
  "confidence": 0.82
}
```

### GET /health

Returns `{ "status": "ok" }`. Used by Render for health checks.

### Retraining the model

```bash
cd ml
venv\Scripts\activate
python training/generate_data.py   # or supply real anonymised data
python training/train.py           # overwrites cycle_model.pkl
```

The Node server loads predictions lazily — no restart needed after retraining.

---

## Running Tests

### Server tests (Jest + Supertest)

```bash
cd server
npm test
npm run test:coverage
```

Tests use `mongodb-memory-server` — no real database connection required. Coverage target is 70% across controllers and services.

### ML tests (pytest)

```bash
cd ml
venv\Scripts\activate
pip install pytest
pytest
```

### Linting

```bash
# Client
cd client && npm run lint

# Server
cd server && npm run lint
```

---

## Deployment

### Frontend — Vercel

1. Connect the GitHub repo to Vercel
2. Set root directory to `client`
3. Build command: `npm run build`
4. Output directory: `dist`
5. Add `VITE_API_URL=https://your-render-api.onrender.com` as an environment variable

### Node API — Render

1. New Web Service, connect GitHub repo
2. Root directory: `server`
3. Build command: `npm install`
4. Start command: `node server.js`
5. Add all environment variables from `.env` in the Render dashboard

### FastAPI ML Service — Render

1. New Web Service, connect same repo
2. Root directory: `ml`
3. Build command: `pip install -r requirements.txt`
4. Start command: `uvicorn app:app --host 0.0.0.0 --port 8000`
5. Set `ML_SERVICE_URL` in the Node service to this Render URL

### Database — MongoDB Atlas

1. Create a free M0 cluster
2. Whitelist all IPs (`0.0.0.0/0`) for Render compatibility
3. Copy the connection string into `MONGO_URI`

---

## Roadmap

- Google OAuth login
- Export cycle and nutrition data as CSV
- Partner sharing mode (share cycle data with a trusted person)
- Symptom-to-cycle correlation analysis
- Retrain the ML model automatically as users log more cycles
- iOS and Android apps via React Native

---

## License

MIT License. See `LICENSE` for details.