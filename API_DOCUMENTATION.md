# 📡 API Documentation — Hospital Appointment Scheduling System

Base URL (dev): `http://127.0.0.1:8000/api`

All write/private endpoints require the JWT header unless noted:

```
Authorization: Bearer <access_token>
```

Roles: `ADMIN`, `DOCTOR`, `PATIENT`. Error responses use HTTP status codes and a JSON body. Most validation errors look like:

```json
{ "detail": "Human friendly message." }
```

or field-level:

```json
{ "appointment_time": ["This field is required."] }
```

---

## 1. Authentication

### POST `/api/auth/register/` — Public
Register a **patient** account (creates `User` with role `PATIENT` + `Patient` profile).

**Request:**
```json
{
  "username": "john_doe",
  "email": "john@example.com",
  "password": "StrongPass123",
  "confirm_password": "StrongPass123",
  "full_name": "John Doe",
  "phone": "9876500001",
  "date_of_birth": "1995-06-15",
  "gender": "MALE",
  "address": "123 Main Street, Chennai"
}
```

**Response `201`:**
```json
{
  "id": 7,
  "username": "john_doe",
  "email": "john@example.com",
  "role": "PATIENT",
  "is_active": true,
  "access_token": "eyJhbGciOiJ...",
  "refresh_token": "eyJhbGciOiJ..."
}
```

**Errors:** `400` — `{"username": ["This username is already taken."]}`, `{"confirm_password": ["Passwords do not match."]}`, `{"password": [...]}`.

---

### POST `/api/auth/login/` — Public
Login with **username or email** + password.

**Request:**
```json
{ "username": "john_doe", "password": "StrongPass123" }
```

**Response `200`:**
```json
{
  "id": 7,
  "username": "john_doe",
  "email": "john@example.com",
  "role": "PATIENT",
  "is_active": true,
  "access_token": "eyJ...",
  "refresh_token": "eyJ..."
}
```

**Errors:** `400` — `{"detail": "Invalid email or password."}`, `{"detail": "Your account has been deactivated."}`.

---

### POST `/api/auth/logout/` — Bearer Token
Blacklists the refresh token.

**Request:** `{ "refresh": "eyJ..." }` → **Response `200`** `{"detail": "Successfully logged out."}`

---

### GET `/api/auth/me/` — Bearer Token
Returns the current user + role profile.

**Response `200` (patient):**
```json
{
  "id": 7,
  "username": "john_doe",
  "email": "john@example.com",
  "role": "PATIENT",
  "is_active": true,
  "profile": { "id": 4, "patient_id": "P00001", "full_name": "John Doe", "phone": "9876500001", "gender": "MALE", "address": "123 Main Street, Chennai", "profile_image": null, "date_of_birth": "1995-06-15" }
}
```

**Errors:** `401` — missing/invalid token.

---

### POST `/api/auth/token/refresh/` (alias `/api/auth/refresh/`) — Public
**Request:** `{ "refresh": "eyJ..." }` → **Response `200`** `{ "access": "eyJ..." }`
**Errors:** `401` — expired/blacklisted refresh token (Simple JWT standard error).

---

## 2. Departments

### GET `/api/departments/` — Public
**Response `200`** (list):
```json
[
  { "id": 1, "name": "Cardiology", "description": "Diagnosis ...", "doctor_count": 1 }
]
```

### POST `/api/departments/` — Admin
**Request:** `{ "name": "Oncology", "description": "Cancer care" }`
**Response `201`** — created department. **Errors:** `400` duplicate name, `403` non-admin.

### GET / PUT / DELETE `/api/departments/<id>/`
- `GET` public → department object
- `PUT` admin → update (partial updates allowed)
- `DELETE` admin → `204` (or `200` `{"detail": ...}`). **Errors:** `403` non-admin, `404` not found.

---

## 3. Doctors

### GET `/api/doctors/` — Public
Supports **search, filter, pagination**.

| Query param       | Description                                    |
| ----------------- | ---------------------------------------------- |
| `search`          | name / specialization / qualification / bio    |
| `department`      | department id                                  |
| `specialization`  | exact specialization                           |
| `availability`    | `1` / `true` → only doctors with active schedules |
| `active`          | `1` / `true` → only active doctors             |
| `page`            | page number (default 1)                        |
| `page_size`       | per page (default 10, max 100)                 |

**Examples:**
```
/api/doctors/?search=arun
/api/doctors/?department=1
/api/doctors/?specialization=Cardiology
/api/doctors/?availability=true&page=1
```

**Response `200`:**
```json
{
  "count": 1,
  "total_pages": 1,
  "current_page": 1,
  "next": false,
  "previous": false,
  "results": [
    {
      "id": 1,
      "doctor_id": "DOC00001",
      "full_name": "Dr. Arun Kumar",
      "phone": "9876543210",
      "specialization": "Cardiology",
      "department": { "id": 1, "name": "Cardiology" },
      "qualification": "MBBS, MD (Cardiology)",
      "experience": 12,
      "consultation_fee": "800.00",
      "profile_image": null,
      "bio": "Senior cardiologist ...",
      "is_active": true
    }
  ]
}
```

### POST `/api/doctors/` — Admin
Creates a **Doctor login account** + profile. **Response `201`** (doctor object).

**Request:**
```json
{
  "username": "dr_new",
  "email": "new@hospital.com",
  "password": "DoctorPass123",
  "full_name": "Dr. New",
  "phone": "98700...",
  "specialization": "Dermatology",
  "department": 3,
  "qualification": "MBBS, MD",
  "experience": 5,
  "consultation_fee": "400.00",
  "bio": "Interested in clinical dermatology."
}
```

**Errors:** `400` duplicate username/email / missing password, `403` non-admin.

### GET `/api/doctors/me/` — Doctor
Returns the current doctor's profile (self). **PUT** updates it (doctor may change their own name/specialization/fee/bio/phone/image/username/email/password). Supports `multipart/form-data` for image upload.

### GET / PUT / DELETE `/api/doctors/<id>/`
- `GET` public → doctor detail
- `PUT` admin → update doctor profile/account
- `DELETE` admin → **soft delete** (deactivates the account, keeps history). **Response `200`** `{"detail": "Doctor ... has been deactivated."}`

### GET `/api/doctors/<id>/available-slots/?date=YYYY-MM-DD` — Public
Dynamically generates slots from the doctor's schedule **for that exact weekday**. Past times today are excluded; booked slots (PENDING/CONFIRMED/COMPLETED) are excluded.

**Response `200`:**
```json
{
  "doctor": 1,
  "date": "2026-09-28",
  "available_slots": ["09:00", "09:30", "10:00", "10:30", "11:00", "11:30", "12:00", "12:30"],
  "all_slots": [
    { "time": "09:00", "booked": false },
    { "time": "09:30", "booked": true },
    ...
  ]
}
```
**Errors:** `400` missing date; `404` doctor not found.

---

## 4. Patients

### GET `/api/patients/` — Admin
Paginated list with `search` (name / username / email) and `page` / `page_size`.
**Response `200`** envelope as with doctors → `results` contain:
```json
{
  "id": 4,
  "patient_id": "P00001",
  "user": { "id": 6, "username": "john_doe", "email": "john@example.com", "role": "PATIENT" },
  "full_name": "John Doe",
  "date_of_birth": "1995-06-15",
  "gender": "MALE",
  "phone": "9876500001",
  "address": "123 Main Street, Chennai",
  "profile_image": null
}
```

### GET `/api/patients/<id>/`
- Admin: any patient. Patient: only **themselves** (`403` otherwise).

### GET / PUT `/api/patients/me/` — Patient
Self profile. `PUT` accepts JSON or `multipart/form-data`; fields: `full_name`, `phone`, `date_of_birth`, `gender`, `address`, `profile_image`, `email`.

---

## 5. Schedules

### GET /api/schedules/ — Doctor or Admin
- Doctor → only their own schedules. Admin → all (filter with `?doctor=<id>`).

**Response `200`** envelope; a schedule:
```json
{
  "id": 1,
  "doctor": 1,
  "doctor_name": "Dr. Arun Kumar",
  "doctor_id_code": "DOC00001",
  "day_of_week": 0,
  "day_name": "Monday",
  "start_time": "09:00:00",
  "end_time": "13:00:00",
  "slot_duration": 30,
  "slot_count": 8,
  "is_available": true
}
```

> `day_of_week`: `0 = Monday ... 6 = Sunday`.

### POST `/api/schedules/` — Doctor (own) or Admin
**Request:**
```json
{
  "doctor": 1,
  "day_of_week": 1,
  "start_time": "09:00:00",
  "end_time": "13:00:00",
  "slot_duration": 30,
  "is_available": true
}
```
**Errors:** `400` `{"end_time": "End time must be after start time."}` / duplicate day for the same doctor (DB constraint); `403` doctor creating a schedule for another doctor.

### GET / PUT / DELETE `/api/schedules/<id>/` — Doctor (own) or Admin
Partial updates supported. Admin can manage any schedule.

---

## 6. Appointments

> **Booking rules (backend enforced):**
> 1. Doctor must be **active** and have an **available schedule** on that weekday.
> 2. Requested time must lie **inside** the schedule and be **aligned to the slot grid**.
> 3. Date/time must not be **in the past**.
> 4. Slot must not be **already booked** — enforced inside a row-locked transaction **and** by a DB `UniqueConstraint(doctor, appointment_date, appointment_time)`.
> 5. Cancelled/rejected appointments free their slot (the booking API reuses the row).

### POST `/api/appointments/` — Patient
**Request:**
```json
{
  "doctor": 1,
  "appointment_date": "2026-09-25",
  "appointment_time": "10:30",
  "reason": "General consultation"
}
```
> The `doctor` key is the readable form; serializers also accept `doctor_id`.

**Response `201`:**
```json
{
  "id": 12,
  "appointment_id": "APT00001",
  "patient": { "id": 4, "patient_id": "P00001", "full_name": "John Doe", "phone": "9876500001", "gender": "MALE", "profile_image": null },
  "doctor": { "id": 1, "doctor_id": "DOC00001", "full_name": "Dr. Arun Kumar", "specialization": "Cardiology", "department": { "id": 1, "name": "Cardiology" }, "consultation_fee": "800.00", "profile_image": null, "is_active": true },
  "appointment_date": "2026-09-25",
  "appointment_time": "10:30:00",
  "reason": "General consultation",
  "status": "PENDING"
}
```

**Common booking errors (`400`):**
```json
{ "detail": "This time slot is already booked." }
{ "detail": "Doctor is not available on the selected date." }
{ "detail": "Doctor is unavailable at the moment." }
{ "detail": "Appointment date cannot be in the past." }
{ "detail": "Requested time is outside the doctor's working hours." }
{ "detail": "Cannot book a time slot that has already passed." }
```

### GET `/api/appointments/` — Admin
All appointments (paginated, searchable). Query: `search`, `status`, `date`.

### GET / PUT / PATCH / DELETE `/api/appointments/<id>/` — Authenticated, owner-scoped
- **Access:** patient → own only; doctor → assigned only; admin → all. Otherwise `403`.
- **PUT/PATCH `{"status": "..."}`** performs a **validated transition**:
  - Patient may only set `CANCELLED` on own appointments.
  - Doctor may `CONFIRMED`/`REJECTED` on PENDING and `COMPLETED` on CONFIRMED (their own).
  - Admin may set any status.
- **DELETE:** patient → cancels (soft); admin → hard delete.

| From       | Available transitions            |
| ---------- | -------------------------------- |
| PENDING    | CONFIRMED, REJECTED, CANCELLED   |
| CONFIRMED  | COMPLETED, CANCELLED             |
| COMPLETED  | — (no transitions)               |

### GET `/api/patient/appointments/` — Patient
Own appointments. Query: `status`, `date`, `page`.

### GET `/api/doctor/appointments/` — Doctor
Assigned appointments. Query: `status`, `date`, `page`.

### GET `/api/admin/appointments/` — Admin
All appointments. Query: `search`, `status`, `date`, `page`.

### Status values
`PENDING`, `CONFIRMED`, `REJECTED`, `CANCELLED`, `COMPLETED`.

---

## 7. Roles & Permissions Summary

| Endpoint                       | Patient | Doctor   | Admin  |
| ------------------------------ | :-----: | :------: | :----: |
| `/auth/*` (login/register)     | ✅      | ✅       | ✅     |
| `/auth/me`                     | ✅      | ✅       | ✅     |
| `GET /departments/`, `/doctors/` | ✅      | ✅       | ✅     |
| Departments write              | ✖️      | ✖️       | ✅     |
| `POST /doctors/` , PUT/DELETE doctor | ✖️ | ✖️   | ✅     |
| `GET /doctors/me`, PUT own profile | ✖️ | ✅      | ✖️     |
| `GET /patients/`               | ✖️      | ✖️       | ✅     |
| `GET /patients/<id>`           | own only| ✖️       | ✅     |
| `GET/PUT /patients/me`         | ✅      | ✖️       | ✖️     |
| `GET /schedules/`, POST/PUT/DELETE | ✖️   | own only | ✅    |
| `POST /appointments/`          | ✅      | ✖️       | ✖️     |
| `GET /appointments/<id>`       | own     | own      | all    |
| Status transitions             | cancel only | own appts | all |
| `GET /admin/appointments/`     | ✖️      | ✖️       | ✅     |

---

## 8. Status Codes

- `200 OK` — success (GET/PUT/PATCH/DELETE)
- `201 Created` — resource created
- `204 No Content` — hard delete (where used)
- `400 Bad Request` — validation / business-rule failure (`detail` message)
- `401 Unauthorized` — missing/expired token
- `403 Forbidden` — authenticated but not allowed
- `404 Not Found` — resource missing
- `500 Internal Server Error` — server problem (never exposed as a Django HTML page to API consumers)