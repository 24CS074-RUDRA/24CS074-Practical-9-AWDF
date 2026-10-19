# Practical 9 — Taskflow

A full-stack task manager with JWT authentication, user-specific tasks, in-memory caching, MongoDB query optimization, and a responsive React interface.

> **Repository:** [24CS074-RUDRA/24CS074-Practical-9-AWDF](https://github.com/24CS074-RUDRA/24CS074-Practical-9-AWDF)
> **Project directory:** `Prac9_AWDF_24CS074`

## Features

- Create an account and sign in using JWT authentication.
- Store passwords as bcrypt hashes; never store plain-text passwords.
- Create, view, update, filter, and delete your own tasks.
- Cache task queries in memory for 60 seconds using user-scoped keys.
- Invalidate the relevant cache entries after successful task changes.
- Optimize MongoDB reads with compound indexes and Mongoose `.lean()` queries.
- Inspect cache hit/miss counters and bypass caching when benchmarking.
- Build and serve the responsive React frontend from the Express application.

## Requirements

- Node.js 18 or later and npm.
- A MongoDB database, either a local MongoDB server or a MongoDB Atlas cluster.

## Clone and install

Clone the GitHub repository and enter the application directory:

```bash
git clone https://github.com/24CS074-RUDRA/24CS074-Practical-9-AWDF.git
cd 24CS074-Practical-9-AWDF/Prac9_AWDF_24CS074
npm install
npm --prefix frontend install
```

On Windows PowerShell, the same commands work from a PowerShell terminal:

```powershell
git clone https://github.com/24CS074-RUDRA/24CS074-Practical-9-AWDF.git
Set-Location .\24CS074-Practical-9-AWDF\Prac9_AWDF_24CS074
npm install
npm --prefix frontend install
```

## Configure environment

Create your local environment file from the example:

```powershell
Copy-Item .env.example .env
```

On macOS or Linux, use `cp .env.example .env`. Open `.env` and set a MongoDB connection string and a new JWT signing secret:

```env
PORT=5000
MONGO_URI=mongodb://127.0.0.1:27017/taskmanager
JWT_SECRET=replace-this-with-a-long-random-secret
CACHE_ENABLED=true
```

For MongoDB Atlas, replace `MONGO_URI` with your cluster connection string and database name. Ensure the database user and network access are configured in Atlas. Do not commit your `.env` file, real database credentials, or JWT secret.

Generate a strong JWT secret with Node.js:

```bash
node -p "require('crypto').randomBytes(48).toString('hex')"
```

Copy the generated value into the `JWT_SECRET` entry in `.env`. Keep it private. Changing this secret invalidates tokens signed using the previous value.

`CACHE_ENABLED` is optional and defaults to enabled; set it to `false` to disable the in-memory cache during benchmarking.

## Build and run

Build the React frontend, run the automated tests, and start the backend:

```bash
npm --prefix frontend run build
npm test
npm run dev
```

The server connects to MongoDB before it begins listening. When it is ready, open [http://localhost:5000](http://localhost:5000).

To run the React development server with Vite hot reload, start the backend using `npm run dev` in one terminal, then run this in another terminal from the project directory:

```bash
npm --prefix frontend run dev
```

Open [http://localhost:3000](http://localhost:3000). Vite proxies `/api` requests to the backend on port 5000.

## Automated tests

With MongoDB available and `.env` configured, run:

```bash
npm test
```

The verification suite exercises authentication, input validation, task access, user isolation, cache hits and misses, cache invalidation, cache telemetry, and cache bypass.

## API reference

Task routes require a JWT in the `Authorization: Bearer <token>` header.

### Authentication

| Method | Endpoint | Description | Authentication |
| --- | --- | --- | --- |
| `POST` | `/api/auth/register` | Register an account | No |
| `POST` | `/api/auth/login` | Sign in and receive a JWT | No |
| `GET` | `/api/auth/me` | Get the signed-in user's profile | Yes |

### Tasks and cache

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/api/tasks` | List the signed-in user's tasks |
| `GET` | `/api/tasks/:id` | Get one of the signed-in user's tasks |
| `POST` | `/api/tasks` | Create a task |
| `PUT` | `/api/tasks/:id` | Update a task |
| `DELETE` | `/api/tasks/:id` | Delete a task |
| `GET` | `/api/tasks/cache-stats` | Read cache telemetry |
| `POST` | `/api/tasks/cache-stats/reset` | Reset cache telemetry counters |

The task-list and single-task GET routes return an `X-Cache` response header with `HIT` or `MISS`. Send `x-skip-cache: true` to bypass caching for an individual request.

## Caching and database indexes

- Task-list cache keys are scoped to the authenticated user: `tasks:<userId>`.
- Single-task cache keys include both the user and task: `task:<userId>:<taskId>`.
- Cached task reads expire after 60 seconds. Successful task creation, update, and deletion invalidate the affected entries.
- The task schema has an index on `user` and a compound `{ user: 1, createdAt: -1 }` index for a user's newest-first task list.
- Read queries use `.lean()` to return plain objects instead of hydrated Mongoose documents.

Use `GET /api/tasks/cache-stats` to inspect hit, miss, set, and invalidation counts. Reset the counters before a benchmark with `POST /api/tasks/cache-stats/reset`.

## Project layout

```text
Prac9_AWDF_24CS074/
├── frontend/       # React and Vite client
├── middleware/     # Authentication and validation
├── models/         # Mongoose user and task models
├── public/         # Static frontend fallback
├── routes/         # Authentication and task API routes
├── utils/          # In-memory cache
├── server.js       # Express server and MongoDB connection
└── test_practical9.js
```
