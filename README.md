# 🚀 AlgoTrackr

> **Track Smarter. Practice Better. Crack Faster.**

AlgoTrackr is an AI-powered DSA Practice Tracker and Mentor built with the MERN stack + ML microservice.
Designed for students preparing for placements — track problems, visualize progress, get AI recommendations and explanations.

## Table of Contents

- [Why AlgoTrackr?](#why-algotrackr)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture Overview](#architecture-overview)
- [Getting Started (Quickstart)](#getting-started-quickstart)
- [Environment Variables](#environment-variables)
- [Project Structure](#project-structure)
- [Database Schemas (Example)](#database-schemas-example)
- [API Endpoints (Examples)](#api-endpoints-examples)
- [ML Service & LLM Integration](#ml-service--llm-integration)
- [Development & Testing](#development--testing)
- [Deployment Notes](#deployment-notes)
- [Performance & Security](#performance--security-best-practices)
- [Roadmap & Future Enhancements](#roadmap--future-enhancements)
- [Contributing](#contributing)
- [License & Credits](#license--credits)
- [Contact](#contact)

---

## Why AlgoTrackr?

Many learners use lists, spreadsheets, or standalone coding sites but lack an integrated, analytics-driven study flow. AlgoTrackr combines:

- **Structured problem logging** (title, topic, difficulty, approach, time)
- **Topic-level & difficulty analytics**
- **Rule/ML-based recommendations** for what to solve next
- **LLM-powered explanations** of incorrect approaches
- **Placement-readiness scoring** and exportable analytics

This README is developer focused — it contains enough to get a developer up and running, extend the app, and deploy it.

## Features

### MVP
- ✅ Email/password auth (JWT + Refresh tokens)
- ✅ Add / Edit / Delete problem entries
- ✅ Dashboard: topic breakdown, difficulty distribution, streaks
- ✅ Rule-based recommendation engine
- ✅ AI Explain endpoint (LLM proxy)
- ✅ Basic ML microservice stub (FastAPI)

### v1 / Future
- [ ] ML ranking model for personalized recommendations
- [ ] Offline training pipeline & scheduled retraining
- [ ] Exportable resume-ready analytics (PDF)
- [ ] Chrome extension, interview simulator, leaderboards, badges

## Tech Stack

### Frontend
- **React + TypeScript**
- **Tailwind CSS**
- Recharts (charts)
- React Query / Axios

### Backend
- **Node.js + Express (TypeScript)**
- Mongoose (MongoDB)
- Zod (validation)
- JWT + bcrypt
- Helmet, rate-limiter, CORS

### ML / LLM
- **Python 3.10+, FastAPI**
- scikit-learn / LightGBM / pandas
- joblib for model persistence
- OpenAI (or local LLM) for explanations

### DevOps
- Docker / Docker Compose
- MongoDB Atlas
- Redis (BullMQ for background jobs)
- Vercel (frontend) / Render or Railway (backend & ML)
- GitHub Actions (CI/CD)

## Architecture Overview

```mermaid
graph LR
    User[React SPA] <--> API[Express API]
    API <--> DB[MongoDB Atlas]
    API --> ML[ML Service FastAPI]
    API --> LLM[LLM Provider OpenAI/local]
    API --> Job[Background Jobs Redis + BullMQ]
```

*Text representation:*
```
[React SPA]  <-->  [Express API]  <-->  [MongoDB Atlas]
                       |
                       +--> [ML Service (FastAPI)]
                       |
                       +--> [LLM Provider (OpenAI/local)]
                       |
                       +--> [Background Jobs (Redis + BullMQ)]
```

- **Frontend** makes authenticated calls to Express.
- **Express** serves REST API, persists data into MongoDB, and calls ML/LLM microservices.
- **ML microservice** provides `/predict` endpoints for recommendations and can serve trained models.
- **LLM usage** is proxied through backend to hide API keys and rate-limit usage.

## Getting Started (Quickstart)

These commands assume you have Docker, Node (>=16), npm/yarn, Python 3.10+, and pip installed.

### 1. Clone

```bash
git clone https://github.com/yourusername/algotrackr.git
cd algotrackr
```

### 2. Run with Docker Compose (dev)

A simple dev compose that runs server, client, mongo, redis, ml-service.

```bash
docker compose up --build
```

Visit:
- **Frontend**: http://localhost:3000
- **Backend**: http://localhost:5000
- **ML Service**: http://localhost:8000/docs (FastAPI docs)

### 3. Manual dev runs (optional)

**Backend**
```bash
cd server
cp .env.example .env
npm install
npm run dev
```

**Frontend**
```bash
cd client
cp .env.example .env
npm install
npm start
```

**ML Service**
```bash
cd ml-service
python -m venv .venv
# Windows: .venv\Scripts\activate
source .venv/bin/activate
pip install -r requirements.txt
uvicorn app:app --reload --host 0.0.0.0 --port 8000
```

## Environment Variables

Create `.env` files for server and ml-service. Example keys:

**server/.env**
```env
PORT=5000
MONGO_URI=mongodb+srv://<user>:<pass>@cluster0.mongodb.net/algotrackr?retryWrites=true&w=majority
JWT_SECRET=supersecret_jwt_key
JWT_REFRESH_SECRET=another_refresh_secret
OPENAI_API_KEY=sk-...
ML_SERVICE_URL=http://localhost:8000
NODE_ENV=development
```

**ml-service/.env**
```env
ML_MODEL_PATH=/models/recommender.joblib
LOG_LEVEL=info
```

> **Note**: Do not commit `.env` to Git. Use GitHub secrets or your platform's secret manager for production.

## Project Structure (recommended monorepo)

```
algotrackr/
├── client/            # React frontend (TypeScript + Tailwind)
├── server/            # Express backend (TypeScript)
│   ├── src/
│   │   ├── controllers/
│   │   ├── routes/
│   │   ├── models/
│   │   ├── services/
│   │   ├── middleware/
│   │   └── server.ts
│   └── package.json
├── ml-service/        # FastAPI ML microservice (Python)
├── docker-compose.yml
├── .github/
│   └── workflows/     # CI pipelines
└── README.md
```

## Database Schemas (Example)

These are example Mongoose payloads / JSON documents.

**users**
```json
{
  "_id": "ObjectId",
  "name": "Harsh Sharma",
  "email": "harsh@example.com",
  "passwordHash": "<bcrypt>",
  "role": "user",
  "meta": { "streak": 0, "totalSolved": 0 },
  "createdAt": "ISODate"
}
```

**problems**
```json
{
  "_id": "ObjectId",
  "userId": "ObjectId",
  "title": "Maximum Subarray",
  "platform": "leetcode",
  "link": "https://leetcode.com/problems/maximum-subarray/",
  "topics": ["arrays","kadane"],
  "difficulty": "easy",
  "timeTakenMins": 22,
  "solved": true,
  "approach": "I iterated and maintained running sum, but failed on negatives",
  "mistakes": "Edge case when all negatives",
  "meta": { "pattern": "kadane" },
  "createdAt": "ISODate"
}
```

**analytics (denormalized)**
```json
{
  "userId": "ObjectId",
  "topicStats": {
    "arrays": { "attempted": 20, "solved": 18, "avgTime": 23 },
    "dp": { "attempted": 8, "solved": 3, "avgTime": 50 }
  },
  "difficultyStats": { "easy": 50, "medium": 30, "hard": 20 },
  "placementScore": 68,
  "lastUpdated": "ISODate"
}
```

**ai_logs**
```json
{
  "userId": "ObjectId",
  "problemId": "ObjectId",
  "type": "explain",
  "payload": { "input": "...", "output": "..." },
  "createdAt": "ISODate"
}
```

> Use indexes on `users.email`, `problems.userId`, `problems.topics` for performance.

## API Endpoints (Examples)

All endpoints prefixed by `/api/v1`. Authentication uses `Authorization: Bearer <token>`.

### Auth
- `POST /api/v1/auth/register` - `{ name, email, password }`
- `POST /api/v1/auth/login` - `{ email, password } => { accessToken, refreshToken }`
- `POST /api/v1/auth/refresh` - `{ refreshToken } => new accessToken`

### Problems
- `GET  /api/v1/problems?topic=dp&difficulty=hard`
- `POST /api/v1/problems`
- `PUT  /api/v1/problems/:id`
- `DELETE /api/v1/problems/:id`

### Analytics
- `GET /api/v1/analytics/dashboard` - Response: `{ topicStats, difficultyStats, progress, placementScore }`

### AI
- `POST /api/v1/ai/recommend` - `{ userId } => { recommendations: [{problemId, title, reason, score}, ...] }`
- `POST /api/v1/ai/explain` - `{ problemTitle, approachText, code (optional) } => { explanation, suggestions }`

### ML
- `POST http://<ml-service>/predict` - `{ userFeatures } => { recommendations: [{problemId,score}, ...] }`

## ML Service & LLM Integration

### Strategy
- **MVP**: Rule-based recommender using analytics (determine weakest topic, suggest next difficulty).
- **v1**: Train a supervised model (RandomForest/LightGBM) to rank candidate problems by success probability.
- **LLM for “explain”**: Use OpenAI or a local LLM to produce human-friendly explanations of incorrect approaches.

### Example FastAPI predict endpoint

```python
# ml-service/app.py (FastAPI)
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()

class UserFeatures(BaseModel):
    user_id: str
    topic_accuracy: dict
    avg_time: dict

@app.post("/predict")
def predict(features: UserFeatures):
    # load model, compute ranking of candidate problems
    return {"recommendations": [{"problemId":"p123", "score":0.85}]}
```

### Important
- Do not call LLM directly from the frontend — proxy via backend to hide keys and rate-limit.
- Cache LLM / ML responses where useful.
- Store `ai_logs` for auditing, debugging and future training data.

## Development & Testing

### Testing
- **Backend**: Jest + Supertest
- **Frontend**: Jest + React Testing Library
- **E2E**: Playwright / Cypress
- **ML**: pytest + unit tests for preprocess & predict functions

### Lint / Format
- ESLint (TypeScript)
- Prettier
- `black` / `isort` for Python

### CI
- GitHub Actions for lint → tests → build
- Run unit tests and run a lightweight integration test against test instance of services

## Deployment Notes

### Frontend
- Vercel (automatic builds) or S3 + CloudFront

### Backend & ML Service
- Render / Railway for hobby / staging
- Docker images on AWS ECS / DigitalOcean / GCP for production
- Use managed MongoDB Atlas
- Use Managed Redis for queues (BullMQ)
- Use secret stores for keys (Render secrets, GitHub Actions secrets, AWS Secrets Manager)

### Scaling
- Horizontal scale backend behind a load balancer
- Autoscale ML workers if heavy inference load
- Use caching (Redis) for repeated expensive results

## Performance & Security Best Practices

### Performance
- Index MongoDB on frequently queried fields
- Use aggregation pipelines to push work to DB
- Cache ML/LLM results for identical requests
- Use pagination for large lists

### Security
- Hash passwords with bcrypt/argon2
- Use HTTPS & secure cookies for tokens
- Sanitize user inputs (avoid SQL/NoSQL injection)
- Rate-limit endpoints (esp. LLM usage)
- Validate requests with Zod
- Keep dependencies up-to-date (Dependabot/Snyk)

## Roadmap & Future Enhancements
- [ ] Improved ranking models (LightGBM/XGBoost)
- [ ] Knowledge graph of DSA concepts & dependencies for guided learning paths
- [ ] Chrome extension for auto import from platforms
- [ ] Interview simulator (timed sessions + scoring)
- [ ] Leaderboards, badges & community features
- [ ] Mobile app (React Native)

## Contributing
1. Fork the repo
2. Create a feature branch (`git checkout -b feat/awesome`)
3. Commit with clear messages
4. Push and open a Pull Request
5. Keep PRs small and focused; include tests

Please follow the code style and run tests locally before opening a PR.

## License & Credits
- **MIT License**
- **Creator**: Harsh Sharma
- **Project name**: AlgoTrackr — DSA Practice Tracker with AI Mentor

## Contact
If you want help building this, want the starter repo scaffolded, CI/CD workflows, Docker Compose & sample .env files, ML training notebook, or architecture diagrams — ping me in the repo issues or contact: panditharshsharma34@gmail.com
