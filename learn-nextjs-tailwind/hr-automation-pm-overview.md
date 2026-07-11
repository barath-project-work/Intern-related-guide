# 📋 HR Automation Platform — Project Overview (For Project Manager)

> **Document Purpose:** This is a simple, non-technical summary of the HR Automation platform.  
> It explains what the system does, how it works from a business perspective,  
> and what value it brings to the team.

---

## 🎯 What Is This Project?

The **HR Automation Platform** is an internal web-based tool that helps HR manage the complete internship lifecycle — from the moment a candidate applies, all the way through to when they complete their internship and receive their completion certificates.

The goal is to **reduce manual effort** by automating:
- Email notifications
- Document management
- Letter generation
- Task tracking
- File organization in Google Drive

---

## 👥 Who Will Use It?

| Person | What They Can Do |
|--------|-----------------|
| **HR / Admin** | Login to the platform, manage applications, schedule interviews, send/receive agreements, upload documents, generate letters, track intern progress |
| **Candidate / Intern** | Receive all communication via email (no login required). Can view their own task tracker after joining |

---

## 🧩 What Are the Features?

### 1. 🔐 Login System
HR logs into the platform using an email ID and password. Only authorized HR personnel can access the system.

---

### 2. 📋 Application Management (Connected to Google Sheets)
- Applications are pulled in from a **Google Sheet** that HR maintains
- HR sees all applications in one dashboard
- For each application, HR can:
  - ✅ **Accept** — Move the candidate forward
  - ❌ **Reject** — Decline the application
  - ⏸ **Hold** — Keep aside for later decision
  - 🗑 **Delete** — Remove from the platform only (the Google Sheet is not affected)
- HR can also **manually add** applications for candidates who apply via email
- Any action taken on the platform is **synced back to the Google Sheet**

---

### 3. 🎤 Interview Scheduling
When a candidate is accepted, HR can schedule an interview:
- Enter the **date & time**
- Add a **Google Meet link**
- Click "Schedule" → the candidate **automatically receives an email** with the details

After the interview, HR marks the outcome:
- **Appointed** → Move forward
- **Not Appointed** → Rejected
- **Hold** → Kept for later decision (can still Appoint or Reject later)

---

### 4. 🤝 Agreement Process (Send → Sign → Receive)
After the candidate is appointed, this is a **critical step**:

1. **HR sends an agreement letter** via email to the candidate
2. The agreement PDF has **blank spaces** for the candidate's name and signature
3. The candidate **prints the agreement, signs it by hand**, and **scans/takes a photo** of it
4. The candidate **emails the signed copy back** to HR
5. HR **uploads the signed copy** to the platform
6. The platform shows a **green "Agreement Received"** badge
7. Only after this is done can HR proceed to the next step

> 💡 **Why this matters:** This replaces the messy process of chasing agreements via email manually. HR can see at a glance whose agreement is pending.

---

### 5. 📄 Joining Letter Generation
Once the signed agreement is received:
- HR clicks "Generate Joining Letter"
- The system **auto-generates** a professional letter with:
  - Candidate's full name
  - Date of joining
  - Company address
  - Role details
- The letter template is designed in **Canva** and uploaded to the platform
- HR **reviews and approves** the letter
- Once approved, the letter is **emailed to the candidate** automatically
- A copy is saved in Google Drive

---

### 6. 📁 Document Upload
HR uploads important candidate documents:
- ✅ Signed Agreement Letter (from the agreement process above)
- ✅ Marksheet
- ✅ Aadhar Card

All documents are **automatically stored in organized folders** in Google Drive.

---

### 7. 📊 Task Tracker (Per Candidate)
Each intern gets their own task tracking system:
- An Excel sheet is created inside a folder named `[Candidate Name] working/`
- HR can assign tasks with dates and descriptions
- The system tracks **number of working days**
- This sheet is **only visible to the candidate** (they can view, not edit)

---

### 8. 📜 Certificate & Letter of Recommendation (LOR)
When the internship ends, if the intern has completed successfully:
- System generates an **Experience Certificate**
- System generates a **Letter of Recommendation (LOR)**
- Both are saved in Google Drive in a certificates folder

If the internship is terminated early, no certificates are generated.

---

### 9. ✏️ Edit Options
HR can edit any information about any candidate at any time — application details, interview notes, documents, etc.

---

## 🔄 How the Full Process Flows

```
1. Applications arrive from Google Sheet
           ↓
2. HR reviews → Accept / Reject / Hold
           ↓
3. (If Accepted) → Schedule Interview
           ↓
4. Interview happens → Appointed / Not Appointed / Hold
           ↓
5. (If Appointed) → Send Agreement to Candidate via Email
           ↓
6. Candidate signs and sends back → HR marks as Received
           ↓
7. Upload Documents (signed agreement, marksheet, aadhar)
           ↓
8. Generate & Approve Joining Letter → Sent to Candidate
           ↓
9. Internship Begins → Task Tracker Active
           ↓
10. Internship Ends → Generate Experience Certificate + LOR
```

---

## 🔗 What Does the System Connect To?

| Connected To | What It Does |
|-------------|-------------|
| **Google Sheets** | Reads applications from a sheet and writes actions back (two-way sync) |
| **Google Drive** | Stores all documents, agreements, letters, and task trackers in organized folders |
| **Google Meet** | Creates meeting links for interviews |
| **Email System** | Sends automated emails to candidates (interview invites, agreements, joining letters, etc.) |
| **Canva** | HR designs the joining letter template in Canva and uploads it to the platform |

---

## 📁 How Files Are Organized in Google Drive

```
HR Drive (Main Folder)
│
├── Applications                  (Linked to Google Sheet)
├── Candidates/
│   └── [Each Candidate Name]/
│       ├── Signed Agreement
│       ├── Marksheet
│       └── Aadhar Card
├── Joining Letters               (All approved letters)
├── Intern Tasks/
│   └── [Candidate Name] working/ (Task Tracker Excel)
└── Certificates/
    ├── Experience Certificates
    └── Letters of Recommendation
```

All folders are **created automatically** by the system — no manual folder setup needed.

---

## 📅 Suggested Build Phases

| Phase | What Gets Built | Approx. Time |
|-------|----------------|-------------|
| **Phase 1 — Foundation** | Login system, application dashboard, accept/reject/hold/delete, Google Sheet connection | Week 1–2 |
| **Phase 2 — Core Features** | Interview scheduling, email automation, document upload, Google Drive integration, manual entry | Week 3–4 |
| **Phase 3 — Advanced** | Joining letter generation, task tracker, working days calculation, edit options, Canva template upload | Week 5–6 |
| **Phase 4 — Completion** | Certificate & LOR generation, termination flow, candidate view, testing & launch | Week 7–8 |

---

## ✅ Key Things to Know

| # | Point |
|---|-------|
| 1 | The **Google Sheet is the starting point** — all applications come from there |
| 2 | Google Sheet and the platform **stay in sync** — actions on one reflect on the other |
| 3 | The system **automates emails** — candidates get notified without HR having to type anything |
| 4 | Candidates **do not log in** to the platform — everything happens via email |
| 5 | The **agreement process is offline** — print, sign, scan, email back |
| 6 | **Agreement must be received** before the joining letter can be generated |
| 7 | All letters go through **HR review and approval** before being sent |
| 8 | Google Drive folders are **created automatically** — no manual organization needed |
| 9 | Deleting from the platform **does not delete from Google Sheet** |
| 10 | The system is built in **phases** — each phase adds more features on top of the previous one |

---

> **Next Step:** Review this document with the team and stakeholders to confirm the requirements.  
> Once approved, the development team can begin setting up the project.

---

*This document is for project management review. A separate technical document exists with detailed implementation specifications.*
