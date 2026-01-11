# Archive Indexer Implementation Pack (MVP)

## 1) Product & Platform Summary

- **Product:** Archive Indexer (Internal Document Archive + Index + Search)
- **Platform:** SharePoint Online (Microsoft 365)
- **Delivery:** SPFx solution (.sppkg) deployed via Tenant App Catalog; eligible for AppSource/SharePoint Store.
- **Identity/Access:** Microsoft 365 users + groups (Entra ID backed).

---

## 2) Architecture (MVP)

### 2.1 SPFx Components

- **Archive Search Web Part**
- **Document Intake (Upload + Index) Web Part**
- **Admin Setup Wizard Web Part**
- *(Optional later)* Application Customizer (global header/search)

### 2.2 Storage (SharePoint-native)

- **Document Libraries** for files
- **SharePoint Lists** for configuration data
- **Search** via SharePoint Search API

> MVP remains SharePoint-native for security, performance, and marketplace readiness.

---

## 3) Information Architecture (MVP Choice)

**Option A (recommended):** One site per company with a single library + branch metadata.

- **Site:** “Archive Indexer – <CompanyName>” (Modern Team Site)
- **Library:** `Archive Documents`
- **Restricted:** Separate library `Archive Restricted` (optional but recommended)

---

## 4) SharePoint IA Build Sheet (MVP)

### 4.1 Libraries

#### Archive Documents

- Versioning: **On (major versions)**
- Require checkout: **Off**
- Content approval: **Off (MVP)**
- Default view: **Archive Search View**

#### Archive Restricted (recommended)

- Same schema as Archive Documents
- Permissions limited to Admin/Librarian/explicit groups

### 4.2 Site Columns (prefix **AI_**)

> Create once at the site level and reuse in content type and libraries.

| Internal Name | Display Name | Type | Required | Notes |
|---|---|---|---|---|
| AI_DocNumber | Document Number | Single line | Yes | Unique business ID (validate via wizard/upload form). |
| AI_DocType | Document Type | Lookup | Yes | Lookup to **AI Document Types** list. |
| AI_Branch | Branch | Lookup | Yes | Lookup to **AI Branches** list. |
| AI_Department | Department | Choice | Yes | Finance / HR / IT / Projects / Legal / Procurement (default). |
| AI_Confidentiality | Confidentiality | Choice | Yes | Internal / Confidential / Restricted. |
| AI_IssueDate | Issue Date | Date | No |  |
| AI_ReceivedDate | Received Date | Date | No |  |
| AI_ProjectCode | Project Code | Single line | No |  |
| AI_VendorClient | Vendor / Client | Single line | No |  |
| AI_Status | Status | Choice | Yes | Draft / Submitted / Approved / Archived. |
| AI_Tags | Tags | Multiple lines | No | MVP: free tags; can switch to managed metadata later. |
| AI_DocOwner | Document Owner | Person | No | Defaults to uploader. |
| AI_RetentionCategory | Retention Category | Choice | No | Optional; map to Purview later. |
| AI_Notes | Notes | Multiple lines | No | Plain text only. |

### 4.3 Content Types

**AI Archived Document**

- Parent: `Document`
- Columns: All **AI_** site columns
- Required: Document Number, Document Type, Branch, Confidentiality, Status

Attach to **Archive Documents** and **Archive Restricted** libraries.

### 4.4 Lists (Configuration)

A) **AI Branches**

| Field | Type | Required | Example |
|---|---|---|---|
| Title (Branch Name) | Single line | Yes | Amman |
| BranchCode | Single line | Yes | AMM |
| Country | Single line | No | Jordan |
| Active | Yes/No | Yes | True |
| DefaultLibraryPath | Single line | No | /sites/Archive/Archive Documents |

B) **AI Document Types**

| Field | Type | Required | Example |
|---|---|---|---|
| Title (Doc Type) | Single line | Yes | Invoice |
| TypeCode | Single line | No | INV |
| RequiredFieldsJson | Multiple lines | No | {"required":["AI_ProjectCode"]} |
| DefaultConfidentiality | Choice | No | Internal |
| DefaultRetentionCategory | Choice | No | Finance-7y |

C) **AI Security Roles**

| Field | Type | Required | Example |
|---|---|---|---|
| Title (Role Name) | Single line | Yes | Archive Librarian |
| SharePointGroupName | Single line | Yes | AI_Archive_Librarian |
| PermissionLevel | Choice | Yes | Read / Contribute / Edit / Full Control |
| Scope | Choice | Yes | Site / Library / RestrictedLibrary |

D) **AI Settings**

| Field | Type | Required | Example |
|---|---|---|---|
| Title (Setting Key) | Single line | Yes | EnableApproval |
| Value | Single line | Yes | false |
| Notes | Multiple lines | No |  |

E) **AI Audit (Optional)**

| Field | Type | Example |
|---|---|---|
| Action | Choice | Search / Upload / UpdateMetadata / Setup |
| ItemUrl | Hyperlink |  |
| Actor | Person |  |
| Timestamp | DateTime |  |
| Details | Multiple lines |  |

### 4.5 Views

**Archive Search View (default)**

- Columns: File Name, Document Number, Branch, Document Type, Department, Confidentiality, Issue Date, Status, Modified, Modified By
- Sort: Modified desc

**By Branch**

- Group by Branch

**Restricted Only (admin)**

- Filter: Confidentiality = Restricted

---

## 5) Wizard Flow Screens (Admin Setup Wizard Web Part)

### Screen 0 — Welcome

- Title: “Archive Indexer Setup Wizard”
- Detect site URL, tenant context, user
- Admin privilege check

### Screen 1 — Choose Architecture

- **Single Library + Branch metadata (recommended)**
- Library per Branch (advanced)

### Screen 2 — Create Components

Checklist:

- Lists (AI Branches, AI Document Types, AI Settings, AI Security Roles)
- Libraries (Archive Documents, optional Archive Restricted)
- Site Columns (AI_*)
- Content Type (AI Archived Document)
- Views

### Screen 3 — Branch Setup

- Add/edit branches in a table
- Optional CSV import
- Defaults include “Head Office” row

### Screen 4 — Document Types Setup

- Preload defaults (Invoice, Contract, Delivery Note, Report, Letter, Drawing)
- Configure default confidentiality + required fields

### Screen 5 — Security & Permissions

- Create new SharePoint groups (recommended) or map existing groups
- Toggle **Restricted Library**
- Apply permissions to libraries/lists

### Screen 6 — Validation & Health Check

- Lists exist
- Libraries exist
- Content type attached
- Required columns present
- Permissions applied
- Sample search call works

### Screen 7 — Finish

- Summary report
- Links to Search / Upload / Settings
- Exportable setup report (JSON/text)

---

## 6) Feature Scope (MVP)

### 6.1 Search Web Part

- Full-text search (SharePoint Search)
- Filters: Branch, Type, Date Range, Department, Confidentiality, Status, Owner
- Results grid: File Name, Document Number, Branch, Type, Issue Date, Status
- Actions: View / Download / Edit metadata / Open in SharePoint

### 6.2 Upload + Index Web Part

- Upload to Archive Documents
- Metadata form with validation
- **Document Number required + unique**
- Required classification (Branch + Type + Confidentiality)
- Optional “Submit for approval” toggle (Phase 1.5)

---

## 7) Permissions Model

### Roles (default)

- **Archive Admin:** Full control (setup + security + settings)
- **Archive Librarian:** Edit + manage metadata + approvals
- **Contributor:** Upload + edit own docs metadata
- **Reader:** Read only

### Restricted Documents

- Store in **Archive Restricted** library with tighter permissions
- Avoid item-level permission sprawl

---

## 8) Security Checklist (MVP)

### SPFx Frontend

- No secrets in client code
- Use SPFx SharePoint/Graph clients
- Validate and sanitize inputs
- Never render user input as HTML
- Least-privilege permissions

### SharePoint / Tenant

- Versioning enabled
- Restricted library permissions locked
- Group-based roles

---

## 9) Marketplace Packaging (MVP)

- `gulp bundle --ship`
- `gulp package-solution --ship`
- Validate metadata in `config/package-solution.json`
- Decide on `skipFeatureDeployment` for tenant-wide availability

### Submission Checklist

- App description + screenshots
- Privacy + security statements
- Support contact
- Permission documentation

---

## 10) Acceptance Criteria (Definition of Done)

- Admin installs .sppkg and runs wizard successfully
- Lists/libraries/columns/content type created automatically
- Role-based permissions applied correctly
- Upload + metadata validation enforced
- Search results + filters work as expected
- Restricted docs are isolated from unauthorized users
- No secrets in client-side code

---

## 11) Roadmap (Post-MVP)

- **Phase 1.5:** Approval workflow, bulk upload, bulk metadata edit
- **Phase 2:** Optional Azure backend (OCR, cross-system indexing, reporting)
- **Phase 3:** Multi-language UI (Arabic/English), records management integration
