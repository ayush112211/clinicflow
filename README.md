# ClinicFlow 🏥

> **Conflict-Free Clinic Front Desk & Appointment Scheduling System**  
> Built to eliminate doctor double-booking, enforce fair 24-hour cancellation rules, and provide sub-second lookups for front desk staff.

---

## 🌟 Product Highlights

- **Mathematical Overlap Prevention**: Prevents doctor double-booking at both the live UI validation stage and transactional database layer using strict interval overlap checking:  
  $$\max(\text{start}_1, \text{start}_2) < \min(\text{end}_1, \text{end}_2)$$
- **Fair 24-Hour Cancellation Policy**:
  - Cancelled with **$\ge 24$ hours notice**: **\$0.00 (Free of charge)**.
  - Cancelled with **$< 24$ hours notice**: Standard **\$25.00 late cancellation fee** assessed to protect doctor calendar utilization.
- **Doctor's Day Schedule View**: Real-time 30-minute daily breakdown (09:00 - 17:00) of any doctor's schedule showing booked visits, free slots, and 1-click booking from empty slots.
- **Instant Lookups, Search, Sort & Pagination**: Search by patient name, phone, or doctor; sort by date, patient, doctor, status, or fee; full pagination with configurable rows per page.
- **Staff Authentication**: Secure registration and login with JWT and bcrypt password hashing, plus 1-click Demo logins for evaluators.
- **One-Page Product Landing Page**: Integrated showcase covering What It Is, Key Features, Target Audience, Operational Impact, and Three Features We Would Build Next.

---

## 🏗️ Architecture & Tech Stack

| Layer | Technology | Rationale |
|---|---|---|
| **Frontend** | React 18, TypeScript, Vite, Tailwind CSS, Lucide Icons | High-performance SPA with instant reactive feedback, live conflict badge, and clean medical UI. |
| **Backend** | Node.js, Express, TypeScript | Decoupled RESTful API with route modularity, strict validation, and error middleware. |
| **Database** | SQLite (`sql.js` WebAssembly engine) | True SQL relational persistence (`clinic.db`) with zero native C++ compiler dependencies, ensuring 100% cross-platform portability across Windows, macOS, and Linux. |
| **Security** | JWT (JSON Web Tokens), `bcryptjs` | Stateless session management with salted password hashing. |
| **Testing** | Node.js Test Runner (`node:test`, `node:assert/strict`), TypeScript | Fast, zero-dependency unit and integration test suite validating edge cases and double-booking rejection. |

---

## 🚀 Quick Start Guide

### Prerequisites
- [Node.js](https://nodejs.org/) v18.0.0 or higher
- `npm` (v9+)

### 1. Clone the Repository
```bash
git clone https://github.com/ap11221/clinicflow.git
cd clinicflow
```

### 2. Install Dependencies
Install dependencies for both backend and frontend:
```bash
# From root directory:
npm run install:all
```
*(Alternatively: `cd server && npm install && cd ../client && npm install`)*

---

## 🏃 Running the Application

### Option A: Production Mode (Unified Single Port)
Build both client and server, then run everything on `http://localhost:5000`:
```bash
# 1. Build server and client
npm run build

# 2. Start unified server
npm start
```
Visit **[http://localhost:5000](http://localhost:5000)** in your browser. The Express server will serve the REST API and the compiled React frontend simultaneously.

### Option B: Development Mode (Hot Reloading)
Open two terminal windows:

**Terminal 1 (Backend API Server on port 5000):**
```bash
cd server
npm run dev
```

**Terminal 2 (Vite Frontend with HMR on port 3000):**
```bash
cd client
npm run dev
```
Visit **[http://localhost:3000](http://localhost:3000)**. Requests to `/api/*` are automatically proxied to port 5000.

---

## 🧪 Running Automated Tests

ClinicFlow includes comprehensive automated tests covering overlap rejection, boundary edge cases, adjacent non-conflicting slots, fair cancellation rules, lookups, and pagination:

```bash
cd server
npm test
```

Expected output:
```
✔ Conflict Engine: Blocks exact duplicate time slots for the same doctor
✔ Conflict Engine: Blocks partial start overlap and end overlap
✔ Conflict Engine: Blocks completely engulfed intervals
✔ Conflict Engine: Allows back-to-back adjacent appointments without conflict
✔ Conflict Engine: Allows different doctors to have appointments at the exact same time
✔ Fair Cancellation Policy: Free cancellation with >24 hours advance notice ($0 fee)
✔ Fair Cancellation Policy: Late cancellation fee applied when <24 hours notice ($25 fee)
✔ Fair Cancellation Policy: Atomic cancellation updates status and logs fee
✔ Search & Lookup: Front desk can find patient and their appointments
✔ Pagination: Returns correct slice and respects page limits
ℹ pass 9
ℹ fail 0
```

---

## 🔑 Demo Credentials

The database is pre-seeded with doctors, patients, and sample appointments across past, today, and future dates. You can sign in using:

| Role | Email | Password | Notes |
|---|---|---|---|
| **Front Desk** | `frontdesk@clinicflow.com` | `frontdesk123` | Full access to booking, daily schedule, directory, cancellations |
| **Administrator** | `admin@clinicflow.com` | `admin123` | System administrator profile |

*(Or click the 1-click **"Quick Demo Login"** buttons inside the Sign In modal).*

---

## 📡 REST API Endpoint Catalog

All API endpoints are prefixed with `/api`. Protected routes accept `Authorization: Bearer <token>`.

### 1. Authentication
| Method | Endpoint | Description | Request Body |
|---|---|---|---|
| `POST` | `/api/auth/register` | Register a new staff user | `{"name": "...", "email": "...", "password": "...", "role": "front_desk" \| "admin"}` |
| `POST` | `/api/auth/login` | Authenticate staff credentials | `{"email": "...", "password": "..."}` |
| `GET` | `/api/auth/me` | Get current user from JWT | *Bearer Token Header* |

### 2. Doctors & Schedules
| Method | Endpoint | Description | Query Parameters |
|---|---|---|---|
| `GET` | `/api/doctors` | List all active clinic doctors | None |
| `GET` | `/api/doctors/:id` | Get doctor details | None |
| `GET` | `/api/doctors/:id/day` | **Doctor's Day View**: Returns all 30m slots for a day with booked/available status | `?date=YYYY-MM-DD` |

### 3. Patients
| Method | Endpoint | Description | Query / Body |
|---|---|---|---|
| `GET` | `/api/patients` | Search & list patients | `?search=Alice` |
| `POST` | `/api/patients` | Register a new patient | `{"full_name": "...", "phone": "...", "email": "...", "notes": "..."}` |
| `GET` | `/api/patients/:id` | Get patient profile & appointment history | None |

### 4. Appointments & Core Scheduling
| Method | Endpoint | Description | Details |
|---|---|---|---|
| `GET` | `/api/appointments` | Search, sort, filter & paginate appointments | `?page=1&limit=10&search=...&sortBy=start_time&sortOrder=asc&doctorId=...&status=...&date=...` |
| `POST` | `/api/appointments` | **Book Appointment**: Enforces atomic double-booking guard. Returns HTTP 409 on conflict. | `{"doctorId": 1, "patientId": 2, "startTime": "...", "endTime": "...", "reason": "..."}` |
| `POST` | `/api/appointments/check-conflict` | Pre-flight live validation before submission | `{"doctorId": 1, "startTime": "...", "endTime": "..."}` |
| `GET` | `/api/appointments/:id/cancel-preview` | Previews elapsed hours and fee (\$0 vs \$25) | None |
| `POST` | `/api/appointments/:id/cancel` | **Cancel Appointment**: Applies fair 24h cancellation rule and logs audit event | `{"reason": "..."}` |
| `PATCH` | `/api/appointments/:id/status` | Update appointment status (`COMPLETED`, `NO_SHOW`, `BOOKED`) | `{"status": "COMPLETED"}` |

### 5. Statistics & Metrics
| Method | Endpoint | Description | Response |
|---|---|---|---|
| `GET` | `/api/stats` | High-level front desk operational overview | `{ todayCount, activeBooked, totalCancelled, totalFees, totalDoctors, conflictsPrevented }` |

---

## 🛠️ Debugging & Troubleshooting

### 1. Database Inspection
The database is stored in `clinic.db` in the server root. To reset or re-seed the database at any time:
```bash
cd server
npm run seed
```

### 2. Testing Double-Booking Conflict Manually via cURL
Try booking an appointment in an already occupied time slot:
```bash
curl -X POST http://localhost:5000/api/appointments \
  -H "Content-Type: application/json" \
  -d '{"doctorId": 1, "patientId": 2, "startTime": "2026-09-17T09:00:00.000Z", "endTime": "2026-09-17T09:30:00.000Z", "reason": "Conflict Test"}'
```
Response:
```json
{
  "error": "Doctor is already booked for appointment with Alice Johnson from 09:00 am to 09:30 am.",
  "conflict": true
}
```

### 3. Port Conflicts
If port 5000 is occupied by another service:
```bash
PORT=5001 npm start
```

---

## 📄 License
MIT License. Created by **ap11221**.
