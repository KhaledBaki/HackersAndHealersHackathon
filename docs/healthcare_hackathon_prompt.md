# CLAUDE PROMPT — PrimaryCare AI Follow-Up Dashboard

---

## SYSTEM ROLE

You are a senior full-stack developer and UX designer building a production-grade healthcare web application for the **AI in Healthcare Co-Design Hackathon** (University of Ottawa / TOH / Bruyère Health Innovation). The product is called **PulseChart** — a clinician-facing dashboard that dramatically reduces follow-up and documentation burden in primary care.

Your output must be a single, fully self-contained `pulsechart.html` file with embedded CSS and JavaScript. No backend, no server — all logic runs client-side using in-memory state simulating a Firebase + Python backend. Use synthetic patient data only (no real patient records).

---

## PROJECT CONTEXT

Primary care physicians lose ~19 hours/week to administrative tasks. The core problem: after a patient visit, follow-up tasks — test results, referrals, medication reminders, and care plan steps — fall through the cracks because no system reliably converts visit decisions into downstream actions.

**PulseChart solves this** by:
1. Ingesting a synthetic patient dataset (conditions, medications, labs, procedures, encounters, immunizations)
2. Running a priority-scoring algorithm to rank which patients need follow-up most urgently
3. Letting doctors manage a prioritized follow-up queue, categorize by work type, route to pharmacist, and send follow-ups (auto or custom)
4. Displaying a Confirmation Log tab showing all sent follow-ups for the session

---

## TECH STACK (SIMULATED)

- **Frontend**: Vanilla HTML/CSS/JS (single file)
- **Data**: Inline synthetic patient dataset (15–20 patients) modeled after Synthea FHIR structure
- **State**: In-memory JS objects (simulating Firebase Firestore)
- **Python backend (simulated)**: Describe in UI comments that severity scoring and queue population logic runs via a Python microservice — simulate it inline in JS
- **Fonts**: Fontshare — `Satoshi` for body, `Cabinet Grotesk` for headings
- **Icons**: Lucide Icons via CDN
- **Charts**: Chart.js via CDN (severity distribution chart)

---

## DESIGN SYSTEM

### Colors (Medical-grade, trustworthy, accessible)
Use a custom healthcare palette — NOT purple gradients or neon. Think: clean, clinical, calm.

```css
:root {
  /* Surfaces */
  --color-bg: #f0f4f8;
  --color-surface: #ffffff;
  --color-surface-2: #f7f9fc;
  --color-surface-offset: #edf1f7;
  --color-border: rgba(30, 60, 100, 0.10);
  --color-divider: rgba(30, 60, 100, 0.08);

  /* Text */
  --color-text: #0f2340;
  --color-text-muted: #5a7090;
  --color-text-faint: #98afc5;
  --color-text-inverse: #f0f4f8;

  /* Primary (Deep Medical Teal) */
  --color-primary: #0a7b82;
  --color-primary-hover: #075f65;
  --color-primary-active: #054549;
  --color-primary-highlight: #d0eced;

  /* Severity palette — used ONLY for priority indicators */
  --color-critical: #c0392b;       /* CRITICAL — immediate action */
  --color-critical-bg: #fdf0ee;
  --color-high: #d35400;           /* HIGH — urgent */
  --color-high-bg: #fef5ee;
  --color-medium: #d4a017;         /* MEDIUM — soon */
  --color-medium-bg: #fefaee;
  --color-low: #27ae60;            /* LOW — routine */
  --color-low-bg: #edfaf3;

  /* Routing */
  --color-pharmacist: #6c3483;
  --color-pharmacist-bg: #f5eef8;
  --color-bloodwork: #1a5276;
  --color-bloodwork-bg: #eaf2fb;
  --color-surgery: #922b21;
  --color-surgery-bg: #fdedec;

  /* Success / Sent */
  --color-success: #1e8449;
  --color-success-bg: #eafaf1;
}
```

### Typography
- **Display font**: `Cabinet Grotesk` (headings ≥18px, page titles)
- **Body font**: `Satoshi` (all UI text, labels, data)
- **Data font**: `font-variant-numeric: tabular-nums lining-nums` on all numbers/timestamps
- Body: 14px base for UI chrome, 16px for reading content
- NO text below 12px

### Layout
- **Left sidebar** (240px): logo, navigation, quick stats
- **Top header bar**: current view title, search, user avatar, dark mode toggle
- **Main content area**: full scroll, single scroll region
- Mobile-responsive: sidebar collapses to hamburger at <768px

---

## SYNTHETIC PATIENT DATASET

Create 18 synthetic patients with the following structure per patient:

```js
{
  id: "PT-001",
  name: "Marie Tremblay",
  age: 67,
  gender: "F",
  lastVisit: "2026-05-14",
  conditions: ["Type 2 Diabetes", "Hypertension", "CKD Stage 2"],
  medications: ["Metformin 1000mg", "Ramipril 10mg", "Rosuvastatin 20mg"],
  labResults: { hba1c: 8.9, egfr: 48, ldl: 3.2, bp: "152/94" },
  immunizations: { flu: "2025-10", pneumococcal: "2022-03" },
  lastProcedure: "Annual Diabetic Foot Exam",
  followUpDue: true,
  followUpReason: "HbA1c elevated — medication adjustment needed",
  severityScore: 87,   // 0–100, computed by scoring algorithm
  category: "bloodwork",  // "bloodwork" | "medication" | "surgery" | "referral" | "routine"
  routedTo: null,  // null | "pharmacist" | "specialist"
  followUpSent: false,
  followUpDate: null,
  followUpSetBy: null,
  followUpNote: "",
  priority: "critical"  // "critical" | "high" | "medium" | "low"
}
```

**Severity scoring algorithm (simulate in JS, describe as Python microservice):**
Score = weighted sum of:
- Active conditions count × 8 (max 40)
- Lab values out of normal range × 10 each (max 30)
- Days since last visit ÷ 3 (max 20)
- Missing/overdue immunizations × 5 (max 10)
- Mapped to priority: 75–100 = critical, 50–74 = high, 25–49 = medium, 0–24 = low

Make patients diverse in age (22–84), conditions, severity levels — ensure at least 3 critical, 5 high, 6 medium, 4 low. Include realistic Canadian names (mix of French, English, and other).

---

## APPLICATION STRUCTURE — 5 VIEWS

Build tab/hash-based routing (`location.hash`). Five views accessible from the sidebar:

---

### VIEW 1: DASHBOARD (default `#dashboard`)

**Header KPI row (4 cards):**
- Total Patients in Queue (with sparkline)
- Critical / High Priority count (red badge)
- Follow-Ups Sent Today
- Avg. Severity Score

**Priority Queue Table (main content):**
Sortable table with columns:
- Patient Name + age/gender chip
- Conditions (first 2 shown + "+N more" tooltip)
- Priority badge (color-coded: CRITICAL / HIGH / MEDIUM / LOW)
- Category chip (🔬 Blood Work / 💊 Medication / 🔪 Surgery / 📋 Referral / ✅ Routine)
- Severity Score (animated number, progress bar behind it)
- Last Visit (relative: "42 days ago")
- Follow-Up Due date
- Actions column: [Send Auto Follow-Up] [Custom Message ✏️] [Route →] [Set Date 📅]

**Filters bar (above table):**
- Filter by Priority (All / Critical / High / Medium / Low)
- Filter by Category (All / Blood Work / Medication / Surgery / Referral / Routine)
- Filter by Routing (All / Unrouted / Pharmacist / Specialist)
- Search by patient name
- Sort by: Severity Score ↓ (default), Name, Last Visit, Due Date

**Probability Distribution Panel (right side or below table):**
A Chart.js bar chart showing the distribution of patients by severity score buckets (0–24, 25–49, 50–74, 75–100). Title: "Queue Severity Distribution". Subtitle: "Simulating Python probability model (Firebase microservice)". Use the severity color palette for bars.

---

### VIEW 2: PATIENT DETAIL (`#patient/{id}` — open by clicking a row)

Full patient profile card showing:
- Patient header: name, age, gender, photo placeholder (initials avatar)
- Conditions list with severity indicators
- Medications list with last refill date
- Recent Lab Results table (value, normal range, status badge: IN RANGE / ELEVATED / CRITICAL)
- Immunizations with overdue flag
- Last 3 encounters (date, reason, outcome)
- Follow-Up Section:
  - Current priority badge + severity score
  - Category selector (doctor can change)
  - Priority override (doctor can bump up/down)
  - Routing option: [Keep with Doctor] [Route to Pharmacist] [Route to Specialist]
  - Follow-Up Date picker (simulated date input)
  - Message options: [Auto Follow-Up] [Custom Message]
  - Custom message textarea (shows when custom selected): pre-filled template based on conditions
  - [Send Follow-Up] button → adds to Confirmation Log

---

### VIEW 3: PHARMACIST INTERFACE (`#pharmacist`)

Shows only patients routed to pharmacist. Two columns:
- **Left**: Routed patient list (name, conditions, medications, priority badge)
- **Right**: Medication action panel for selected patient:
  - Medication review table (current meds, interaction flags, refill status)
  - "Deprescribing Suggestion" section (generated text based on polypharmacy check)
  - Action buttons: [Request Coverage Check] [Flag Drug Interaction] [Approve Refill] [Send Reminder]
  - Message composer with pharmacist signature

Include a stat at top: "X patients routed to pharmacist | Y pending action"

---

### VIEW 4: ADD PATIENT (`#add-patient`)

A clean multi-step form (3 steps with animated step indicator):

**Step 1 — Basic Info:**
- Full Name, Date of Birth, Gender, Health Card Number
- Primary Physician (dropdown)

**Step 2 — Clinical Info:**
- Conditions (multi-select chips + add custom)
- Current Medications (multi-input rows)
- Last Visit Date
- Reason for Follow-Up

**Step 3 — Priority Settings:**
- Category selector (Blood Work / Medication / Surgery / Referral / Routine)
- Initial Priority (auto-calculated preview shows score)
- Follow-Up Date (date picker)
- Routing Option (Doctor / Pharmacist / Specialist)
- Notes textarea

On [Add to Queue] submit: animate patient card into the queue, show success toast, redirect to dashboard with new patient highlighted.

---

### VIEW 5: CONFIRMATION LOG (`#confirmations`)

**This is the simulation output tab.** Every sent follow-up appears here.

**Log table columns:**
- Timestamp (hh:mm:ss, animate on new entry)
- Patient Name
- Priority badge
- Message Type (AUTO / CUSTOM)
- Category
- Routed To (Doctor / Pharmacist / Specialist)
- Message Preview (first 80 chars, expandable)
- Status badge: SENT ✓ (green) / PENDING ⏳ / FAILED ✗

**Top of page:**
- "X follow-ups sent this session" counter (animated)
- "X auto | X custom" breakdown
- [Export Log CSV] button (downloads in-memory log as CSV)
- [Clear Log] button (with confirm dialog)

**Empty state:**
Beautiful empty state: medical clipboard icon, "No follow-ups sent yet. Select a patient and send your first follow-up." with an arrow pointing toward the dashboard.

**Auto-demo mode:**
Include a [Run Demo] button that simulates sending 5 follow-ups over 3 seconds with staggered animations to show the system in action for demo/presentation purposes.

---

## INTERACTION DETAILS

### Follow-Up Flow (Critical UX)

When doctor clicks [Send Auto Follow-Up] on any patient:
1. Button shows loading spinner for 800ms (simulating Firebase write)
2. Patient row updates: follow-up status chip changes to "SENT ✓" with green animation
3. Toast notification: "Follow-up sent to [Patient Name]" slides in from top-right
4. Entry appears in Confirmation Log in real time
5. Dashboard KPI "Follow-Ups Sent Today" increments with animation

When doctor clicks [Custom Message ✏️]:
1. A side drawer slides in from the right (not a modal — keeps table context visible)
2. Shows: patient summary header, pre-filled message template based on conditions
3. Doctor can edit the message freely
4. Priority dropdown to override
5. Route toggle: Doctor / Pharmacist / Specialist
6. [Send] button triggers same flow as auto above

### Priority Adjustment
Doctor can click any priority badge to get a small popover with 4 options (Critical/High/Medium/Low). Selecting one:
- Animates the badge color change
- Re-sorts the table if sort is by priority
- Logs the change: "Priority changed: HIGH → CRITICAL by Dr. [User]"

### Routing Flow
[Route →] button opens a small popover:
- [Keep with Doctor] (default)
- [Route to Pharmacist 💊] (purple badge on patient row, patient appears in pharmacist view)
- [Route to Specialist 🏥]
After routing, row gets a routing chip.

---

## SIDEBAR NAVIGATION

```
🏠  Dashboard          (active state: filled background, accent border-left)
👤  Patients           (→ #patient list view, shortcut to search)
💊  Pharmacist         (shows badge count of routed patients)
➕  Add Patient
📋  Confirmation Log   (shows badge count of sent follow-ups)

--- divider ---

⚙️  Settings (stub — not fully implemented)
```

Bottom of sidebar:
- User card: "Dr. Sarah Chen | Family Medicine" with initials avatar
- Version: "PulseChart v1.0 | Hackathon Build"

---

## ANIMATIONS & MICRO-INTERACTIONS

- **Table rows**: fade-in with `opacity` + `translateY(8px)` stagger (20ms per row)
- **KPI numbers**: count-up animation on page load (600ms)
- **Priority badges**: smooth color transitions on change (200ms)
- **Severity score bars**: fill animation on load (400ms ease-out)
- **New log entries**: slide-in from right with green flash
- **Toast notifications**: slide in from top-right, auto-dismiss after 3s
- **Side drawer**: `translateX(100%)` → `translateX(0)` with `cubic-bezier(0.16, 1, 0.3, 1)` (300ms)
- **Step form**: crossfade between steps (opacity + slight translate)
- Respect `prefers-reduced-motion`

---

## SVG LOGO

Design an inline SVG logo for "PulseChart":
- A stylized ECG pulse line (single clean wave) that ends in a small upward chart bar
- Color: `var(--color-primary)` (teal)
- Works at 24px and full size
- Clean, minimal, professional — no drop shadows or gradients

---

## QUALITY REQUIREMENTS

- Full light AND dark mode with toggle (sun/moon icon in top bar)
- WCAG AA contrast on all text/surface combinations
- Keyboard navigable (tab through table, Enter to open patient)
- Semantic HTML: `<nav>`, `<main>`, `<table>`, `<thead>`, proper `aria-label`s
- Every state handled: loading skeleton, empty state, error state, success state
- No text below 12px; body text at 14px (UI) / 16px (content)
- Mobile: sidebar collapses to hamburger at 768px, table gets card view at 480px
- Numbers use `font-variant-numeric: tabular-nums`
- `content-visibility: auto` on off-screen table rows

---

## TONE & AESTHETIC

- **NOT**: purple gradients, glowing orbs, neon, card borders in bright colors
- **YES**: clean white cards on soft blue-gray background, teal accents only on CTAs and active states, severity colors only for priority indicators, generous whitespace, subtle shadows
- Feel: Linear + Stripe + a medical records system — trustworthy, fast, purposeful
- One "wow moment": when the Confirmation Log receives a new entry, a subtle green pulse radiates from the row

---

## WHAT NOT TO DO

- Do NOT add a login/auth screen (skip to dashboard directly)
- Do NOT use colored side borders on cards (use surface elevation instead)
- Do NOT center-align all headings (left-align UI content)
- Do NOT use emoji as primary design elements (use Lucide icons)
- Do NOT make the severity chart's colors random — they must use the exact severity palette
- Do NOT add placeholder "Lorem ipsum" text — all content must be medically realistic
- Do NOT create separate HTML files — everything in one `pulsechart.html`

---

## FINAL DELIVERABLE

Output a single `pulsechart.html` file:
- Fully functional with all 5 views
- 18 synthetic patients loaded in-memory
- All interactions working (send follow-up, custom message drawer, routing, add patient, confirmation log)
- Both light and dark mode
- Responsive at 375px and 1280px
- Ready to open in any modern browser with zero dependencies to install

The prototype must be impressive enough that a clinician judge watching a 3-minute demo immediately understands the value and says "I would actually use this."

