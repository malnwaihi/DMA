# Archive Indexer Implementation Pack (MVP)

## 1) Information Architecture (Confirmed)

**Option A: Single site collection**

- One SharePoint site collection (e.g., “Jordan Archive”).
- **Separate library per branch** (confirmed).
- **Restricted documents**: separate restricted library per branch (recommended for permission isolation).
- **Managed metadata**: use **Term Store** for Document Type and Tags.

**Library naming convention**

- `Archive - <Branch Name>`
- `Archive - <Branch Name> (Restricted)`

---

## 2) SharePoint IA Build Sheet (MVP)

### 2.1 Site Columns (Site Collection level)

> Create as **Site Columns** in a dedicated group, e.g., `Archive Indexer Columns`.

| Column Name | Type | Required | Notes |
|---|---|---|---|
| Document Number | Single line of text | Yes | Unique business ID (validate per branch library). |
| Document Type | Managed Metadata | Yes | Term set `Document Types`. |
| Branch | Single line of text | Yes | Pre-filled per branch library (hidden in UI). |
| Department | Choice | No | Controlled list. |
| Vendor / Client | Single line of text | No |  |
| Project Code | Single line of text | No |  |
| Issue Date | Date and Time (Date only) | No |  |
| Received Date | Date and Time (Date only) | No |  |
| Confidentiality | Choice | Yes | Public / Internal / Confidential / Restricted. |
| Status | Choice | Yes | Draft / Submitted / Approved / Archived. |
| Tags | Managed Metadata | No | Term set `Archive Tags`. |
| Owner | Person or Group | No |  |
| Retention Category | Choice | No | Tenant-specific values. |
| Notes | Multiple lines of text | No | Plain text (no rich HTML). |

### 2.2 Content Types

Create content type: **Archive Document**

- Parent: `Document`
- Add all site columns listed above
- Set **Document Number**, **Document Type**, **Confidentiality**, **Status** to Required

### 2.3 Libraries (per branch)

For each branch:

- **Library A:** `Archive - <Branch Name>`
  - Content type: `Archive Document` (default)
  - Versioning: On (major versions)
  - Require checkout: Off
  - Metadata required: On
- **Library B:** `Archive - <Branch Name> (Restricted)`
  - Same schema, but permissions locked down

### 2.4 Lists (Configuration)

Create each as a SharePoint List:

1) **Branches**
   - Branch Name (Title)
   - Code (single line)
   - Country (single line)
   - Active (Yes/No)
   - Default Library Path (single line)

2) **Security Roles**
   - Role Name (Title)
   - SharePoint Group (person/group)

3) **App Settings**
   - Key (Title)
   - Value (single line)
   - Notes (multiple lines)

> **Document Types** and **Tags** managed in the **Term Store** (not lists).

### 2.5 Term Store

Create in Term Store:

- Term Set: `Document Types`
- Term Set: `Archive Tags`

---

## 3) Wizard Flow Screens (Admin Setup Wizard Web Part)

### Step 0: Welcome + Environment Detection

- Title: “Archive Indexer Setup Wizard”
- Shows detected site URL, tenant, user.
- Checks admin privileges.

### Step 1: Branch Libraries

- Show list of branches (from Branches list or manual entry).
- For each branch:
  - Create `Archive - <Branch Name>`
  - Create `Archive - <Branch Name> (Restricted)`
- Output: success per branch.

### Step 2: Columns + Content Type

- Create Site Columns (if missing).
- Create Content Type `Archive Document`.
- Add to each branch library and set as default.

### Step 3: Term Store Binding

- Confirm `Document Types` and `Archive Tags` term sets exist.
- Link columns to term sets.

### Step 4: Permissions Mapping

- Map roles to SharePoint groups:
  - Archive Admin
  - Archive Librarian
  - Contributor
  - Reader
- Apply permissions:
  - Standard libraries: Reader/Contributor/Librarian/Admin
  - Restricted libraries: Librarian/Admin only

### Step 5: Optional Folder Seeding

- Toggle to create base folders (e.g., Department/Year).

### Step 6: Health Check + Summary

- Validate:
  - Lists created
  - Libraries created
  - Columns applied
  - Content type applied
  - Permissions set
- Show exportable report.

---

## 4) Document Intake Web Part (UI Spec + Optional Approval Toggle)

### Upload + Index Form

**Required fields**

- Document Number (unique per branch)
- Document Type (managed metadata)
- Confidentiality
- Status

**Optional fields**

- Department
- Vendor/Client
- Project Code
- Issue Date
- Received Date
- Tags (managed metadata)
- Owner
- Retention Category
- Notes

### Optional Toggle: “Submit for approval”

- Toggle shown when **App Settings** key `approval_enabled = true`.
- If enabled, set Status = `Submitted` and assign approval task (Phase 1.5 stub).
- If disabled, Status defaults to `Draft`.

### Validation

- Document Number required and **unique** within selected branch library.
- Document Type + Confidentiality required.
- Status required.

---

## 5) Search Web Part (UI + Filters)

### Search Sources

- SharePoint Search API
- Scope: branch libraries user has access to
- Restricted libraries only returned if user has permission

### Filters

- Branch (auto from library)
- Document Type
- Department
- Date range (Issue Date / Received Date)
- Confidentiality
- Status
- Owner

### Results Grid Columns

- File Name
- Document Number
- Branch
- Document Type
- Issue Date
- Status
- Actions: View / Download / Edit Metadata / Open in SharePoint

---

## 6) Security Checklist (SPFx + Tenant)

### SPFx Frontend

- Do not render raw HTML from user input.
- Sanitize and validate metadata fields.
- Use PnPjs/Graph with least privilege.
- Avoid storing secrets in client-side code.

### SharePoint Tenant

- Enable versioning on all branch libraries.
- Use separate restricted libraries with tighter permissions.
- Use groups and role mapping for permissions.

---

## 7) Marketplace Publishing Checklist (MVP)

### Packaging

- Build SPFx `.sppkg` package.
- Use tenant app catalog deployment.
- Enable tenant-wide availability via `skipFeatureDeployment` if desired.

### Store Readiness

- Provide setup guide (wizard instructions).
- Security and privacy statement (data remains in tenant).
- Document required permissions.

### AppSource/Store Requirements

- Add solution description, screenshots, and support URL.
- Provide versioned release notes.
- Include configuration and uninstall instructions.

---

## 8) Next Implementation Artifacts (Optional)

- Branch list seeding template (CSV).
- Term Store import CSV.
- Setup Wizard wireframes.
- Role mapping defaults (JSON template).
