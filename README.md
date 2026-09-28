# 🏥 Hospital Appointment Scheduling System

A complete full-stack **Hospital Appointment Scheduling System** built with **React (Vite)** on the frontend and **Django REST Framework + MySQL** on the backend. Patients can search doctors, view real-time availability and book appointments; doctors manage their schedules and appointments; admins manage the entire hospital.

---

## ✨ Features

### Everyone (Public)
- Hospital landing page with hero, departments, services and call-to-action
- Browse & search doctors (name / specialization), filter by department & availability
- View doctor profile details (qualification, experience, fee, bio)
- Patient registration and login

### Patient
- Personal dashboard (upcoming / total / confirmed / completed / cancelled)
- Real-time **dynamic slot generation** per doctor & date (no hardcoded slots)
- Book appointments with double-booking prevention (backend + DB constraint)
- Cancel own appointments, view own appointment history, update own profile

### Doctor
- Dashboard with today's / pending / confirmed / completed appointments
- Create & manage own weekly schedule (slot duration, availability toggle)
- Accept, reject and complete appointments assigned to them
- View patient details for their appointments; update own profile

### Admin
- Dashboard with hospital-wide statistics
- Full CRUD for **Doctors** (creates login accounts), **Departments**, **Patients** (view), **Schedules**, **Appointments**
- Change appointment status and manage every record

### Cross cutting
- JWT authentication (Simple JWT) with refresh tokens and logout blacklisting
- Role-based permissions enforced on the **backend** (not just the UI)
- MySQL database with proper relationships, `UniqueConstraint`s and indexes
- Django admin configured for all models
- Search, filters, pagination, loading & empty states, friendly error messages
- Postman-compatible REST API (documented in `API_DOCUMENTATION.md`)

---

## 🧰 Technology Stack

| Layer     | Technology                                           |
| --------- | ---------------------------------------------------- |
| Frontend  | React 18, Vite, React Router DOM, Axios, Bootstrap 5 |
| Backend   | Python, Django, Django REST Framework, Simple JWT    |
| Database  | MySQL 8                                              |
| Testing   | Django TestCase + Django REST Framework APITestCase  |
| Version   | Git / GitHub                                         |

---

## 📁 Folder Structure

```
hospital-appointment-system/
│
├── backend/
│   ├── manage.py
│   ├── requirements.txt
│   ├── .env.example
│   ├── hospital_backend/           # project: settings, urls, wsgi, asgi
│   ├── accounts/                   # custom User, JWT auth, roles
│   ├── core/                       # shared permissions, pagination, utils
│   ├── departments/                # department CRUD
│   ├── doctors/                    # doctors, search/filter, available slots
│   ├── patients/                   # patients & profiles
│   ├── schedules/                  # doctor schedules + slot generation
│   └── appointments/               # bookings, business rules, status flow
│
├── frontend/
│   ├── package.json
│   ├── vite.config.js
│   ├── index.html
│   ├── .env.example
│   └── src/
│       ├── components/             # Navbar, Sidebar, Cards, Badges, Loading...
│       ├── pages/                  # public + patient + doctor + admin pages
│       ├── services/               # axios API service modules
│       ├── context/                # AuthContext (JWT state)
│       ├── hooks/                  # useFetch
│       ├── utils/                  # formatters, constants, error helpers
│       ├── App.jsx
│       ├── main.jsx
│       └── index.css
│
├── README.md
├── API_DOCUMENTATION.md
└── .gitignore
```

---

## 🚀 Prerequisites

- **Python 3.10+**
- **Node.js 18+**
- **MySQL 8.0** running locally
- (Optional) MySQL client for creating the database

---

## 🗄️ MySQL Setup

1. Open a terminal / MySQL Workbench / MySQL Shell and connect as root:

   ```sql
   CREATE DATABASE hospital_appointment_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```

2. Your database credentials go into `backend/.env` (never committed).

---

## ⚙️ Backend Setup

```bash
cd backend

# 1. Create a virtual environment
python -m venv env

# 2. Activate it
# Windows:
env\Scripts\activate
# macOS / Linux:
source env/bin/activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure environment variables
copy .env.example .env        # Windows
# cp .env.example .env        # macOS / Linux
#    -> then edit .env and set DB_USER / DB_PASSWORD (+ SECRET_KEY)

# 5. Run migrations
python manage.py makemigrations
python manage.py migrate

# 6. (Optional) seed demo data — creates departments + demo accounts
python manage.py seed_data

# 7. Run the API server
python manage.py runserver
```

Backend runs at **http://127.0.0.1:8000/** (API root under `/api/`).

---

## 🎨 Frontend Setup

```bash
cd frontend

npm install

copy .env.example .env        # Windows
# cp .env.example .env        # macOS / Linux

npm run dev
```

Frontend runs at **http://localhost:5173/**.

---

## 🔐 Environment Variables

### `backend/.env`

```env
SECRET_KEY=change-me-to-a-long-random-string
DEBUG=True
ALLOWED_HOSTS=127.0.0.1,localhost

DB_NAME=hospital_appointment_db
DB_USER=root
DB_PASSWORD=your_mysql_password
DB_HOST=127.0.0.1
DB_PORT=3306

CORS_ALLOWED_ORIGINS=http://localhost:5173,http://127.0.0.1:5173
```

### `frontend/.env`

```env
VITE_API_BASE_URL=http://127.0.0.1:8000/api
```

---

## 👤 Demo Accounts

Run `python manage.py seed_data` once, then use:

| Role    | Username | Password    |
| ------- | -------- | ----------- |
| Admin   | `admin`  | `admin123`  |
| Doctor  | `drarun` | `doctor123` |
| Patient | `patient`| `patient123`|

> `Dr. Arun Kumar` is seeded with a **Monday–Friday 09:00–13:00 (30 min)** schedule so you can immediately test slot booking.

---

## 📡 API Information

Full endpoint documentation with request/response examples: **[`API_DOCUMENTATION.md`](./API_DOCUMENTATION.md)**

Quick reference:

```
POST  /api/auth/register/            Register a patient account
POST  /api/auth/login/               Login (access + refresh tokens)
POST  /api/auth/logout/              Logout (blacklists refresh token)
GET   /api/auth/me/                  Current user + role profile
POST  /api/auth/token/refresh/       Refresh access token

GET   /api/departments/              List departments (public)
POST  /api/departments/              Create department (admin)
GET/PUT/DELETE /api/departments/<id>  Manage department (admin write)

GET   /api/doctors/                  List/search/filter doctors (public)
POST  /api/doctors/                  Create doctor account (admin)
GET   /api/doctors/me/               Doctor self profile
GET/PUT/DELETE /api/doctors/<id>     Doctor detail / admin manage
GET   /api/doctors/<id>/available-slots/?date=YYYY-MM-DD   Dynamic slots

GET   /api/patients/                 List patients (admin)
GET   /api/patients/<id>/            Patient detail (admin/owner)
GET/PUT /api/patients/me/            Patient own profile

GET/POST /api/schedules/             List/Create schedules (doctor own / admin all)
GET/PUT/DELETE /api/schedules/<id>   Manage schedule

POST  /api/appointments/             Book appointment (patient)
GET   /api/appointments/             All appointments (admin)
GET/PUT/DELETE /api/appointments/<id> Detail / status change / delete
GET   /api/patient/appointments/     Patient's own appointments
GET   /api/doctor/appointments/      Doctor's own appointments
GET   /api/admin/appointments/       All appointments (admin)
```

---

## 🧪 Tests

```bash
cd backend
python manage.py test
```

53 tests covering registration, login, JWT, permissions, department/doctor/schedule CRUD, dynamic slots, instrumented booking rules (double-booking, past-date, availability), role-based access control and status transitions.

---

## 🧑‍🤝‍🧑 User Roles

| Capability                     | Patient | Doctor | Admin |
| ------------------------------ | :-----: | :----: | :---: |
| Browse doctors / departments   | ✅      | ✅     | ✅    |
| Book / cancel own appointments | ✅      | —      | —     |
| Set + manage own schedule      | —       | ✅     | ✅    |
| Accept / reject / complete own appointments | — | ✅ | ✅ |
| Manage doctors                  | —       | —      | ✅    |
| Manage departments / all patients / all appointments | — | — | ✅ |

---

## 🖼️ Screenshots

> _Add screenshots of the Home page, Doctor search, Booking flow, doctor/admin dashboards here when presenting the portfolio._

---

## 🔮 Future Improvements

- Email / SMS appointment reminders
- Payment integration for consultation fees (Razorpay/Stripe)
- Video consultation links for telemedicine
- Prescription management and records module
- Audit logs for admin actions
- Role-specific rate limiting & refresh-token rotation UI handling

---

## 📝 License

Educational / portfolio project. Feel free to reuse and extend.