<div align="center">

<img src="https://img.shields.io/badge/RecipeVault-🍽️-orange?style=for-the-badge" alt="RecipeVault"/>

# RecipeVault

**A full-stack recipe management system — search, filter, and manage your recipes with a clean UI, RESTful API, and MongoDB NoSQL backend.**

[![Node.js](https://img.shields.io/badge/Node.js-≥18-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org)
[![Express](https://img.shields.io/badge/Express-4.x-000000?style=flat-square&logo=express&logoColor=white)](https://expressjs.com)
[![MongoDB](https://img.shields.io/badge/MongoDB-6-47A248?style=flat-square&logo=mongodb&logoColor=white)](https://www.mongodb.com)
[![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker&logoColor=white)](https://www.docker.com)
[![License](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)

[Features](#-features) · [Quick Start](#-quick-start) · [API Reference](#-api-reference) · [Architecture](#-architecture) · [Local Dev](#-local-development)

</div>

---

## 📸 Overview

RecipeVault is a **three-tier web application** for managing recipes with full CRUD operations, real-time search, and category filtering. Built with a NoSQL document store (MongoDB) and served through an Nginx reverse proxy, it runs entirely in Docker with a single command.

```
Browser → Nginx (port 80) → Express API (port 5000) → MongoDB (port 27017)
```

---

## ✨ Features

| Feature | Details |
|---|---|
| 🔍 **Real-time search** | Searches across name, ingredients, and description simultaneously |
| 🗂️ **Category filtering** | Breakfast · Lunch · Dinner · Dessert · Snack · Drink |
| 📋 **Full CRUD** | Create, read, update, and delete recipes via REST API |
| 🔒 **Rate limiting** | 100 requests / 15 min per IP — protects against abuse |
| 🌐 **CORS control** | Configurable allowed origins via environment variable |
| 🛡️ **Input sanitisation** | Allowlist validation + regex injection prevention |
| 🐳 **One-command deploy** | Fully Dockerized — frontend, backend, and database |
| 💾 **Persistent storage** | MongoDB data survives container restarts via named volume |
| 🏥 **Health check** | `/health` endpoint used by Docker and load balancers |

---

## 🛠️ Tech Stack

| Layer | Technology | Version |
|---|---|---|
| Frontend | HTML · CSS · Vanilla JavaScript | — |
| Web Server | Nginx (reverse proxy + static files) | latest |
| Backend | Node.js · Express | ≥18 · 4.x |
| Database | MongoDB (NoSQL) | 6 |
| ODM | Mongoose | 7.x |
| Containerisation | Docker · Docker Compose | — |
| Diagrams | PlantUML | — |

---

## 🚀 Quick Start

> The fastest path — everything runs in Docker, nothing to install except Docker itself.

### Prerequisites

- [Docker Desktop](https://docs.docker.com/get-docker/) (includes Docker Compose)

### 1 — Clone

```bash
git clone https://github.com/your-username/recipevault.git
cd recipevault
```

### 2 — Configure environment

```bash
cp .env.example .env
# Defaults work out of the box — no edits needed for local Docker use
```

<details>
<summary>What's in <code>.env</code>?</summary>

```env
MONGO_URI=mongodb://localhost:27017/recipevault   # overridden by Docker Compose
PORT=5000                                          # internal Express port
BACKEND_PORT=5001                                  # host port exposed by Docker
CORS_ORIGIN=                                       # leave empty for dev; set in prod
```

</details>

### 3 — Start containers

```bash
cd docker
docker-compose up -d
```

This starts three containers:

| Container | Role | Host Port |
|---|---|---|
| `recipevault-frontend` | Nginx — serves UI + proxies `/api` to backend | `80` |
| `recipevault-api` | Express — REST API | `5001` |
| `recipevault-mongo` | MongoDB 6 | `27017` |

The backend waits for MongoDB to pass its health check before starting.

### 4 — Seed sample data (optional)

```bash
docker exec recipevault-api node seed.js
```

This loads 4 sample Middle-Eastern recipes (Shakshuka, Koshari, Om Ali, Falafel Wrap). Safe to re-run — skips if data already exists.

### 5 — Open the app

| | URL |
|---|---|
| 🌐 **Frontend** | http://localhost |
| 📡 **API** | http://localhost:5001/api/recipes |
| 🏥 **Health** | http://localhost:5001/health |

---

## 📡 API Reference

**Base URL:** `http://localhost:5001/api`

### Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/recipes` | List all recipes |
| `GET` | `/recipes?search=pasta` | Search name · ingredients · description |
| `GET` | `/recipes?category=Dinner` | Filter by category |
| `GET` | `/recipes?search=egg&category=Breakfast` | Combine filters |
| `GET` | `/recipes/:id` | Get a single recipe by ID |
| `POST` | `/recipes` | Create a new recipe |
| `PUT` | `/recipes/:id` | Update a recipe |
| `DELETE` | `/recipes/:id` | Delete a recipe |
| `GET` | `/health` | Health check |

### Request Body (POST / PUT)

```json
{
  "name": "Classic Pancakes",
  "category": "Breakfast",
  "description": "Fluffy pancakes with a golden crust.",
  "prepTime": 10,
  "cookTime": 15,
  "servings": 4,
  "difficulty": "Easy",
  "ingredients": "2 cups flour\n1 cup milk\n2 eggs\n2 tbsp butter",
  "instructions": "Mix dry ingredients.\nAdd wet ingredients.\nCook on medium heat."
}
```

### Response (201 Created)

```json
{
  "_id": "664f1a2b3c4d5e6f7a8b9c0d",
  "name": "Classic Pancakes",
  "category": "Breakfast",
  "description": "Fluffy pancakes with a golden crust.",
  "prepTime": 10,
  "cookTime": 15,
  "servings": 4,
  "difficulty": "Easy",
  "ingredients": "2 cups flour\n1 cup milk\n2 eggs\n2 tbsp butter",
  "instructions": "Mix dry ingredients.\nAdd wet ingredients.\nCook on medium heat.",
  "createdAt": "2025-05-04T20:25:00.000Z",
  "updatedAt": "2025-05-04T20:25:00.000Z"
}
```

### Field Constraints

| Field | Type | Rules |
|---|---|---|
| `name` | String | Required · max 120 chars |
| `category` | String | Required · one of: `Breakfast` `Lunch` `Dinner` `Dessert` `Snack` `Drink` |
| `description` | String | Optional · max 500 chars |
| `prepTime` | Number | Minutes · min 0 |
| `cookTime` | Number | Minutes · min 0 |
| `servings` | Number | min 1 |
| `difficulty` | String | `Easy` · `Medium` · `Hard` |
| `ingredients` | String | Newline-separated list |
| `instructions` | String | Newline-separated steps |

### Error Responses

| Status | Cause |
|---|---|
| `400` | Validation error or invalid ObjectId |
| `404` | Recipe not found |
| `429` | Rate limit exceeded (100 req / 15 min) |
| `500` | Internal server error |

---

## 🏗️ Architecture

### Folder Structure

```
recipevault/
├── backend/
│   ├── config/
│   │   └── db.js                  # Mongoose connection — exits on failure (Docker restart)
│   ├── controllers/
│   │   └── recipeController.js    # CRUD handlers — MVC pattern
│   ├── middleware/
│   │   └── errorHandler.js        # Centralised error handling (Mongoose + generic)
│   ├── models/
│   │   └── Recipe.js              # Schema with text index (name×10, ingredients×5, desc×1)
│   ├── routes/
│   │   └── recipes.js             # Express Router — maps verbs → controller functions
│   ├── Dockerfile
│   ├── package.json
│   ├── seed.js                    # Sample data loader (idempotent)
│   └── server.js                  # Entry point — CORS, rate limit, routes, error handler
│
├── frontend/
│   ├── app.js                     # Fetch-based API client + DOM logic
│   ├── index.html
│   ├── style.css
│   └── nginx.conf                 # Static files + /api reverse proxy to backend:5000
│
├── docker/
│   ├── docker-compose.yml         # Three-service stack with health check
│   └── Dockerfile                 # Frontend image (Nginx + static files)
│
├── Digrams/                       # PlantUML source + rendered PNGs
│   ├── Arch/                      # System architecture
│   ├── Component/                 # Component diagram
│   ├── Datamodeling/              # Data model
│   ├── Deployment/                # Docker deployment topology
│   ├── Sequance/                  # Sequence diagrams (create & view flows)
│   └── UseCase/                   # Use case diagram
│
├── .env.example
└── README.md
```

### Request Flow

```
User Action (browser)
      │
      ▼
 Nginx :80
  ├─ GET /          → serves index.html + static assets
  └─ /api/*         → proxy_pass → Express :5000
                              │
                              ├─ Rate Limiter (100 req/15min)
                              ├─ CORS check
                              ├─ Body parser
                              ├─ Router → Controller
                              │       └─ Mongoose query → MongoDB :27017
                              └─ Error Handler (Validation / CastError / 500)
```

### MongoDB Data Model

```js
{
  _id:          ObjectId,           // auto-generated
  name:         String,             // required, max 120 chars
  category:     String,             // enum: 6 values
  description:  String,             // max 500 chars
  prepTime:     Number,             // minutes
  cookTime:     Number,             // minutes
  servings:     Number,
  difficulty:   String,             // Easy | Medium | Hard
  ingredients:  String,             // newline-delimited (flat, no sub-docs)
  instructions: String,             // newline-delimited
  createdAt:    Date,               // auto via timestamps: true
  updatedAt:    Date                // auto via timestamps: true
}

// Compound text index for search relevance:
// name (weight 10) > ingredients (weight 5) > description (weight 1)
```

---

## 💻 Local Development

> No Docker needed — run the API directly with Node.js and a local MongoDB.

### Prerequisites

- Node.js ≥ 18
- MongoDB running on `localhost:27017` ([MongoDB Community](https://www.mongodb.com/try/download/community))

### Setup

```bash
# 1. Install backend dependencies
cd backend
npm install

# 2. Set up environment
cp ../.env.example ../.env
# MONGO_URI defaults to mongodb://localhost:27017/recipevault — no change needed

# 3. Start dev server (nodemon — auto-restarts on file changes)
npm run dev
```

```bash
# 4. Optional: seed the database
npm run seed
```

Open `frontend/index.html` in your browser. The frontend auto-detects that it's running from the filesystem (`file://` protocol) and points API calls to `http://localhost:5000/api` automatically.

### Available Scripts

| Command | Description |
|---|---|
| `npm start` | Start production server |
| `npm run dev` | Start dev server with nodemon (hot reload) |
| `npm run seed` | Seed MongoDB with sample recipes |

---

## 🔒 Security Details

| Measure | Implementation |
|---|---|
| **CORS** | Allowlist via `CORS_ORIGIN` env var — blocks unlisted origins in production |
| **Rate limiting** | `express-rate-limit` — 100 req / 15 min per IP, returns `429` |
| **Input sanitisation** | All fields cast to correct types and trimmed before Mongoose validation |
| **Allowlist validation** | `category` and `difficulty` checked against hardcoded arrays before DB write |
| **Regex injection** | Search terms escaped with `str.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')` before `$regex` |
| **ObjectId safety** | Mongoose `CastError` caught and returned as `400` — not `500` |

---

## 🐳 Docker Reference

### Container Summary

| Container | Built from | Exposed port | Depends on |
|---|---|---|---|
| `recipevault-frontend` | `docker/Dockerfile` | `80:80` | `backend` |
| `recipevault-api` | `backend/Dockerfile` | `5001:5000` | `mongo` (healthy) |
| `recipevault-mongo` | `mongo:6` (official) | `27017:27017` | — |

### Useful Commands

```bash
# Start all containers in background
docker-compose up -d

# View logs
docker-compose logs -f backend

# Stop everything
docker-compose down

# Stop and delete volumes (wipes database)
docker-compose down -v

# Rebuild after code changes
docker-compose up -d --build

# Open a shell inside the API container
docker exec -it recipevault-api sh
```

---

## 📊 Diagrams

Architecture and design diagrams (PlantUML source + rendered PNG) are in the `Digrams/` folder:

| Diagram | Description |
|---|---|
| `Arch/` | Overall system architecture |
| `Component/` | Component relationships |
| `Datamodeling/` | MongoDB document model |
| `Deployment/` | Docker container topology |
| `Sequance/` | Request sequence flows |
| `UseCase/` | User interaction use cases |

---

## 📄 License

MIT © 2025 — Built for **CSE323 · NoSQL Databases**

