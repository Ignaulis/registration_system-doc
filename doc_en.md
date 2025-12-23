# Registration System (form builder + capacity control + email confirmations)

## Overview
This project is a **registration system** for creating and managing registration forms across multiple days (e.g., Monday–Sunday), with activities assigned to each day. Each activity has a **time slot** and a **capacity limit**, and applicants register via a unique public form URL. The system automatically manages availability, sends email confirmations, and provides admins with full registration visibility and management tools.

---

## Core capabilities

### 1) Form creation & management
- Admins can create a new form by defining:
  - form title
  - form date / time range (as needed)
  - days (Monday, Tuesday, …) and activities per day
- Each day can contain **any number of activities**, e.g.:
  - 10:00 – Sensory activity (8 seats)
  - 12:00 – Movement activity (10 seats)
  - 14:00 – Creative workshop (6 seats)
- Admins can:
  - view all created forms in a single list
  - edit a form (title, dates, days, activities, capacity)
  - delete a form

### 2) Unique public form link (UUID)
- Every form automatically gets a unique **UUID** and a dedicated public URL.
- The URL can be:
  - opened to fill out the form
  - shared with others (parents / applicants)
- The link serves as the “public entry point” for a specific form.

### 3) Capacity control & selection restrictions
- Applicants can only select activities that still have available seats.
- When an activity is full:
  - it becomes **unselectable** (disabled / hidden depending on UI logic)
  - the system also prevents submitting a registration for a full activity even if someone tries to bypass the UI (server-side validation)
- Availability is calculated from registrations (counting selections for a given activity).

### 4) Registration (public flow)
- The applicant opens the public form URL and submits:
  - contact information (based on form requirements)
  - selected activities by day and time
- After successful submission:
  - the applicant receives a **confirmation email**
  - the email includes a clear summary of selected activities and times

### 5) Admin registration management
- Admins can view all submitted registrations:
  - list view with search / filters
  - filtering by a specific activity (e.g., show only “Sensory 10:00”)
- Admin actions available:
  - edit
  - delete
  - view registration details

### 6) Communication & cancellation
- Admins can quickly contact an applicant (e.g., via a “write now” action).
- The confirmation email includes an option to **cancel the registration**:
  - once canceled, seats become available again for the affected activities
- Admins can also cancel from the admin panel.

---

## Architecture (high-level)

```text
Applicant (public)
  └─ Public form URL (UUID)
        ├─ Form view (days + activities + availability)
        ├─ Selections (only if seats are available)
        └─ Submit registration
                ↓
Server-side (Next.js)
  ├─ Fetch form + activities from DB
  ├─ Capacity validation during submission
  ├─ Persist registration in DB (Supabase)
  └─ Send confirmation email (Nodemailer)
                ↓
Admin panel (protected)
  ├─ Google OAuth login (NextAuth)
  ├─ Access control via admins table (allowlist)
  ├─ Forms CRUD (create/edit/delete)
  ├─ Form activity management (days/time/capacity)
  ├─ Registrations list + filtering
  ├─ Edit/delete/cancel registrations
  └─ Communication with applicants
```

---

## Database

### Registration domain (4 tables)
The registration domain is modeled with 4 core tables:

1) **`forms`**  
- registration forms (title, date, UUID, etc.)

2) **`form_activities`**  
- all activities belonging to a specific form (day, time, capacity)  
- **relation:** `form_activities.form_id` → `forms.id` (one form has many activities)

3) **`registrations`**  
- applicant registrations for a specific form (contacts, status, created_at, etc.)  
- **relation:** `registrations.form_id` → `forms.id` (one form has many registrations)

4) **`registration_activities`**  
- join table describing which activities were selected in a registration  
- **relations:**
  - `registration_activities.registration_id` → `registrations.id`
  - `registration_activities.form_activity_id` → `form_activities.id`

### Admin access table
5) **`admins`**  
- allowlist of authorized admin users (e.g., email addresses)
- used to grant access to the admin panel

### Relationship logic (summary)
- A **form** (`forms`) has many **form activities** (`form_activities`).
- A **form** (`forms`) has many **registrations** (`registrations`).
- A **registration** (`registrations`) has many selected **registration activities** (`registration_activities`).
- `registration_activities.form_activity_id` always points to a specific `form_activities` row, which enables:
  - accurate occupancy calculations (count selections per activity)
  - filtering registrations by activity
  - ensuring selected activities belong to the same form

*(Detailed column-level schema and internal identifiers are intentionally not described publicly.)*

---

## Authentication & access control
- Admin login uses **Google OAuth**
- Authentication is implemented via **NextAuth**
- A signed-in user gets admin access only if their email exists in the `admins` table (DB allowlist).

---

## Tech stack
- **Next.js** `15.5.2`
- **React** `19.1.0`
- **Tailwind CSS** `v4`
- **Supabase** (`@supabase/supabase-js`) – database
- **NextAuth** – Google OAuth for admin area
- **Nodemailer** – transactional emails
- **UUID** – unique form links
- UI / UX:
  - **Framer Motion**
  - **React Icons**

---

## End-to-end flow

### Applicant registration
1. Applicant opens a form URL (UUID)
2. The system displays days, activities, time slots, and available seats
3. Applicant selects activities (only if seats are available)
4. Submits the registration
5. Server validates capacity and writes to DB:
   - a row in `registrations`
   - selected rows in `registration_activities`
6. A confirmation email is sent with a summary and a cancellation option

### Admin workflow
1. Admin signs in via Google OAuth
2. Creates/edits forms and their activities (`forms` + `form_activities`)
3. Monitors registrations and filters by activity
4. Edits/deletes/cancels registrations and contacts applicants
5. Availability is updated based on `registration_activities` records
