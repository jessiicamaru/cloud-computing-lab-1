As a **WEP user (Manager, Functional Leader, HRVP Admin, or Employee)**, I want **to access a centralized archive of completed performance reviews from all closed cycles, with visibility governed by my role and my relationship to the reviewed employee** so that **I can reference historical performance data, track long-term trajectories, and make informed decisions without requesting records from HR**.

**Summary:** The **Review Archive** is a standalone sidebar menu item accessible to all authenticated WEP users. It provides a browsable, searchable, and filterable view of completed reviews from all closed review cycles. The archive enforces **asymmetric visibility rules**: a current manager sees the full review history of their current direct reports (including reviews conducted by prior managers), while a former manager sees only the reviews they personally conducted. The same rules apply to archived 360 feedback summaries. Functional Leaders see all historical records for employees in their department — regardless of which manager conducted the review or which department the employee was in at the time. HRVP Admins have unrestricted access to all records across all departments and cycles, including records for departed employees. Employees can view only their own historical self-assessments — they cannot see manager assessments or 360 feedback unless the manager explicitly shares a record with them. Individual review records can be exported as PDF. Reviews for departed employees are visible to HRVP Admins only.

> **Note:** All employee names, scores, cycle years, and review content referenced in this story are illustrative examples.

---

### Functional Requirements

#### A. Archive Navigation & Layout

- The Review Archive is a **standalone sidebar menu item** labeled "Review Archive" with an archive icon, visible to all authenticated WEP users (Manager, Functional Lead, HRVP Admin, Employee).
- Clicking it opens the archive landing page containing:
  - **Cycle/Year filter** (see Section B).
  - **Search bar** (see Section B).
  - **Results table** showing all review records the current user has access to, based on their role and relationship rules.
- The archive is read-only — no review content can be edited from the archive. All records are displayed in view-only mode.

#### B. Cycle & Year Filter and Search

- **Cycle/Year filter:** A dropdown or tab selector listing all closed review cycles in reverse chronological order (most recent first). Options include "All Cycles" (default — shows records from every closed cycle) and individual cycle entries (e.g., "2025 Year-End," "2024 Year-End," "2023 Year-End").
- **Search bar:** A text input allowing the user to search by employee name, manager name, or department. Search filters the results table in real time.
- **Department filter (HRVP Admin and Functional Lead only):** A dropdown to scope results by department. Hidden for Manager and Employee roles.
- When filters or search are applied, the results table updates immediately.

#### C. Results Table

- The results table displays one row per archived review record with columns:
  - **Employee** (name with avatar initials).
  - **Reviewing Manager** (the manager who conducted the review in that cycle).
  - **Department** (the employee's department at the time of the review).
  - **Cycle / Year** (e.g., "2025 Year-End").
  - **Final Rating** (numeric score with color-coded pill — same color scale as F21e Rating Distribution).
  - **Status** (COMPLETED / OVERDUE — reflecting the review's state at cycle close).
- **Sorting:** Default sort by Cycle/Year descending (most recent first), then by Employee name ascending. All columns are sortable by clicking headers.
- **Pagination:** If the result set exceeds 50 rows, the table supports pagination or virtual scrolling.

#### D. Archive Record Card (Expanded View)

- Clicking a row in the results table opens an **Archive Record Card** — a detailed read-only view of the completed review. The card contains expandable sections:
  - **Review Summary Header:** Employee name, reviewing manager, department (at time of review), cycle/year, final rating, and review status.
  - **Employee Self-Assessment** (expandable, collapsed by default): The employee's locked self-assessment text as submitted.
  - **Manager Year-End Assessment** (expandable, collapsed by default): The manager's evaluation text, competency ratings, and final rating as finalized. **Visibility rules apply** — see Sections E and I.
  - **360 Peer Feedback Summary** (expandable, collapsed by default): The archived 360 feedback cards (reviewer name, title, department, feedback text). Marked "CONFIDENTIAL." **Visibility rules apply** — see Sections F and I.
  - **HR Send-Back History** (expandable, collapsed by default, HRVP Admin only): If the review went through the HR Review Gate (F22a), the send-back feedback notes and lock action are shown. Hidden for all other roles.
- **Export to PDF button:** Available on each record card (see Section J).

#### E. Manager Visibility Rules

- The archive applies **asymmetric visibility** based on the manager's relationship to the employee: **Rule 1 — Current Manager (full history access):**
  - If the employee is **currently** a direct report of the viewing manager (per the live Entra ID org chart), the manager can see **all** historical reviews for that employee across all closed cycles — including reviews conducted by prior managers.
  - The current manager can view the full record card: self-assessment, manager assessment, and 360 feedback summary for every archived cycle.

  **Rule 2 — Former Manager (own records only):**
  - If the employee is **no longer** a direct report of the viewing manager, the manager can see **only** the reviews they personally conducted (where they were the `reviewing_manager_id`).
  - The former manager can view the full record card for their own reviews (self-assessment, their manager assessment, and the 360 feedback they requested).
  - The former manager **cannot** see reviews conducted by other managers for the same employee — those records do not appear in their archive results at all.

  **Rule 3 — Manager assessment content visibility:**
  - Within any record the manager has access to (per Rules 1 or 2), the manager can see the full manager assessment text and the 360 feedback summary.

- **Access determination logic:**
  - For each archived assessment, check: Is the viewing user the `reviewing_manager_id` on this assessment? → **Yes** → grant access (Rule 2 baseline).
  - Then check: Is the employee a current direct report of the viewing user? → **Yes** → grant access to ALL of that employee's archived assessments (Rule 1 override).
  - If neither condition is met → **deny access**. The record does not appear in the results table.

#### F. 360 Review Archive Visibility

- The same asymmetric rules from Section E apply to archived 360 feedback: **Current Manager:**
  - Can view all historical 360 Peer Feedback Summaries for their current direct reports, including 360 feedback requested by prior managers in prior cycles.

  **Former Manager:**
  - Can view only the 360 Peer Feedback Summaries that **they requested** during cycles when they were the reviewing manager.
  - Cannot see 360 feedback requested by subsequent managers.

  **Functional Leader:**
  - Can view all historical 360 Peer Feedback Summaries for employees in their department (see Section G).

  **HRVP Admin:**
  - Full access to all 360 Peer Feedback Summaries, all employees, all cycles.

  **Employee:**
  - Cannot see any 360 feedback in the archive (see Section I).

- The 360 Peer Feedback Summary remains marked **"CONFIDENTIAL — Private to Manager & HR"** in the archive, just as it is in the active workspace.

#### G. Functional Leader Visibility

- The Functional Leader can view **all historical reviews** for employees **currently** in their department — regardless of:
  - Which manager conducted the review.
  - Which department the employee was in at the time of the review.
- Example: Employee A was in Finance in 2023 and transferred to Operations in 2024. The current Functional Leader of Operations can see Employee A's 2023 Finance review AND 2024 Operations review.
- **Functional Leader transition:** If a new Functional Leader takes over a department, they immediately inherit full access to all historical records for that department's current employees. The former Functional Leader loses access to records they did not personally conduct as a reviewing manager (they retain access to any reviews where they were the `reviewing_manager_id` per Section E, Rule 2).
- **Departed employees:** Functional Leaders cannot see records for departed employees — those are HRVP Admin-only (see Section H).
- The Functional Leader can view: self-assessment, manager assessment, and 360 feedback summary for all accessible records.
- The **department filter** dropdown is visible and pre-scoped to their department. They cannot view other departments' records.

#### H. HRVP Admin Visibility

- The HRVP Admin has **unrestricted access** to all archived reviews across all departments, all cycles, and all employees — including:
  - Reviews conducted by any manager.
  - 360 Peer Feedback Summaries from any cycle.
  - HR Send-Back History (F22a feedback and lock actions).
  - **Departed employee records:** Reviews for employees who have left the organization (Entra ID status = Inactive/Terminated) remain visible in the archive for HRVP Admin only.
- The HRVP Admin can use the department filter, search, and cycle filter without any scope restrictions.
- Departed employees are visually distinguished in the results table with a badge: "DEPARTED" (neutral/gray badge) next to their name.

#### I. Employee Visibility

- Employees can access the Review Archive from the sidebar and view **only their own** historical reviews.
- **What the employee can see:**
  - Their own self-assessment text for each closed cycle.
  - A summary line showing the cycle year, final rating, and review status.
- **What the employee cannot see:**
  - Manager Year-End Assessment text — unless the manager has explicitly shared the record (see Section I.1).
  - 360 Peer Feedback Summary — never visible to the employee in the archive.
  - HR Send-Back History — never visible to the employee.
- The employee's archive view is a simplified list of their own reviews with expandable self-assessment sections and a final rating display. **I.1 — Manager Shares Review Record with Employee:**
  - From the Archive Record Card, a manager (current or former, for records they have access to) can click a **"Share with Employee"** action.
  - A confirmation dialog appears: "Share this review record with [Employee Name]? They will be able to view the manager assessment for the [Cycle Year] review. This cannot be undone."
  - Upon confirmation:
    - The employee gains read-only access to the **manager assessment section** of that specific archived record. The 360 Peer Feedback Summary and HR Send-Back History remain hidden.
    - A "SHARED" badge appears on the record in the employee's archive view.
    - The share action is logged in the audit trail (HRVP Admin only).
    - The share is **permanent and per-record** — it applies to that one specific cycle's review. It does not grant blanket access to all historical manager assessments.
  - The HRVP Admin and Functional Leader can also initiate a share on behalf of the manager if needed.

#### J. Export to PDF

- Each Archive Record Card has an **"Export to PDF"** button.
- The PDF contains:
  - A header with: Employee name, reviewing manager, department, cycle/year, final rating, and review status.
  - The review sections the current user has access to — respecting the same visibility rules as the on-screen view.
  - A "CONFIDENTIAL" watermark on pages containing 360 Peer Feedback Summary or HR Send-Back History.
  - A footer with: "Generated by [User Name] on [Date] · WEP Performance Intelligence."
- **Visibility-scoped export:** The PDF includes only the content the user can see. An employee's PDF contains only their self-assessment and final rating (plus manager assessment if shared). A manager's PDF contains the full record card. An HRVP Admin's PDF contains everything including HR Send-Back History.
- **File name format:** `WEP_Review_[EmployeeName]_[CycleYear].pdf` (e.g., `WEP_Review_MichaelZhang_2025.pdf`).
- The file is generated server-side and delivered as a browser download.

---

### Technical Requirements

- **Archive data source:** Archived reviews are read from the existing `assessments`, `360_responses`, `360_requests`, and `hr_assessment_feedback` tables, filtered to `cycle.status = 'Closed'`. No separate archive table is needed — the archive is a read-only view over existing data.
- **Access control query (Manager):**

  SELECT a.\* FROM assessments a
  JOIN employees e ON a.employee_id = e.id
  WHERE a.cycle_id IN (SELECT id FROM review_cycles WHERE status = 'Closed')
  AND (
  a.reviewing_manager_id = :current_user_id -- Rule 2: own records
  OR e.current_manager_id = :current_user_id -- Rule 1: current direct reports
  )

- **Access control query (Functional Lead):**

  SELECT a.\* FROM assessments a
  JOIN employees e ON a.employee_id = e.id
  WHERE a.cycle_id IN (SELECT id FROM review_cycles WHERE status = 'Closed')
  AND e.department = :current_user_department
  AND e.status = 'Active' -- Excludes departed employees

- **Access control query (HRVP Admin):**

  SELECT a.\* FROM assessments a
  WHERE a.cycle_id IN (SELECT id FROM review_cycles WHERE status = 'Closed')
  -- No department or manager filter — full access including departed employees

- **Access control query (Employee):**

  SELECT a.\* FROM assessments a
  WHERE a.employee_id = :current_user_id
  AND a.cycle_id IN (SELECT id FROM review_cycles WHERE status = 'Closed')

- **Manager assessment visibility for Employee:** The `assessments` table has a new boolean column `shared_with_employee` (default false). When a manager shares a record, this flag is set to true. The Employee API response includes the manager assessment text only when `shared_with_employee = true`.
- **360 archive query:** Same access rules — join `360_responses` with `360_requests` and filter by the same role-based conditions. For former managers, filter `360_requests.requesting_manager_id = :current_user_id`.
- **Departed employees:** The employee table has a `status` column (Active / Inactive / Terminated). The archive query for HRVP Admin does not filter on status. All other roles filter `e.status = 'Active'`.
- **Share audit:** Share actions are logged in `hr_assessment_audit` with `action_type = 'share_with_employee'`.
- **PDF generation:** Server-side PDF rendering using a templating engine. The API endpoint validates the user's access to the requested record before generating the PDF. The PDF content is assembled based on the same role-based visibility rules.
- **Performance:** Archive queries may span many cycles and thousands of records. Results are paginated server-side (50 per page). Indexes on `assessments(reviewing_manager_id)`, `assessments(employee_id)`, `employees(current_manager_id)`, and `employees(department)` are critical for query performance.

### Special Cases & Edge Case Handling

#### SC1: Employee Transfers Between Departments Across Multiple Cycles

**Scenario:** Employee A was in Finance (2022), transferred to Operations (2023), then to Deal Team (2024). The current Functional Leaders of each department view the archive.

**Expected behavior:**

- The **current** Functional Leader of Deal Team (Employee A's current department) can see all 3 reviews (2022, 2023, 2024) — because Employee A is currently in their department, they inherit the full history regardless of prior departments.
- The Functional Leader of Operations can see Employee A's records **only if Employee A is still in Operations**. Since Employee A has moved to Deal Team, the Operations Functional Leader sees nothing for Employee A (Employee A is no longer in their department).
- The Functional Leader of Finance similarly sees nothing for Employee A.
- The HRVP Admin sees all 3 records regardless of department.

---

#### SC2: Manager Transfers to a Different Department but Retains Former Direct Reports' Records

**Scenario:** Manager B conducted reviews for 3 employees in Operations during 2023. In 2024, Manager B transferred to the Deal Team. Those 3 employees now report to Manager C.

**Expected behavior:**

- Manager B can still see the 2023 reviews they conducted (Rule 2 — own records). These records remain in their archive permanently.
- Manager B cannot see the 2024 reviews conducted by Manager C.
- Manager C (current manager) can see both the 2023 reviews (conducted by Manager B) and the 2024 reviews (conducted by themselves) for those 3 employees (Rule 1 — current direct reports).

---

#### SC3: Former Functional Leader Loses Department-Wide Access

**Scenario:** Sarah was the Functional Leader for Operations until June 2025. David became the new Functional Leader in July 2025.

**Expected behavior:**

- David (new Functional Leader) immediately inherits full access to all historical reviews for all Operations employees — including reviews from before his appointment.
- Sarah (former Functional Leader) loses department-wide access. She retains access only to reviews where she was the `reviewing_manager_id` (if she personally conducted any reviews).
- If Sarah is also a Manager with current direct reports, she retains access to her current direct reports' full history per Rule 1.

---

#### SC4: Departed Employee Records

**Scenario:** Employee X left the organization in 2024. Their reviews from 2022, 2023, and 2024 exist in the archive.

**Expected behavior:**

- **HRVP Admin:** Can see all 3 reviews. Employee X appears with a "DEPARTED" badge.
- **Functional Leader:** Cannot see Employee X's records (departed employees are excluded for Functional Leaders).
- **Former Manager (who conducted the review):** Can still see the reviews they personally conducted for Employee X, even though Employee X has departed. The `reviewing_manager_id` match still grants access.
- **Current Manager:** Not applicable — departed employees have no current manager in the org chart.

---

#### SC5: Employee Views Archive but Has No Closed Cycle Reviews

**Scenario:** A new employee joined after the most recent cycle closed and has no historical reviews.

**Expected behavior:**

- The archive displays an empty state: "No review records found. Your archived reviews will appear here after your first review cycle is completed."
- The sidebar menu item is still visible — it is not hidden for new employees.

---

#### SC6: Manager Attempts to Share a Record They Don't Have Access To

**Scenario:** A former manager somehow navigates to a record conducted by a subsequent manager (e.g., via a direct URL or stale bookmark).

**Expected behavior:**

- The API validates access before rendering the record. If the user is not the `reviewing_manager_id` and the employee is not their current direct report, the API returns a 403.
- The UI displays: "You do not have access to this review record."
- The "Share with Employee" action is never available on records the user cannot view.

---

#### SC7: Multiple Managers Share Different Cycle Records with the Same Employee

**Scenario:** Manager B shares the 2023 review with Employee A. Manager C shares the 2024 review with Employee A. No one shares the 2025 review.

**Expected behavior:**

- Employee A sees 3 records in their archive: 2023, 2024, 2025.
- The 2023 record shows: self-assessment + manager assessment (SHARED badge).
- The 2024 record shows: self-assessment + manager assessment (SHARED badge).
- The 2025 record shows: self-assessment only (no share, no manager assessment visible).
- Each share is independent and per-record.

---

#### SC8: HRVP Admin or Functional Leader Shares a Record on Behalf of a Manager

**Scenario:** The reviewing manager is unavailable, and the Functional Leader wants to share the record with the employee.

**Expected behavior:**

- Both HRVP Admin and Functional Leader can initiate the "Share with Employee" action on records they have access to.
- The confirmation dialog is the same.
- The audit trail logs the actual user who performed the share (not the reviewing manager).
- The employee sees the manager assessment regardless of who initiated the share.

---

#### SC9: PDF Export for a Record with Shared Manager Assessment

**Scenario:** Employee A exports a PDF of their 2024 review, which has been shared by the manager.

**Expected behavior:**

- The PDF includes: self-assessment, manager assessment (since it was shared), and final rating.
- The PDF does not include: 360 Peer Feedback Summary or HR Send-Back History (never visible to employees).
- The "SHARED" badge is noted in the PDF header: "Manager assessment shared on [date]."

---

#### SC10: Archive Query Performance with Many Cycles and Employees

**Scenario:** The organization has 500 employees and 5 closed cycles, resulting in ~2,500 archived records.

**Expected behavior:**

- Server-side pagination returns 50 records per page.
- Filters (cycle, department, search) are applied at the database level, not client-side.
- Indexes on `reviewing_manager_id`, `employee_id`, `current_manager_id`, and `department` ensure query response times remain under 500ms.
- The PDF export for a single record generates within 5 seconds.

---

#### SC11: Share Action Is Irreversible

**Scenario:** A manager shares a review record with an employee, then regrets it and wants to revoke the share.

**Expected behavior:**

- The share is **permanent and cannot be undone.** There is no "Unshare" action.
- The confirmation dialog explicitly warns: "This cannot be undone."
- If the share was made in error, the HRVP Admin can note the issue but cannot revoke the employee's access to the shared manager assessment through the system.
- This permanence is intentional — it prevents confusion where an employee sees content that is later hidden.
