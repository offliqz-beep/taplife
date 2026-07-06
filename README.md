# Smart Card Attendance Management System

**IoT-Based Attendance Management System** — University of Kigali thesis project by MANZI Maurice (June 2026).

Physical RFID smart cards + ESP32 readers record class attendance through a web dashboard.

## Stack

| Layer | Technology |
|-------|------------|
| Frontend | React + Tailwind CSS (Vite) |
| Backend | Node.js + Express |
| Database | PostgreSQL |
| IoT | ESP32 + RC522 RFID (MIFARE cards) |

## Quick start

### 1. Database

Create PostgreSQL database `taplife_db` (password in `.env`):

```sql
CREATE DATABASE taplife_db;
```

Tables are created automatically when the backend starts.

### 2. Backend

```bash
cd backend
npm install
npm run dev
```

API: `http://localhost:5000`

**Default admin** (seeded if none exists): `admin@uok.ac.rw` / `admin123`

**Default IoT device key:** `attendance-device-key-2026`

### 3. Frontend

```bash
cd frontend
npm install
npm run dev
```

App: `http://localhost:3000`

### 4. IoT firmware (NodeMCU ESP8266)

1. Open `iot-firmware/attendance_reader/attendance_reader.ino` in Arduino IDE
2. Install libraries: **Adafruit PN532**, **LiquidCrystal I2C**, **ArduinoJson**
3. Set `WIFI_SSID` and `WIFI_PASSWORD` (any network with internet)
4. For **different networks** (campus reader + home dashboard): set `USE_CLOUD_API = true` and `CLOUD_API_HOST` to your deployed API hostname (see below)
5. For **local testing** on same Wi-Fi: set `USE_CLOUD_API = false` and `LOCAL_API_HOST` to your PC IP
6. Flash to NodeMCU

## Cross-network setup (IoT + dashboard on different Wi-Fi)

Database is already on Neon (cloud). Deploy the **backend API** so both the IoT reader and browser use the same public URL:

```
IoT reader (campus Wi-Fi)  ──HTTPS──►  Render API  ◄──HTTPS──  Dashboard (any network)
                                              │
                                              ▼
                                        Neon PostgreSQL
```

### Deploy backend to Render

1. Push this repo to GitHub
2. Go to [render.com](https://render.com) → **New** → **Blueprint** → connect the repo
3. Set environment variables when prompted:
   - `DATABASE_URL` — your Neon connection string (from `backend/.env`)
   - `JWT_SECRET` — a long random secret
4. After deploy, copy your URL (e.g. `https://taplife-api.onrender.com`)

### Point frontend to cloud API

Edit `frontend/.env`:

```env
VITE_API_URL=https://taplife-api.onrender.com/api
```

Restart frontend: `npm run dev`

### Point IoT firmware to cloud API

In `attendance_reader.ino`:

```cpp
const bool USE_CLOUD_API = true;
const char* CLOUD_API_HOST = "taplife-api.onrender.com";  // your Render hostname
```

Re-flash the NodeMCU. The device only needs internet — not the same network as your PC.

> **Note:** Render free tier sleeps after inactivity; the first card tap may take ~30s to wake the server.

## Workflow

1. **Admin** registers physical cards and links them to students (`/cards`)
2. **Lecturer** creates an attendance session (`/attendance`)
3. **Lecturer** activates the session on an IoT reader (`/devices`)
4. **Student** taps RFID card on ESP32 — reader POSTs to `/api/iot/attendance`
5. **Lecturer/Admin** views reports and absenteeism alerts (`/reports`)

## Test IoT endpoint (curl)

```bash
curl -X POST http://localhost:5000/api/iot/attendance \
  -H "Content-Type: application/json" \
  -H "X-Device-Key: attendance-device-key-2026" \
  -d "{\"card_uid\":\"A1B2C3D4\"}"
```

## Project structure

```
taplife-system/
├── backend/          # Express API
├── frontend/         # React dashboard
├── iot-firmware/     # ESP32 Arduino sketch
└── README.md
```

## Scope

This system covers **attendance only** (thesis §1.7.6). Cafeteria, transport, wallet, and access control are out of scope.
