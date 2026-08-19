As a **User (any role)**, I want **the sidebar navigation to show only the menu items relevant to my role** so that **I see a clean, uncluttered interface tailored to my responsibilities and I am not confused by pages I cannot access**.

**Summary:** The sidebar navigation is dynamically configured based on the user's active role. Employees see a focused set of items for their own review process. Managers see their employee-level items plus team-management tools — but a Manager with no direct reports loses those team tools (Team Overview, Manager Evaluation Center). HR Admins always see the full administrative interface, including the team tools, and are never gated on direct reports. Executives see organization-wide views plus the manager-assessment workspace, but not Team Overview. The menu updates on the next login or page refresh when roles or reporting lines change.

---

### Menu Configuration by Role

#### Employee (non-manager)

| # | Menu Item | Description |

|---|---|---|

| 1 | **Goal Setting** | Access personal goal creation, editing, and Goal Summary Cards. Badge counter shows draft goal count. |

| 2 | **My Review** | Access self-assessment form (mid-year and year-end). Badge shows completion status. |

| 3 | **My 360** | View and complete pending 360 feedback requests from other employees' managers. Badge shows pending request count. |

| 4 | **Org Chart** | View the organization hierarchy tree. |

**Not shown to employees:** Team Overview, Manager Evaluation Center, Outgoing 360 Review Management, HR VP Insights, Cycle Settings, User Management, 360 Reviewer Management, Assessment Exemptions.

---

#### Manager (has direct reports)

| # | Menu Item | Description |

|---|---|---|

| 1 | **Goal Setting** | Access personal goal creation + view team goals. |

| 2 | **My Review** | Access own self-assessment form. |

| 3 | **My 360** | View and complete pending 360 feedback requests. |

| 4 | **Team Overview** | View direct reports' review status, assessment queue, goal progress. Badge shows pending assessments count. |

| 5 | **Manager Evaluation Center** | Access the full evaluation workspace: locked self-assessment review, manager assessment form, 360 Peer Feedback Summary, sign & finalize, send feedback. Displayed as a separate, clearly visible tab. |

| 6 | **Outgoing 360 Review Management** | Consolidated view of all outgoing 360 requests across all direct reports. "Invite Peers to Review" button. |

| 7 | **Org Chart** | View the organization hierarchy tree. |

**Not shown to managers:** HR VP Insights, Cycle Settings, User Management, 360 Reviewer Management, Assessment Exemptions.

---

#### HR Admin

| # | Menu Item | Description |

|---|---|---|

| 1 | **Goal Setting** | Access personal goals (if HR Admin is also an employee in the cycle). |

| 2 | **My Review** | Access own self-assessment (if applicable). |

| 3 | **My 360** | View and complete pending 360 feedback requests (if applicable). |

| 4 | **Team Overview** | View team review status across the department. Always available to HR Admins — never gated on personal direct reports. |

| 5 | **Manager Evaluation Center** | Access the evaluation workspace (department-scoped). Always available to HR Admins — never gated on personal direct reports. |

| 6 | **Outgoing 360 Review Management** | Manage 360 requests for the department. Always available to HR Admins. |

| 7 | **HR VP Insights** | Organization-wide dashboards, KPIs, performance trends, AI insights, calibration view. |

| 8 | **Cycle Settings** | Review cycle management, phase dates, per-step deadlines, notification cadence configuration. |

| 9 | **User Management** | Manage user accounts, roles, and permissions. |

| 10 | **360 Reviewer Management** | Search and toggle employees active/inactive for 360 participation. Configure reviewer caps. |

| 11 | **Assessment Exemptions** | Manage the senior leadership exemption list. |

| 12 | **Org Chart** | View the organization hierarchy tree with full admin controls (export, sync, audit trail). |

| 13 | **Login Audit Log** | View employee login status (logged in / never logged in). |

---

#### Executive

| # | Menu Item | Description |

|---|---|---|

| 1 | **Goal Setting** | Access personal goals (when the goal-setting phase is open). |

| 2 | **My 360** | View and complete pending 360 feedback requests. |

| 3 | **Manager Evaluation Center** | Access the manager-assessment workspace. |

| 4 | **Executive Summary** | High-level organization performance summary, department comparison, AI strategic insights. |

| 5 | **Org Chart** | View the organization hierarchy tree. |

| 6 | **Review Archive** | Browse finalized reviews. |

**Not shown to executives:** Team Overview, My Review, and admin-only tools (Cycle Settings, User Management, 360 Reviewer Management, Assessment Exemptions, Login Audit Log).

_Note (BA 2026-07-23): executives retain every tab except Team Overview — earlier this section listed only Executive Summary + Org Chart, which no longer matches the confirmed decision._

---

#### Department Lead (HRVP view, scoped)

| # | Menu Item | Description |

|---|---|---|

| 1–7 | Same as Manager | All manager-level items. |

| 8 | **Department Insights** | Department-scoped dashboards: performance trends, at-risk flagging, development themes, strategic narrative — scoped to their department only. |

---

### Functional Requirements

- The sidebar navigation renders dynamically based on the authenticated user's **role** and **team structure** (whether they have direct reports).

- Menu items that the user does not have access to are **completely hidden** — not shown as grayed out or disabled. Users should never see items they cannot interact with.

- **Badge counters** are displayed on relevant menu items (Goal Setting, My Review, My 360, Team Overview, Manager Evaluation Center) showing pending action counts.

- If a user's role changes (e.g., promoted from Employee to Manager), the navigation updates on the next login or page refresh. No manual cache clearing is required.

- **Manager Evaluation Center** must be a **separate, clearly visible tab** in the manager sidebar — not hidden within another menu or only accessible from the HR view. This addresses the bug reported in CR-13 where this link was lost during a recent menu change.

- **Exempt users (B7):** If a user is assessment-exempt, their "My Review" menu item is replaced with a static message or hidden, depending on the exemption configuration. Their manager-level menu items remain visible.

---

### Technical Requirements

- Navigation configuration is role-based and stored as a mapping: role → list of menu items.

- The navigation component reads the user's role and `has_direct_reports` flag on each page load to determine which items to render.

- Badge counter values are fetched via lightweight API endpoints that return counts only (not full data).

- The navigation must be performant — menu rendering should complete within 200ms of page load.

- If a user has multiple roles (e.g., Manager + HR Admin), the navigation shows the **union** of all applicable menu items.

---

### Special Cases & Edge Case Handling

#### SC1: User Has Multiple Roles

**Scenario:** A user is both a Manager (has direct reports) and has been granted HR Admin access (e.g., Julie per CR-10).

**Expected behavior:**

- The navigation shows the union of all applicable menu items. Julie sees all Manager items plus all HR Admin items.
- There is no conflict — items from both roles are merged into a single sidebar.

---

#### SC2: Manager Loses All Direct Reports

**Scenario:** A Manager's last direct report is reassigned to another manager. The Manager now has zero direct reports.

**Expected behavior:**

- On the next login or page refresh, the Team Overview, Manager Evaluation Center, and Outgoing 360 Review Management items are hidden.
- The user's role remains "Manager" (role is not automatically changed), but the navigation adapts based on the `has_direct_reports` flag.
- If direct reports are later reassigned back, the items reappear.

---

#### SC3: New User Synced from Microsoft 365 — First Login

**Scenario:** A new employee is synced with the default "Employee" role and logs in for the first time.

**Expected behavior:**

- The navigation shows Employee-level items only: Goal Setting, My Review, My 360, Org Chart.
- If the user is later promoted to Manager and assigned direct reports, the navigation updates automatically on the next login.

---

#### SC4: Org Chart Visibility Decision (CR-12)

**Scenario:** The decision on whether to keep Org Chart in the employee menu is tentative (Kate's recommendation: yes, keep it).

**Expected behavior:**

- **Current implementation:** Org Chart is visible to all roles including Employees, since there is no org chart available elsewhere in the organization's systems.
- If the decision changes in the future, the Org Chart menu item can be removed from the Employee role's navigation configuration without a code change (configuration-level update).

AC

AC1: Employee sees only employee-level menu items

  Given the user is an Employee with no direct reports

  When they view the sidebar navigation

  Then they see: Goal Setting, My Review, My 360, and Org Chart

  And they do not see: Team Overview, Manager Evaluation Center, or any HR items

AC2: Manager sees employee + manager menu items

  Given the user is a Manager with direct reports

  When they view the sidebar navigation

  Then they see: Goal Setting, My Review, My 360, Team Overview, Manager Evaluation Center, Outgoing 360 Review Management, and Org Chart

  And they do not see: HR VP Insights, Cycle Settings, or User Management

AC3: Manager Evaluation Center is a separate visible tab

  Given the user is a Manager

  When they view the sidebar navigation

  Then "Manager Evaluation Center" appears as a clearly visible, separate menu item

  And it is not hidden within another menu or sub-section

AC4: HR Admin sees all administrative items

  Given the user is an HR Admin

  When they view the sidebar navigation

  Then they see: all employee/manager items (if applicable) plus HR VP Insights, Cycle Settings, User Management, 360 Reviewer Management, Assessment Exemptions, and Login Audit Log

AC5: Executive sees org-wide views + the manager-assessment workspace, but not Team Overview

  Given the user is an Executive

  When they view the sidebar navigation

  Then they see: Goal Setting, My 360, Manager Evaluation Center, Executive Summary, Org Chart, and Review Archive

  And they do not see: Team Overview, My Review, or admin-only tools (Cycle Settings, User Management)

AC6: Badge counters are displayed

  Given the Manager has 3 pending assessments

  When they view the sidebar navigation

  Then the "Team Overview" or "Manager Evaluation Center" badge shows "3"

AC7: Navigation updates on role change

  Given an Employee is promoted to Manager

  When they log in after the role change

  Then the sidebar navigation shows Manager-level items including Team Overview and Manager Evaluation Center

AC8: Exempt user's menu is adjusted

  Given a user is on the assessment exemption list (B7)

  When they view the sidebar navigation

  Then "My Review" shows the exemption message or is hidden

  And all manager-level items remain visible

AC9: Menu items the user cannot access are completely hidden

  Given an Employee views the sidebar

  When the navigation renders

  Then no grayed-out or disabled items are shown — only accessible items are visible
