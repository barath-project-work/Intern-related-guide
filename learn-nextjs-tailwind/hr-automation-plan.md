# 📋 HR Automation — Project Planning Document

> **Project:** HR Automation Platform  
> **Role:** Full Stack Developer Intern  
> **Status:** Planning Phase  
> **Date:** July 2026

---

## 📑 Table of Contents

1. [Project Overview](#1-project-overview)
2. [User Roles & Permissions](#2-user-roles--permissions)
3. [Complete Feature List](#3-complete-feature-list)
4. [Workflow Diagrams](#4-workflow-diagrams)
5. [Pages & Routes Map](#5-pages--routes-map)
6. [Data Models / Database Schema](#6-data-models--database-schema)
7. [Integration Points](#7-integration-points)
8. [File Structure Organization](#8-file-structure-organization)
9. [Google Drive Organization](#9-google-drive-organization)
10. [Development Phases & Priority](#10-development-phases--priority)
11. [Learning Roadmap](#11-learning-roadmap)

---

## 1. Project Overview

### 1.1 What is HR Automation?

An internal web platform for HR to manage the entire **internship lifecycle** — from receiving applications to generating experience certificates. The system automates repetitive tasks like email notifications, document generation, and folder organization.

### 1.2 Core Purpose

| Area | Description |
|------|-------------|
| **Application Tracking** | Fetch, review, and manage intern applications from Google Sheets |
| **Interview Management** | Schedule interviews, auto-send Google Meet links & emails |
| **Document Management** | Upload & organize candidate documents in Google Drive |
| **Letter Generation** | Auto-generate joining letters, experience certificates, LORs |
| **Task Tracking** | Per-candidate task tracker with working day calculations |
| **Lifecycle Management** | Handle intern onboarding → active → termination/completion |

### 1.3 Key Principles

- **Two-way sync with Google Sheets** — actions on the platform reflect back to the sheet
- **Google Drive integration** — automatic folder creation and file storage
- **Email automation** — candidates receive updates automatically
- **Canva template** — joining letter template designed in Canva, manually uploaded
- **Role-based access** — HR and Candidate see different views

---

## 2. User Roles & Permissions

### 👤 Role 1: HR / Admin

| Permission | Scope |
|------------|-------|
| Login | ID + Password |
| View all applications | Full list |
| Accept / Reject / Hold applications | Write |
| Delete applications (platform only) | Write |
| Manually enter applications | Create |
| Schedule interviews | Create |
| Send agreement to candidate | Create |
| Mark agreement as received | Update |
| Upload candidate documents (marksheet, aadhar) | Create |
| Generate & approve joining letters | Write / Approve |
| Generate experience certificates & LORs | Create |
| View all candidates' task trackers | Read |
| Terminate internship | Write |
| Edit any data | Update |
| Access HR Drive | Full |

### 👤 Role 2: Candidate / Internee

| Permission | Scope |
|------------|-------|
| View own task tracker | Read-only |
| Receive automated emails | Auto |
| Print & sign agreement (offline) | Manual |
| Send signed agreement to HR via email | Manual |
| View own documents (agreement, etc.) | Read-only |

---

## 3. Complete Feature List

### 🔐 F1 — Authentication System

**Description:** HR logs in with ID and password. Candidates do not log in — they interact via email.

**Details:**
- Login page with credential-based authentication
- Session management
- Role-based access control (RBAC)
- Logout functionality

---

### 📋 F2 — Application Management (Google Sheet Sync)

**Description:** Applications are fetched from a Google Sheet and displayed in the platform. HR can take actions that sync back.

**Sub-features:**

| Feature | Detail |
|---------|--------|
| **F2.1 — Fetch from Sheet** | Load all applications from a specified Google Sheet |
| **F2.2 — Accept** | Mark application as accepted → moves to interview stage |
| **F2.3 — Reject** | Mark application as rejected → candidate filtered out |
| **F2.4 — Hold** | Move to "Currently In Hold" section for later decision |
| **F2.5 — Delete** | Remove from platform only (NOT from Google Sheet) |
| **F2.6 — Manual Entry** | Manually add an application for candidates who applied via email |

**Hold Behavior:**
- Separate page showing only "on hold" applications
- From hold page: can still Accept or Reject

**Two-Way Sync Detail:**
| Action | Platform DB | Google Sheet |
|--------|-------------|--------------|
| Application fetched | ✅ Added | — (already in sheet) |
| Accept | ✅ Updated | ✅ Updated |
| Reject | ✅ Updated | ✅ Updated |
| Hold | ✅ Updated | ✅ Updated |
| Delete | ✅ Removed | ❌ No change |
| Manual Entry | ✅ Added | ✅ Added |

---

### 🎤 F3 — Interview Scheduling

**Description:** When a candidate is accepted, schedule their interview.

**Workflow:**
1. HR clicks "Schedule Interview" for an accepted candidate
2. Modal/form opens with fields: **Time** + **Google Meet Link**
3. HR clicks "Schedule" → email auto-sent to candidate
4. After interview: three options appear

**Post-Interview Actions:**

```
                ┌──────────────┐
                │   Interview  │
                │    Done      │
                └──────┬───────┘
                       │
                ┌──────┴──────┐
                │             │
                ▼             ▼
          ┌─────────┐   ┌──────────┐
          │Appointed│   │Not Apptd │
          └─────────┘   └──────────┘
                │             │
                │       ┌─────┴─────┐
                │       │           │
                │       ▼           ▼
                │  ┌─────────┐  ┌─────────┐
                │  │  Hold   │  │ Not App │
                │  └─────────┘  └─────────┘
                │       │
                │  ┌────┴────┐
                │  │         │
                │  ▼         ▼
                │  Apptd  Not Apptd
                ▼
         ┌────────────────────┐
         │ Send Agreement to  │
         │ Candidate (Email)  │
         └────────┬───────────┘
                  │
         ┌────────┴───────────┐
         │                    │
         ▼                    ▼
   ┌──────────────┐   ┌──────────────┐
   │ Candidate    │   │ Agreement    │
   │ Prints &     │   │ Status:      │
   │ Signs        │   │ Pending      │
   └──────┬───────┘   └──────────────┘
          │
          ▼
   ┌────────────────────┐
   │ Candidate Sends    │
   │ Signed Copy via    │
   │ Email to HR        │
   └────────┬───────────┘
            │
            ▼
   ┌────────────────────┐
   │ HR Uploads Signed  │
   │ Agreement to       │
   │ Platform           │
   └────────┬───────────┘
            │
   ┌────────┴───────────┐
   │                    │
   ▼                    ▼
┌──────────────┐   ┌──────────────┐
│ Agreement    │   │ Agreement    │
│ Received ✅  │   │ Not Received │
│ → Proceed    │   │ → Follow Up  │
└──────┬───────┘   └──────────────┘
       │
       ▼
  ┌────────────────────┐
  │   Onboarding       │
  │ (Upload Docs)      │
  └────────┬───────────┘
```

---

### 🤝 F3.5 — Agreement Workflow (Send → Sign → Receive)

**Description:** After a candidate is appointed post-interview, HR sends an agreement letter via email for the candidate to sign offline. The signed copy comes back via email and is reflected in the platform.

**The Complete Flow:**

| Step | Action | Who | Detail |
|------|--------|-----|--------|
| 1 | Generate agreement | HR / System | System creates agreement PDF with placeholder fields (name, signature, date) |
| 2 | Send via email | HR / Auto | Agreement PDF sent to candidate's email |
| 3 | Print & sign | Candidate | Candidate prints the agreement, signs it manually |
| 4 | Send back via email | Candidate | Candidate scans/photographs signed agreement and emails it back to HR |
| 5 | Mark as received | HR | HR uploads the signed agreement to the platform (or system detects email) |
| 6 | Status updated | Auto | Agreement status changes to `received` → unlocks joining letter stage |

**Agreement Template Elements (Placeholders):**

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
         INTERNSHIP AGREEMENT LETTER
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Date: [Auto: Current Date]

To,
[Candidate Full Name]
[Candidate Address]

Subject: Internship Agreement

Dear [Candidate Full Name],

We are pleased to offer you an internship at [Company Name].

... terms and conditions ...

_________________________________
Company Representative Signature

_________________________________
Candidate Signature              ← Placeholder for manual sign

Candidate Name: [Empty]          ← To be filled by candidate
Date: [Empty]                    ← To be filled by candidate
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Agreement Statuses Tracked in Platform:**

| Status | Meaning |
|--------|---------|
| `not_sent` | Agreement hasn't been sent yet |
| `sent` | Agreement emailed to candidate, awaiting signature |
| `received` | Signed agreement received back |
| `expired` | Follow-up needed (not received after X days) |

**How It Reflects in the Platform:**
- Once HR marks the agreement as received, the platform shows a **green "Agreement Received ✅"** badge on the candidate's profile
- The signed PDF is stored in the candidate's folder in Drive
- The system records the date received
- Only after agreement is marked `received` can HR proceed to Joining Letter generation

**Email to Candidate (Agreement Send):**

```
Subject: Your Internship Agreement with [Company Name]

Dear [Candidate Name],

Congratulations on your selection! Please find attached your
Internship Agreement Letter.

Instructions:
1. Print the attached agreement
2. Sign at the designated space
3. Scan or take a clear photo
4. Reply to this email with the signed copy

Please ensure this is done within [X] days.

Best regards,
HR Team
[Company Name]
```

---

### 📄 F4 — Joining Letter Generation

**Description:** Generate a joining letter for appointed candidates.

**Workflow:**

| Step | Action | Who |
|------|--------|-----|
| 1 | Click "Generate Joining Letter" | HR |
| 2 | System auto-generates letter with: DOJ, address, full name, all details | Auto |
| 3 | HR reviews the letter | HR |
| 4 | HR approves/rejects | HR |
| 5 | If approved → email to candidate with letter attached | Auto |
| 6 | Letter saved to HR Drive in joining letters folder | Auto |

**Template Source:**
- Template designed in **Canva** (externally)
- HR manually uploads the template to the platform
- System fills in the template with candidate data

**Letter Elements:**
- Date of Joining
- Candidate's Full Name
- Company Address
- Role/Position
- Terms & Conditions
- HR Signature

---

### 📁 F5 — Document Upload & Drive Organization

**Description:** Upload candidate documents with automatic folder creation.

> **Note:** The **Signed Agreement Letter** upload here happens when HR manually uploads the signed PDF received from the candidate via email. See [F3.5 — Agreement Workflow](#-f35--agreement-workflow-send--sign--receive) for the full flow.

**Documents to Upload:**
| Document | Type | Required | Source |
|----------|------|----------|--------|
| Signed Agreement Letter | PDF | ✅ | Received from candidate via email (F3.5) |
| Marksheet | PDF/Image | ✅ | Candidate provides |
| Aadhar Card | PDF/Image | ✅ | Candidate provides |

**Auto-Folder Creation:**
```
HR Drive/
├── Applications/          # (from Google Sheet)
├── Candidates/
│   ├── John Doe/
│   │   ├── Agreement.pdf
│   │   ├── Marksheet.pdf
│   │   └── Aadhar.pdf
│   └── Jane Smith/
│       ├── Agreement.pdf
│       ├── Marksheet.pdf
│       └── Aadhar.pdf
├── Joining Letters/
│   ├── John_Doe_Joining_Letter.pdf
│   └── Jane_Smith_Joining_Letter.pdf
├── Intern Tasks/
│   ├── John Doe working/
│   │   └── Task_Tracker.xlsx
│   └── Jane Smith working/
│       └── Task_Tracker.xlsx
└── Certificates/
    ├── Experience/
    │   ├── John_Doe_Experience.pdf
    │   └── Jane_Smith_Experience.pdf
    └── LOR/
        ├── John_Doe_LOR.pdf
        └── Jane_Smith_LOR.pdf
```

---

### 📊 F6 — Task Tracker (Per Candidate)

**Description:** Each candidate has their own task tracker Excel sheet.

**Details:**
- Inside the `[Candidate Name] working/` folder
- Excel sheet with task tracking columns
- **Only visible to the candidate** (candidate view-only)
- Tracks: number of working days, daily tasks, status

**Task Tracker Columns (suggested):**
| Date | Task Description | Status | Remarks |
|------|-----------------|--------|---------|
| — | — | — | — |

**Working Days Calculation:**
- System tracks the number of working days
- Displayed in candidate's profile/portal

---

### 📜 F7 — Certificate & LOR Generation

**Description:** When internship ends, generate completion documents.

**Two Possible States:**
| State | Action |
|-------|--------|
| ✅ Internship Complete | Generate **Experience Certificate** + **Letter of Recommendation (LOR)** |
| ❌ Terminated | No certificate generated |

**Experience Certificate Elements:**
- Candidate Name
- Internship Duration (start → end date)
- Role
- Company Name & Logo
- HR Signature

**LOR Elements:**
- Candidate Name
- Performance description
- Company letterhead
- HR Signature

---

### ✏️ F8 — Edit Capability

**Description:** HR can edit any candidate's data across all sections.

**Editable Fields:**
- Candidate personal details
- Application status
- Interview details
- Document metadata
- Generated letter content (before approval)

---

## 4. Workflow Diagrams

### 4.1 Full Application Lifecycle Flow

```
                  ┌─────────────┐
                  │  Google     │
                  │  Sheet      │
                  └──────┬──────┘
                         │ Fetch
                         ▼
                  ┌─────────────┐
                  │  Dashboard  │
                  │ (All Apps)  │
                  └──────┬──────┘
                         │
         ┌───────────────┼───────────────┐
         │               │               │
         ▼               ▼               ▼
   ┌──────────┐   ┌──────────┐   ┌──────────┐
   │  Accept  │   │  Reject  │   │   Hold   │
   └────┬─────┘   └──────────┘   └────┬─────┘
        │                              │
        ▼                              ▼
   ┌──────────┐                  ┌──────────┐
   │Schedule  │                  │ Hold     │
   │Interview │                  │ Page     │
   └────┬─────┘                  └────┬─────┘
        │                             │
        ▼                    ┌────────┴────────┐
   ┌──────────┐              │                 │
   │Interview │              ▼                 ▼
   │Completed │         ┌─────────┐      ┌─────────┐
   └────┬─────┘         │ Accept  │      │ Reject  │
        │               └────┬────┘      └─────────┘
   ┌────┴────┐               │
   │         │               ▼
   ▼         ▼          ┌──────────┐
┌──────┐ ┌────────┐     │Schedule  │
│Apptd │ │Not App │     │Interview │
└──┬───┘ └────────┘     └──────────┘
   │
   ▼
┌──────────────────────────────┐
│  💼 F3.5 — Agreement Flow    │
│                               │
│  1. HR sends agreement PDF   │
│     via email (with blanks)  │
│  2. Candidate prints & signs │
│  3. Candidate scans & sends  │
│     back via email           │
│  4. HR uploads to platform   │
│     → Status: Received ✅    │
└──────────────┬───────────────┘
               │
               ▼
┌──────────────┐
│ Onboarding   │
│ (Upload Docs)│
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Joining      │
│ Letter Gen   │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ Internship   │
│ Active       │
├──────────────┤
│ Task Tracker │
└──────┬───────┘
       │
   ┌───┴────┐
   │        │
   ▼        ▼
┌──────┐ ┌──────┐
│Complete│Terminated│
└──┬───┘ └──────┘
   │
   ▼
┌──────────────┐
│ Exp Cert +   │
│ LOR Generated│
└──────────────┘
```

### 4.2 Email Automation Flow

```
Action
  │
  ▼
┌─────────────┐
│ Trigger     │
│ Event       │
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ Send Email  │
│ (Auto)      │
└──────┬──────┘
       │
       ▼
Events that trigger email:
├── Interview Scheduled → Candidate
├── Agreement Sent to Candidate → Candidate (with PDF attachment)
├── Joining Letter Approved → Candidate
├── Internship Complete → Certificate sent
├── Agreement Not Received (Follow-up) → Candidate
└── Rejected → Rejection notification
```

---

## 5. Pages & Routes Map

### 5.1 Complete Page List

| # | Page | Route | Role | Description |
|---|------|-------|------|-------------|
| 1 | **Login** | `/login` | Public | HR login with ID & password |
| 2 | **Dashboard** | `/dashboard` | HR | All applications overview |
| 3 | **Hold List** | `/hold` | HR | Applications currently on hold |
| 4 | **Interview Schedule** | `/interview/:id` | HR | Schedule interview for a candidate |
| 5 | **Interview Queue** | `/interviews` | HR | All scheduled interviews |
| 5b | **Agreement** | `/candidate/:id/agreement` | HR | Send agreement, track status, upload signed copy |
| 6 | **Documents** | `/candidate/:id/documents` | HR | Upload & view candidate documents |
| 7 | **Joining Letter** | `/candidate/:id/joining-letter` | HR | Generate, review & approve |
| 8 | **Candidates List** | `/candidates` | HR | All appointed/interning candidates |
| 9 | **Candidate Profile** | `/candidate/:id` | HR | Full candidate details & actions |
| 10 | **Manual Entry** | `/add-application` | HR | Manually add an application |
| 11 | **Task Tracker** | `/candidate/:id/tasks` | HR + Candidate | View task tracker (candidate: read-only) |
| 12 | **Settings** | `/settings` | HR | Google Sheet config, Drive config, Canva template upload |
| 13 | **Certificates** | `/candidate/:id/certificates` | HR | Generate Experience Cert & LOR |

### 5.2 Navigation Structure (HR Side)

```
Dashboard ──────────────────────────────────
├── 📊 All Applications (default view)
│   ├── Accept → Schedule Interview
│   ├── Reject
│   └── Hold → /hold
├── ⏸ Hold List
├── 📅 Interviews
├── 👥 Candidates
│   └── [Candidate]
│       ├── 📁 Documents
│       ├── 📄 Joining Letter
│       ├── 📊 Task Tracker
│       └── 📜 Certificates
├── ➕ Add Application
└── ⚙️ Settings
```

### 5.3 Route Hierarchy (Next.js App Router)

```
src/app/
├── login/
│   └── page.tsx                    # F1 - Login
├── (dashboard)/                     # Layout with sidebar
│   ├── layout.tsx                   # Dashboard layout (protected)
│   ├── dashboard/
│   │   └── page.tsx                 # F2 - All applications
│   ├── hold/
│   │   └── page.tsx                 # F2.4 - Hold list
│   ├── interviews/
│   │   ├── page.tsx                 # F3 - Interview queue
│   │   └── [id]/
│   │       └── page.tsx             # F3 - Schedule interview
│   ├── candidates/
│   │   ├── page.tsx                 # F8 - All candidates
│   │   └── [id]/
│   │       ├── page.tsx             # F8 - Candidate profile
│   │       ├── agreement/
│   │       │   └── page.tsx         # F3.5 - Agreement workflow
│   │       ├── documents/
│   │       │   └── page.tsx         # F5 - Document upload
│   │       ├── joining-letter/
│   │       │   └── page.tsx         # F4 - Joining letter
│   │       ├── tasks/
│   │       │   └── page.tsx         # F6 - Task tracker
│   │       └── certificates/
│   │           └── page.tsx         # F7 - Certificates & LOR
│   ├── add-application/
│   │   └── page.tsx                 # F2.6 - Manual entry
│   └── settings/
│       └── page.tsx                 # Integrations config
└── api/
    ├── auth/
    │   └── [...nextauth]/
    │       └── route.ts             # Auth API
    ├── applications/
    │   ├── route.ts                 # GET (list), POST (manual entry)
    │   └── [id]/
    │       └── route.ts             # PATCH (status), DELETE
    ├── interviews/
    │   └── route.ts                 # POST (schedule)
    ├── documents/
    │   └── route.ts                 # POST (upload to Drive)
    ├── agreement/
    │   └── route.ts                 # POST (send agreement), PATCH (mark received)
    ├── letters/
    │   ├── joining/
    │   │   └── route.ts             # POST (generate)
    │   └── certificate/
    │       └── route.ts             # POST (generate)
    ├── sheets/
    │   └── sync/
    │       └── route.ts             # POST (sync with sheet)
    └── emails/
        └── route.ts                 # POST (send email)
```

---

## 6. Data Models / Database Schema

### 6.1 Tables (Supabase / PostgreSQL)

#### `users` — HR Users

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID (PK) | Primary key |
| `email` | TEXT (unique) | Login email |
| `password_hash` | TEXT | Hashed password |
| `name` | TEXT | HR name |
| `role` | TEXT | `'hr'` or `'admin'` |
| `created_at` | TIMESTAMP | Auto |

#### `applications` — All Applications

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID (PK) | Primary key |
| `sheet_row_id` | TEXT | Reference to Google Sheet row |
| `full_name` | TEXT | Candidate's name |
| `email` | TEXT | Candidate's email |
| `phone` | TEXT | Phone number |
| `position` | TEXT | Applied position |
| `status` | TEXT | `pending`, `accepted`, `rejected`, `hold`, `appointed`, `not_appointed` |
| `agreement_status` | TEXT | `not_sent`, `sent`, `received`, `expired` (new) |
| `agreement_sent_at` | TIMESTAMP | When agreement was emailed to candidate |
| `agreement_received_at` | TIMESTAMP | When signed agreement was received |
| `source` | TEXT | `sheet` or `manual` |
| `sheet_synced` | BOOLEAN | Whether status is synced back to sheet |
| `created_at` | TIMESTAMP | Auto |
| `updated_at` | TIMESTAMP | Auto |

#### `interviews` — Scheduled Interviews

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID (PK) | Primary key |
| `application_id` | UUID (FK) | References `applications.id` |
| `scheduled_at` | TIMESTAMP | Date & time of interview |
| `meet_link` | TEXT | Google Meet link |
| `status` | TEXT | `scheduled`, `appointed`, `not_appointed`, `hold` |
| `email_sent` | BOOLEAN | Whether confirmation email was sent |
| `created_at` | TIMESTAMP | Auto |

#### `documents` — Uploaded Documents

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID (PK) | Primary key |
| `application_id` | UUID (FK) | References `applications.id` |
| `type` | TEXT | `agreement`, `marksheet`, `aadhar` |
| `file_name` | TEXT | Original file name |
| `drive_url` | TEXT | Google Drive file URL |
| `drive_folder_id` | TEXT | Google Drive folder ID |
| `uploaded_at` | TIMESTAMP | Auto |

#### `agreement_logs` — Agreement Audit Trail (new)

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID (PK) | Primary key |
| `application_id` | UUID (FK) | References `applications.id` |
| `action` | TEXT | `sent`, `received`, `followed_up` |
| `details` | TEXT | Extra info (e.g., "Sent to candidate@email.com") |
| `created_at` | TIMESTAMP | Auto |

#### `letters` — Generated Letters

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID (PK) | Primary key |
| `application_id` | UUID (FK) | References `applications.id` |
| `type` | TEXT | `joining`, `experience`, `lor` |
| `content` | TEXT | Generated letter content |
| `status` | TEXT | `draft`, `approved`, `sent` |
| `drive_url` | TEXT | Final PDF URL in Drive |
| `created_at` | TIMESTAMP | Auto |
| `approved_at` | TIMESTAMP | When approved |
| `sent_at` | TIMESTAMP | When emailed to candidate |

#### `task_trackers` — Task Tracker Data

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID (PK) | Primary key |
| `application_id` | UUID (FK) | References `applications.id` |
| `date` | DATE | Task date |
| `task_description` | TEXT | What was done |
| `status` | TEXT | `pending`, `in_progress`, `completed` |
| `remarks` | TEXT | Optional notes |
| `working_days_count` | INTEGER | Running count of working days |
| `created_at` | TIMESTAMP | Auto |

#### `settings` — Platform Configuration

| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID (PK) | Primary key |
| `key` | TEXT (unique) | Setting key |
| `value` | JSONB | Setting value |
| `updated_at` | TIMESTAMP | Auto |

**Example Settings:**
```json
{
  "google_sheet_id": "1abc...",
  "google_sheet_range": "A2:F",
  "google_drive_root_folder_id": "folder_123",
  "canva_template_url": "https://canva.com/...",
  "company_name": "ABC Corp",
  "company_address": "123 Street, City"
}
```

---

## 7. Integration Points

### 7.1 Integrations Overview

| # | Integration | Purpose | Direction |
|---|-------------|---------|-----------|
| 1 | **Google Sheets API** | Fetch & sync applications | Two-way |
| 2 | **Google Drive API** | Store documents & letters | Upload |
| 3 | **Google Meet API** | Generate meeting links | Create |
| 4 | **Gmail API / Nodemailer** | Send automated emails | Send |
| 5 | **Canva (Manual)** | Design template, upload PDF | Import |

### 7.2 Google Sheets — Sync Logic

```
Fetch Flow:
  Google Sheet → Read rows → Map to applications → Store in DB
  (Run on: page load, manual sync button, or scheduled)

Write Flow:
  Platform action → Update DB → Write to Google Sheet
  (Run on: Accept, Reject, Hold, Manual Entry)
```

### 7.3 Email Automation Triggers

| Event | Email Content | To |
|-------|--------------|-----|
| Interview Scheduled | Time, Google Meet link, date | Candidate |
| **Agreement Sent** | **PDF attachment with blanks for name & signature + instructions to print, sign, and reply** | **Candidate** |
| **Agreement Follow-up** | **Reminder to send signed agreement if not received within X days** | **Candidate** |
| Joining Letter Approved | Attached PDF, joining instructions | Candidate |
| Rejected | Polite rejection notice | Candidate |
| Internship Complete | Certificate attached, next steps | Candidate |
| Terminated | Termination notice | Candidate |

---

## 8. File Structure Organization

### 8.1 Project Directory (Next.js)

```
hr-automation/
├── src/
│   ├── app/                    # App Router pages
│   ├── components/             # Reusable components
│   │   ├── ui/                 # Button, Card, Modal, Input etc.
│   │   ├── layout/             # Sidebar, Navbar, ProtectedLayout
│   │   ├── applications/       # ApplicationTable, ApplicationCard
│   │   ├── interviews/         # InterviewForm, InterviewCard
│   │   ├── documents/          # UploadZone, DocumentList
│   │   └── letters/            # LetterPreview, LetterEditor
│   ├── lib/                    # Utilities & config
│   │   ├── supabase/           # Supabase client & queries
│   │   ├── google/             # Sheets, Drive, Gmail APIs
│   │   ├── email/              # Email templates & sender
│   │   └── utils.ts            # Shared helpers
│   ├── types/                  # TypeScript types
│   └── middleware.ts           # Auth middleware
├── public/                     # Static assets
├── .env.local                  # Environment variables
├── tailwind.config.ts
├── next.config.ts
└── package.json
```

---

## 9. Google Drive Organization

### 9.1 Folder Structure

```
📁 HR Drive (Root)
│
├── 📁 Applications/                    # Linked to Google Sheet
│   └── (auto-synced)
│
├── 📁 Candidates/                      # One folder per candidate
│   ├── 📁 [Candidate Name]/
│   │   ├── 📄 Agreement_Letter.pdf        # Signed copy received from candidate
│   │   ├── 📄 Marksheet.pdf
│   │   └── 📄 Aadhar.pdf
│   └── 📁 [Candidate Name]/
│       └── ...
│
├── 📁 Joining Letters/                 # All approved joining letters
│   ├── 📄 [Name]_Joining_Letter.pdf
│   └── ...
│
├── 📁 Intern Tasks/                    # Per-candidate task trackers
│   ├── 📁 [Candidate Name] working/
│   │   └── 📊 Task_Tracker.xlsx        # Visible to candidate
│   └── ...
│
└── 📁 Certificates/                    # Completion documents
    ├── 📁 Experience/
    │   ├── 📄 [Name]_Experience.pdf
    │   └── ...
    └── 📁 LOR/
        ├── 📄 [Name]_LOR.pdf
        └── ...
```

### 9.2 Folder Auto-Creation Logic

```javascript
// When a candidate is appointed:
// When a candidate is appointed:
createFolder(`Candidates/${candidate.name}`)
createFolder(`Intern Tasks/${candidate.name} working/`)
createFile(`Intern Tasks/${candidate.name} working/Task_Tracker.xlsx`)

// When agreement is sent to candidate:
// (email sent with attachment, nothing saved to Drive yet)

// When signed agreement is received:
saveFile(`Candidates/${candidate.name}/Agreement_Letter_Signed.pdf`)

// When joining letter is approved:
saveFile(`Joining Letters/${candidate.name}_Joining_Letter.pdf`)

// When internship completes:
saveFile(`Certificates/Experience/${candidate.name}_Experience.pdf`)
saveFile(`Certificates/LOR/${candidate.name}_LOR.pdf`)
```

---

## 10. Development Phases & Priority

### Phase 1: Foundation 🏗️ (Week 1-2)

| Priority | Feature | Why First? |
|----------|---------|------------|
| 🥇 P0 | Authentication (F1) | Everything needs login |
| 🥇 P0 | Dashboard & Application List (F2.1) | Core page |
| 🥇 P0 | Accept / Reject / Hold actions (F2.2-2.4) | Primary actions |
| 🥇 P0 | Hold Page (F2.4 sub) | Separate view needed |
| 🥇 P0 | Delete Application (F2.5) | Data management |
| 🥇 P0 | Google Sheet Integration | Foundation for data |

**Key Learning Goals:**
- Next.js project setup & App Router
- Tailwind CSS layout & components
- Supabase auth & database
- Google API authentication

### Phase 2: Core Features 🚀 (Week 3-4)

| Priority | Feature |
|----------|---------|
| 🥈 P1 | Manual Entry (F2.6) |
| 🥈 P1 | Interview Scheduling (F3) |
| 🥈 P1 | Email Automation |
| 🥈 P1 | Document Upload (F5) |
| 🥈 P1 | Google Drive Integration |

**Key Learning Goals:**
- Server Actions & API routes
- Email sending (Nodemailer / Resend)
- Google Drive API
- File upload handling

### Phase 3: Advanced 🔧 (Week 5-6)

| Priority | Feature |
|----------|---------|
| 🥉 P2 | Joining Letter Generation (F4) |
| 🥉 P2 | Task Tracker (F6) |
| 🥉 P2 | Working Days Calculation |
| 🥉 P2 | Edit Capability (F8) |
| 🥉 P2 | Canva Template Upload |

**Key Learning Goals:**
- PDF generation (jsPDF / PDFKit)
- Canva template processing
- Excel file generation
- Complex state management

### Phase 4: Completion 🏁 (Week 7-8)

| Priority | Feature |
|----------|---------|
| 🔵 P3 | Certificate & LOR Generation (F7) |
| 🔵 P3 | Termination Flow |
| 🔵 P3 | Candidate View (read-only task tracker) |
| 🔵 P3 | Testing & Bug Fixes |
| 🔵 P3 | Deployment |

**Key Learning Goals:**
- Full lifecycle testing
- Deployment (Vercel)
- Performance optimization
- Documentation

---

## 11. Learning Roadmap

### Phase 1 Foundational Skills

```mermaid
Next.js App Router
├── File-based routing
├── Layouts (nested, shared)
├── Server Components
└── Client Components ("use client")

Tailwind CSS
├── Utility classes
├── Responsive design (sm:, md:, lg:)
├── Flexbox & Grid
└── Dark mode

Supabase
├── Project setup
├── Authentication (email/password)
├── Database (tables, RLS)
└── Client SDK (@supabase/supabase-js)
```

### Phase 2: Intermediate Skills

```mermaid
Next.js
├── Server Actions (form handling)
├── API Route Handlers
├── Dynamic routes [id]
├── Loading & Error states
└── Middleware (auth protection)

Tailwind CSS
├── Custom theme configuration
├── Component patterns (Card, Table, Modal)
├── Animations & transitions
└── Composition with clsx/twMerge

Google APIs
├── Google Sheets API (read/write)
├── Google Drive API (create folders, upload)
├── Google Meet API (create meetings)
└── OAuth 2.0 / Service Account

Email
├── Nodemailer / Resend
├── Email templates (react-email)
└── SMTP configuration
```

### Phase 3: Advanced Skills

```mermaid
PDF Generation
├── jsPDF / PDF-Lib
├── Template filling
├── Watermarks & signatures
└── PDF merging

File Handling
├── Multer / UploadThing
├── File validation (type, size)
├── Progress indicators
└── Secure URLs

Excel Generation
├── ExcelJS / SheetJS
├── Cell formatting
├── Formulas (working days)
└── Sharing permissions
```

---

## 🧠 Key Takeaways

| # | Lesson |
|---|--------|
| 1 | **Start with planning** — You're doing this right now. Document everything first. |
| 2 | **Google Sheet is the source of truth** — Two-way sync means your DB and sheet must stay in sync |
| 3 | **Folders are auto-created** — Drive structure follows a strict naming convention |
| 4 | **Email is automation-friendly** — Every major action triggers an email |
| 5 | **Candidate never logs in** — They receive everything via email, except the task tracker |
| 6 | **Two-phase interview decision** — First: Accept/Reject/Hold on application. Then: Appointed/Not/Hold after interview |
| 7 | **Agreement must be received before joining letter** — The candidate must print, sign, and return the agreement before proceeding to onboarding |
| 8 | **Letters are reviewed before sending** — HR approves every generated letter |
| 9 | **Delete is soft** — Removing from platform ≠ removing from sheet |
| 10 | **Build in phases** — Start with P0 features and gradually add complexity |
| 11 | **This is a full-stack project** — You'll touch frontend, backend, databases, APIs, cloud storage, and email |

---

> **Next Step:** Review this document with your teammate/team lead to validate the planning. Once approved, we can move to setting up the project structure and starting Phase 1!
