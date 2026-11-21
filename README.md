# Personal Finance Manager (Ledgerly)

Track income, expenses, budgets and savings goals. Built with **React + Vite + Tailwind** (frontend), **Node.js + Express + Prisma** (backend) and **PostgreSQL** (database, started with Docker).

---

## Beginner Run Guide (Windows)

### Step 1: Install two programs (once)

1. **Node.js (LTS version)** from https://nodejs.org. Run the installer and keep clicking Next.
2. **Docker Desktop** from https://www.docker.com/products/docker-desktop. Install it, restart your PC if asked, then **open Docker Desktop and wait until it says it is running**.

Now open **PowerShell** (press the Windows key, type `powershell`, press Enter) and check both installs:

```powershell
node -v
npm -v
docker --version
```

Each command should print a version number. If one says "not recognized", close PowerShell, open it again, and retry. If it still fails, reinstall that program.

### Step 2: Unzip the project

Unzip `finance-manager.zip` somewhere easy, for example `C:\Projects\finance-manager`.

### Step 3: Start the database (Terminal 1)

In PowerShell, go to the project folder and start PostgreSQL:

```powershell
cd C:\Projects\finance-manager
docker compose up -d
```

The first time it downloads the database image (about a minute). You can check it with `docker ps`: you should see a container called `finance_db`.

### Step 4: Start the backend (Terminal 2)

Open a **new** PowerShell window:

```powershell
cd C:\Projects\finance-manager\server
npm install
npm run setup
npm run dev
```

- `npm install` downloads the packages.
- `npm run setup` creates the database tables and adds sample data.
- `npm run dev` starts the API. You should see `API running at http://localhost:5000/api`.

**Leave this window open.**

### Step 5: Start the frontend (Terminal 3)

Open **another new** PowerShell window:

```powershell
cd C:\Projects\finance-manager\client
npm install
npm run dev
```

You should see `Local: http://localhost:5173/`. **Leave this window open too.**

### Step 6: Open the app

Go to **http://localhost:5173** in your browser. You will see a dashboard filled with sample data.

### Next time you want to use it

1. Open Docker Desktop (wait until it runs).
2. Terminal 1: `docker compose up -d` in the project folder.
3. Terminal 2: `cd server` then `npm run dev`.
4. Terminal 3: `cd client` then `npm run dev`.

To stop: press `Ctrl + C` in the two terminals. To stop the database: `docker compose down` (your data is kept).

### The 3 most common errors

**1. "Can't reach database server at localhost:5432" (Prisma error P1001)**
The database is not running. Open Docker Desktop, wait until it is running, then run `docker compose up -d` in the project folder and try again. If you already have PostgreSQL installed on your PC, it may be using port 5432. Either stop it, or change `5432:5432` to `5433:5432` in `docker-compose.yml` and use `localhost:5433` in `server/.env`.

**2. "npm is not recognized" or "running scripts is disabled on this system"**
- "Not recognized": close and reopen PowerShell. If it persists, reinstall Node.js.
- "Scripts is disabled": run this once, then retry:
  `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned`

**3. The page loads but shows "Cannot reach the server" (or "port already in use")**
The backend (Terminal 2) is not running or crashed. Look at that window for the error, fix it, and run `npm run dev` again. If it says the port is in use, close the other program using it or change `PORT` in `server/.env` and `VITE_API_URL` in `client/.env` (restart both after editing).

---

## How the project is organized

The project has three parts that talk to each other. The **database** (PostgreSQL, running in Docker) stores everything. The **server** folder is the backend: an Express API that receives requests like "give me this month's transactions", checks the data with Zod, uses Prisma to read or write the database, and sends back JSON. Inside it, `routes` match URLs, `controllers` handle the request and response, and `services` hold the real logic (like adding up spending per category). The **client** folder is the React app that runs in your browser: `pages` are the screens (Dashboard, Transactions, Budgets...), `components` are reusable pieces (buttons, modals, forms), and `services/api.js` is the single place that talks to the backend. The Prisma file `server/prisma/schema.prisma` defines the tables, and `server/prisma/seed.js` fills them with starter data.

```
finance-manager/
  docker-compose.yml        starts PostgreSQL
  server/
    prisma/schema.prisma    database tables
    prisma/seed.js          default categories + sample data
    src/routes/             URL -> controller
    src/controllers/        request/response handling
    src/services/           business logic + database queries
    src/validators/         Zod input rules
    src/middleware/         validation + central error handling
  client/
    src/pages/              one file per screen
    src/components/         reusable UI pieces
    src/services/api.js     all backend calls
    src/hooks/useFetch.js   loading / error / data helper
```

## Features

- **Dashboard:** total balance, this month's income and expenses, budget usage, last 5 transactions, spending by category, 6-month trend
- **Transactions:** add, edit, delete, filter by type / category / date range, search notes, 10 per page
- **Categories:** defaults plus custom ones with colors. Deleting a category that has transactions shows a warning and moves them to "Other"
- **Budgets:** per category per month. Green under 80%, orange 80-100%, red over budget. Each month starts fresh (with a "Copy last month" shortcut)
- **Reports:** monthly summary, savings rate, comparison with last month, top category, category breakdown, CSV export
- **Savings goals:** contributions with history, progress, days remaining, mark completed, multiple goals

## API reference

All responses look like `{ "success": true, "data": ... }`. Errors look like `{ "success": false, "message": "..." }`.

| Area | Endpoints |
| --- | --- |
| Transactions | `GET /api/transactions?type=&categoryId=&startDate=&endDate=&search=&page=&limit=` , `POST` , `PUT /:id` , `DELETE /:id` |
| Categories | `GET /api/categories?type=` , `POST` , `PUT /:id` , `DELETE /:id?force=true` |
| Budgets | `GET /api/budgets?month=&year=` , `GET /summary` , `POST` (updates if it already exists) , `PUT /:id` , `DELETE /:id` |
| Goals | `GET /api/goals` , `POST` , `PUT /:id` , `DELETE /:id` , `POST /:id/contribute` |
| Reports | `GET /api/reports/monthly` , `/categories` , `/trend` , `/balance` , `/export` (CSV) |

Dates are `YYYY-MM-DD`. Types are `INCOME` or `EXPENSE`.

## Useful commands

```powershell
cd server
npm run db:studio     # opens a visual database browser
npm run db:seed       # re-adds default categories (never duplicates)
```

To start with a completely empty database: `npx prisma migrate reset` (this deletes all data, then re-runs the seed).

To show another currency, change `CURRENCY` in `client/src/utils/format.js` (for example `'NPR'` or `'EUR'`).

## Ideas for deploying later

- Database + backend: Render, Railway or Fly.io (set `DATABASE_URL`, `CLIENT_URL`; start command `npx prisma migrate deploy && node src/server.js`)
- Frontend: Vercel or Netlify (build command `npm run build`, output folder `dist`, set `VITE_API_URL` to your backend URL)
- This app has no login, so anyone with the link can see and change the data. Add authentication before putting real data online.

## Not included yet

The `recurring_transactions` table exists in the database but has no screen or automatic job yet. Login/authentication is also not included.
