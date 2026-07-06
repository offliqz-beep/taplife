# Online Database Setup (Recommended: Neon)

For your thesis and real deployments, use a **hosted PostgreSQL** database instead of running PostgreSQL only on your PC.

## Recommended: [Neon](https://neon.tech)

| Why Neon | Details |
|----------|---------|
| Free tier | Enough for development and thesis demo |
| PostgreSQL | Same as your app — no code changes |
| Connection string | One `DATABASE_URL` in `.env` |
| Always online | Backend can run on Render/Railway while DB stays in the cloud |

**Alternatives:** [Supabase](https://supabase.com) (PostgreSQL + extras), [Railway](https://railway.app), [ElephantSQL](https://www.elephantsql.com) (smaller free tier).

---

## Step 1 — Create Neon database

1. Go to [https://neon.tech](https://neon.tech) and sign up (GitHub/Google).
2. Click **New Project** → name it `taplife`.
3. Copy the **connection string** (looks like):
   ```
   postgresql://user:password@ep-xxxx.region.aws.neon.tech/neondb?sslmode=require
   ```

## Step 2 — Configure backend

Edit `backend/.env`:

```env
PORT=5000

# Cloud database (Neon) — use this instead of local DB_HOST/DB_PORT when online
DATABASE_URL=postgresql://user:password@ep-xxxx.region.aws.neon.tech/neondb?sslmode=require

JWT_SECRET=your_long_random_secret_here
JWT_EXPIRE=7d
NODE_ENV=production
```

**Local PostgreSQL** still works if you leave `DATABASE_URL` empty and keep:

```env
DB_HOST=localhost
DB_PORT=5432
DB_NAME=taplife_db
DB_USER=postgres
DB_PASSWORD=your_password
```

## Step 3 — Start backend (creates tables automatically)

```bash
cd backend
npm install
npm run dev
```

On first start, the app runs `database.sql` and migrations on the cloud database.

## Step 4 — Deploy backend online (required for different networks)

Host the API so the NodeMCU can reach it from campus Wi-Fi while you use the dashboard from home:

### Option A — Render Blueprint (recommended)

This repo includes `render.yaml`. After pushing to GitHub:

1. [Render](https://render.com) → **New** → **Blueprint** → select repo
2. Set `DATABASE_URL` (Neon) and `JWT_SECRET`
3. Copy the deployed URL (e.g. `https://taplife-api.onrender.com`)

### Option B — Manual Render web service

- Root directory: `backend`
- Build: `npm install`
- Start: `npm start`
- Health check: `/api/health`

### Firmware (HTTPS)

In `iot-firmware/attendance_reader/attendance_reader.ino`:

```cpp
const bool USE_CLOUD_API = true;
const char* CLOUD_API_HOST = "taplife-api.onrender.com";
```

Re-flash the NodeMCU.

## Step 5 — Frontend

If backend is online, set `frontend/.env`:

```env
VITE_API_URL=https://your-app.onrender.com/api
```

Rebuild: `npm run build`

---

## Security notes

- Never commit `.env` with real passwords to Git.
- Use a strong `JWT_SECRET` in production.
- Neon connection strings include the password — keep them private.

## Default admin (created on first run)

- Email: `admin@uok.ac.rw`
- Password: `admin123`

Change this password after first login in production.
