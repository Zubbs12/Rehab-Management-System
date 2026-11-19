# Drug Rehabilitation Centre Management System

A backend REST API built with **NestJS** for managing users, patients, rooms and patient admissions in a drug rehabilitation centre.
Made for the Advanced Web Technology university project.

---

## 1. Main Features

- User registration and login with **JWT** authentication
- Passwords hashed with **bcrypt** (passwords are never returned in responses)
- Role-based access: **ADMIN** and **STAFF**
- Full **CRUD** for Users, Patients, Rooms and Admissions
- **Database transactions** for admitting and discharging a patient
- Input validation with **DTOs + class-validator** (global `ValidationPipe`)
- Business rules: no duplicate emails, no admission into a full room, no deleting patients/rooms that have active admissions
- Seed script for an initial ADMIN account and fictional sample data

## 2. Technologies Used

| Technology | Purpose |
|---|---|
| NestJS (TypeScript) | Backend framework |
| MySQL | Database |
| TypeORM | ORM (entities, repositories, transactions) |
| JWT (`@nestjs/jwt`) | Authentication tokens |
| bcrypt | Password hashing |
| class-validator / class-transformer | DTO validation |

## 3. Project Structure

```
src/
├── main.ts                  # starts the app, global ValidationPipe
├── app.module.ts            # root module, MySQL connection
├── config.ts                # reads the .env file
├── seed.ts                  # creates admin + sample data
├── auth/                    # register, login, guards, decorator
│   ├── dto/
│   ├── guards/              # jwt-auth.guard.ts, roles.guard.ts
│   └── decorators/          # roles.decorator.ts
├── users/                   # entity, dto, service, controller, module
├── patients/                # entity, dto, service, controller, module
├── rooms/                   # entity, dto, service, controller, module
└── admissions/              # entity, dto, service (transactions), controller, module
```

Entity relationships:
- One **Patient** has many **Admissions**
- One **Room** has many **Admissions**
- Each **Admission** belongs to one Patient and one Room

## 4. Installation

**Requirements:** Node.js 18 or newer, and MySQL (MySQL Server, or XAMPP / WAMP).

```bash
cd rehab-centre
npm install
```

## 5. MySQL Database Configuration

**Step 1 – Create the database.** Open the MySQL console (or phpMyAdmin / MySQL Workbench) and run:

```sql
CREATE DATABASE rehab_centre;
```

Using the terminal:

```bash
mysql -u root -p
```
```sql
CREATE DATABASE rehab_centre;
EXIT;
```

**Step 2 – Set your MySQL details in the `.env` file** (in the project root):

```env
PORT=3000

DB_HOST=localhost
DB_PORT=3306
DB_USERNAME=root
DB_PASSWORD=your_mysql_password
DB_NAME=rehab_centre

JWT_SECRET=rehab_centre_super_secret_key_change_me

ADMIN_NAME=System Admin
ADMIN_EMAIL=admin@rehab.local
ADMIN_PASSWORD=Admin@123
```

> XAMPP users: the default MySQL user is `root` with an **empty** password, so leave `DB_PASSWORD=` empty.

The tables (`users`, `patients`, `rooms`, `admissions`) are created **automatically** when the app (or the seed script) first connects, because TypeORM `synchronize: true` is enabled.

## 6. How to Run the Project

**Step 1 – Create the initial ADMIN account and sample data (run once):**

```bash
npm run seed
```

**Step 2 – Start the server:**

```bash
npm run start:dev
```

The API runs at **http://localhost:3000**

Other useful commands:

```bash
npm run build        # compile the project (checks for TypeScript errors)
npm run start:prod   # run the compiled version (after npm run build)
```

## 7. How to Create the Initial ADMIN Account

Run:

```bash
npm run seed
```

It creates:

| Field | Value |
|---|---|
| Email | `admin@rehab.local` |
| Password | `Admin@123` |
| Role | ADMIN |

You can change these in the `.env` file (`ADMIN_EMAIL`, `ADMIN_PASSWORD`) **before** running the seed.
The seed also adds 3 sample rooms (101, 102, 201) and 3 fictional patients.
It is safe to run more than once – it never creates duplicates.

> Anyone who registers through `POST /auth/register` gets the **STAFF** role. Only an ADMIN can create another ADMIN (through `POST /users`).

## 8. Roles and Permissions

| Module | ADMIN | STAFF |
|---|---|---|
| Auth (register / login) | Public | Public |
| Users | Full access | No access |
| Patients | Full access | Full access |
| Rooms | Full access | View only (GET) |
| Admissions | Full access | Full access |

## 9. API Endpoints

`Auth` column: 🔓 = public, 🔑 = needs `Authorization: Bearer <token>`

| Method | Endpoint | Description | Auth / Role |
|---|---|---|---|
| POST | `/auth/register` | Register a new STAFF user | 🔓 |
| POST | `/auth/login` | Login, returns JWT token | 🔓 |
| POST | `/users` | Create a user | 🔑 ADMIN |
| GET | `/users` | Get all users | 🔑 ADMIN |
| GET | `/users/:id` | Get one user | 🔑 ADMIN |
| PATCH | `/users/:id` | Update a user | 🔑 ADMIN |
| DELETE | `/users/:id` | Delete a user | 🔑 ADMIN |
| POST | `/patients` | Create a patient | 🔑 ADMIN, STAFF |
| GET | `/patients` | Get all patients | 🔑 ADMIN, STAFF |
| GET | `/patients/:id` | Get one patient | 🔑 ADMIN, STAFF |
| PATCH | `/patients/:id` | Update a patient | 🔑 ADMIN, STAFF |
| DELETE | `/patients/:id` | Delete a patient | 🔑 ADMIN, STAFF |
| POST | `/rooms` | Create a room | 🔑 ADMIN |
| GET | `/rooms` | Get all rooms | 🔑 ADMIN, STAFF |
| GET | `/rooms/:id` | Get one room | 🔑 ADMIN, STAFF |
| PATCH | `/rooms/:id` | Update a room | 🔑 ADMIN |
| DELETE | `/rooms/:id` | Delete a room | 🔑 ADMIN |
| POST | `/admissions` | **Admit a patient** (transaction) | 🔑 ADMIN, STAFF |
| GET | `/admissions` | Get all admissions | 🔑 ADMIN, STAFF |
| GET | `/admissions/:id` | Get one admission | 🔑 ADMIN, STAFF |
| PATCH | `/admissions/:id` | Update admission date | 🔑 ADMIN, STAFF |
| PATCH | `/admissions/:id/discharge` | **Discharge a patient** (transaction) | 🔑 ADMIN, STAFF |
| DELETE | `/admissions/:id` | Delete a discharged admission | 🔑 ADMIN, STAFF |

### HTTP status codes used

| Code | Meaning |
|---|---|
| 200 | Success (GET, PATCH, DELETE, login) |
| 201 | Created (POST) |
| 400 | Validation error, invalid id, or a business rule failed (room full, active admission, etc.) |
| 401 | Missing / invalid token, or wrong email/password |
| 403 | Logged in, but the role is not allowed |
| 404 | Record not found |
| 409 | Duplicate email or duplicate room number |

## 10. Testing with Postman

### Step 1 – Login as ADMIN

`POST http://localhost:3000/auth/login`  → Body → **raw → JSON**

```json
{
  "email": "admin@rehab.local",
  "password": "Admin@123"
}
```

Response:

```json
{
  "message": "Login successful",
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": { "id": 1, "name": "System Admin", "email": "admin@rehab.local", "role": "ADMIN" }
}
```

**Copy the `accessToken`.** For every protected request, open the **Authorization** tab in Postman → Type **Bearer Token** → paste the token.

> Tip: in Postman you can create a collection variable `token` and use `{{token}}` in the Bearer Token field.

### Step 2 – Register a STAFF user and login

`POST /auth/register`

```json
{
  "name": "Staff One",
  "email": "staff1@rehab.local",
  "password": "staff123"
}
```

Then `POST /auth/login` with the same email and password to get the STAFF token.

### Step 3 – Create a room (ADMIN token)

`POST /rooms`

```json
{
  "roomNumber": "301",
  "capacity": 2
}
```

### Step 4 – Create a patient (ADMIN or STAFF token)

`POST /patients`

```json
{
  "name": "Sample Patient Four",
  "age": 30,
  "gender": "Male",
  "phone": "01700000004",
  "address": "78 Example Lane, Sample City",
  "emergencyContact": "01800000004"
}
```

### Step 5 – Admit the patient (transaction)

`POST /admissions`

```json
{
  "patientId": 1,
  "roomId": 1
}
```

Then call `GET /rooms/1` – `occupiedBeds` is now increased by 1.

### Step 6 – Discharge the patient (transaction)

`PATCH /admissions/1/discharge` (no body needed)

Then call `GET /rooms/1` – `occupiedBeds` is decreased by 1 and the admission status is `DISCHARGED`.

### Step 7 – Update examples (PATCH)

`PATCH /patients/1`
```json
{ "age": 29, "phone": "01700000099" }
```

`PATCH /rooms/1`
```json
{ "capacity": 4 }
```

### Things to test to show your examiner

| Test | Expected result |
|---|---|
| Call `GET /patients` with **no token** | 401 Unauthorized |
| Call `GET /users` with the **STAFF token** | 403 Forbidden |
| Call `POST /rooms` with the **STAFF token** | 403 Forbidden |
| Register the same email twice | 409 Conflict |
| `POST /auth/register` with `"email": "abc"` | 400 with validation messages |
| Admit a patient into a **full room** | 400 "Room ... is full" |
| Admit the same patient twice | 400 "already admitted" |
| `GET /patients/abc` | 400 (id must be a number) |
| `GET /patients/9999` | 404 Not found |
| Delete a patient / room that has an **active admission** | 400 (discharge first) |
| Send `"occupiedBeds": 5` when creating a room | 400 (property not allowed) |

## 11. How the Transactions Work (for your viva)

Both operations are in `src/admissions/admissions.service.ts` and use a TypeORM **QueryRunner**:

```
createQueryRunner()  →  connect()  →  startTransaction()
        try {
           ...all database steps...
           commitTransaction()        // save everything
        } catch {
           rollbackTransaction()      // undo everything if any step fails
        } finally {
           release()                  // give the connection back
        }
```

**Admit:** check patient exists → check room exists → check room has a free bed → check the patient has no other active admission → create admission → `occupiedBeds + 1`.

**Discharge:** find admission → check it is `ACTIVE` → set status `DISCHARGED` and `dischargeDate` → `occupiedBeds - 1`.

If any step throws an error, **nothing** is saved. The room row is locked during the transaction (`pessimistic_write`), so two staff members cannot take the last free bed at the same time.

## 12. Design Notes

- **`occupiedBeds` is not editable through the API.** It changes only through admit/discharge, so it can never be wrong. Creating or updating a room with `occupiedBeds` returns a 400 error.
- **Admission `PATCH` only updates `admissionDate`.** Room and status changes must go through the admit/discharge transactions.
- **Deleting a patient or room** that has only discharged history is allowed; their old admission records are deleted automatically (`onDelete: 'CASCADE'`).
- **`synchronize: true`** creates tables automatically. This is convenient for a university project; real production systems use migrations.

## 13. Common Problems

| Problem | Fix |
|---|---|
| `ER_BAD_DB_ERROR: Unknown database 'rehab_centre'` | Create the database first (see section 5) |
| `ER_ACCESS_DENIED_ERROR` | Check `DB_USERNAME` and `DB_PASSWORD` in `.env` |
| `ECONNREFUSED 127.0.0.1:3306` | MySQL is not running – start MySQL (or XAMPP) |
| `EADDRINUSE: port 3000` | Change `PORT` in `.env`, or stop the other program using port 3000 |
| `bcrypt` fails to install | Use Node.js LTS (18, 20 or 22) and run `npm install` again |
