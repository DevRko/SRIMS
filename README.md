# 📦 SRIMS — Stationery Requisition & Inventory Management System

<div align="center">

![Next.js](https://img.shields.io/badge/Next.js-14-black?style=for-the-badge&logo=next.js)
![TypeScript](https://img.shields.io/badge/TypeScript-5-blue?style=for-the-badge&logo=typescript)
![Prisma](https://img.shields.io/badge/Prisma-5.22-2D3748?style=for-the-badge&logo=prisma)
![MySQL](https://img.shields.io/badge/MySQL-8.0-4479A1?style=for-the-badge&logo=mysql&logoColor=white)
![Tailwind CSS](https://img.shields.io/badge/Tailwind_CSS-3-38B2AC?style=for-the-badge&logo=tailwind-css)
![Docker](https://img.shields.io/badge/Docker-Ready-2496ED?style=for-the-badge&logo=docker)

**A production-grade, full-stack web application for managing office stationery requisitions, multi-role approvals, inventory movements, and item issuance — built by [ESPL](https://eduplex.in).**

[Features](#-features) · [Tech Stack](#-tech-stack) · [Architecture](#-architecture) · [Getting Started](#-getting-started) · [Roles & Permissions](#-roles--permissions) · [Modules](#-modules) · [Database Schema](#-database-schema) · [Deployment](#-deployment)

</div>

---

## ✨ Overview

SRIMS is an end-to-end **Stationery Requisition and Inventory Management System** designed for medium to large organisations. It replaces paper-based or spreadsheet-driven stationery procurement workflows with a structured, role-gated, fully-audited digital system.

From raising a requisition to stock receiving, approval, issuance, and reporting — every step is tracked, every actor is logged, and every data point is visible in real time.

> **Demo-ready out of the box.** The app ships with a rich mock-data layer so it runs fully functional with zero external dependencies — just `npm install && npm run dev`. Switching to a real MySQL database is a single environment variable away.

---

## 🚀 Features

### Core Workflows
- **Requisition lifecycle** — Draft → Pending → Approved → Issued / Partial (or Rejected)
- **3-step wizards** for New Requisition, Stock Inward (GRN), and Issue Items — each with draft persistence so in-progress work survives navigation
- **Atomic Approve-and-Issue** — approvers can approve and dispatch items in a single action from the Pending Approvals page
- **Auto-Approval Rules** — configurable per priority level (Low / Normal / Urgent), attributed to a synthetic "Auto-Approval System" actor in the audit log

### Inventory Management
- Real-time stock ledger with INWARD / OUTWARD / ADJUSTMENT transaction types
- GRN (Goods Received Note) wizard with drag-and-drop file attachments
- Manual outward and adjustment forms with optional requisition linkage
- Low-stock alerts with configurable minimum thresholds and severity tiers (LOW / CRITICAL / OUT_OF_STOCK)

### Reporting & Exports
- Requisition history with date-range filter
- Requisition Analytics — item demand patterns, fulfillment rates, per-user behavior
- Inventory overview with live donut chart
- Every report exports to both **CSV** and **PDF** (fully client-side via jsPDF + autotable — no backend needed)

### System
- Full **Audit Log** — every create, update, delete, and status change is recorded with actor, entity, before/after details, and timestamp
- In-app **Notification** system — low-stock alerts, submission notifications, approval/rejection events, issuance confirmations
- **Profile** page with avatar upload (JPG/PNG/WEBP, 2 MB cap)
- **Forgot/Reset Password** flow — token generation with 1-hour expiry (email send is a stub ready to wire to any provider: Resend, SES, Postmark)

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| **Framework** | [Next.js 14](https://nextjs.org) (App Router) |
| **Language** | TypeScript 5 |
| **UI** | React 18 + [Tailwind CSS](https://tailwindcss.com) + custom design tokens |
| **Component Library** | [Radix UI](https://www.radix-ui.com) (Dialog, Dropdown, Select, Tooltip, Popover, Avatar, Checkbox, ScrollArea) |
| **Icons** | [Lucide React](https://lucide.dev) |
| **Charts** | [Recharts](https://recharts.org) (Area, Bar, Donut) |
| **State Management** | [Zustand](https://zustand-demo.pmnd.rs) with `persist` middleware → `localStorage` |
| **Forms** | [React Hook Form](https://react-hook-form.com) + [Zod](https://zod.dev) validation |
| **Authentication** | [NextAuth.js v4](https://next-auth.js.org) — Credentials provider + JWT sessions |
| **ORM** | [Prisma](https://prisma.io) v5.22.0 |
| **Database** | MySQL 8.0 |
| **Password Hashing** | [bcryptjs](https://github.com/dcodeIO/bcrypt.js) |
| **PDF Export** | [jsPDF](https://github.com/parallax/jsPDF) + [jspdf-autotable](https://github.com/simonbengtsson/jsPDF-AutoTable) |
| **Excel/CSV Export** | [SheetJS (xlsx)](https://sheetjs.com) |
| **Fonts** | Geist VF + Geist Mono VF |
| **Containerisation** | Docker + Docker Compose |

---

## 🏗 Architecture

```
srims/
├── prisma/
│   ├── schema.prisma          # 14-model MySQL schema
│   └── seed.ts                # Seeds DB from mock-data with bcrypt-hashed passwords
│
├── src/
│   ├── app/
│   │   ├── (auth)/            # Login, Forgot Password, Reset Password
│   │   ├── (dashboard)/       # All protected pages (route group)
│   │   │   ├── dashboard/
│   │   │   ├── requisitions/  # new | my | drafts
│   │   │   ├── approvals/     # pending | approved | rejected
│   │   │   ├── inventory/     # overview | inward | outward | adjust | transactions | low-stock
│   │   │   ├── issue/         # items | queue | history
│   │   │   ├── masters/       # categories | items | departments | suppliers | users | roles | auto-approval
│   │   │   ├── reports/       # overview | requisition-analytics
│   │   │   └── system/        # profile | settings | audit-logs | help
│   │   └── api/
│   │       ├── users/         # GET | POST /api/users
│   │       ├── users/[id]/    # GET | PATCH | DELETE /api/users/[id]
│   │       └── auth/          # NextAuth handler + forgot-password + reset-password
│   │
│   ├── components/
│   │   ├── layout/            # Sidebar, Topbar, PageHeader
│   │   ├── shared/            # DataTable, StatCard, StatusPill, WizardStepper, ...
│   │   ├── icons/items/       # ItemIcon (25-preset + custom data-URL renderer)
│   │   └── providers/         # Providers wrapper (SessionProvider, QueryClientProvider)
│   │
│   ├── stores/
│   │   └── app-store.ts       # Zustand store — single source of truth for all client state
│   │
│   └── lib/
│       ├── auth.ts            # Dual-mode NextAuth config (mock ↔ Prisma)
│       ├── navigation.ts      # RBAC-aware sidebar nav config
│       ├── prisma.ts          # Lazy Prisma client with isDatabaseConfigured flag
│       ├── server-auth-state.ts # Server-side deactivation registry (mock mode)
│       ├── utils.ts           # formatCurrency, formatDate, cn()
│       └── data/mock-data.ts  # Full demo dataset (users, items, categories, requisitions, ...)
│
├── Dockerfile
├── docker-compose.yml
└── package.json
```

### Dual-Mode Authentication

`src/lib/auth.ts` transparently switches between two credential-check paths at runtime:

- **Mock mode** (no `DATABASE_URL`) — credentials are checked against the bundled `mock-data.ts` list. Zero setup — works immediately after `npm install && npm run dev`.
- **Database mode** (`DATABASE_URL` set) — credentials are checked against MySQL via Prisma with bcrypt-hashed passwords. This is also what makes users created through Masters → Users actually able to log in.

The Prisma client is **lazily instantiated** (`getPrismaClient()`) so the app never crashes when no database is configured.

### Zustand Store with Selective Persistence

The global `useAppStore` persists all durable data to `localStorage` (users, departments, categories, suppliers, items, requisitions, GRNs, issuances, transactions, audit logs, notifications, auto-approval settings, wizard drafts). It deliberately **excludes** `currentUser` (re-synced from the NextAuth JWT on load) and the shopping cart (cleared on submit).

Auto-approval is handled by a **pure `computeAutoApproval()` function** — no side effects, called identically from all three places a requisition can become PENDING (fresh submit, draft re-submit, Drafts page submit), ensuring consistent behaviour regardless of entry point.

---

## 🏁 Getting Started

### Prerequisites

- Node.js ≥ 18
- npm ≥ 9

### Zero-setup (mock-data mode)

```bash
git clone https://github.com/your-org/srims.git
cd srims
npm install
npm run dev
```

Open [http://localhost:3000](http://localhost:3000). No database, no environment variables needed.

A `.env.local` with a working `NEXTAUTH_SECRET` is already included for local development.

### Demo Credentials

| Role | Email | Password | Access Level |
|---|---|---|---|
| **Admin** | rahul@srims.com | Admin@123 | Full system access |
| **User** | priya@srims.com | User@123 | Requisitions only |
| **Approver** | amit@srims.com | Approver@123 | Approvals + Reports |
| **Inventory Manager** | sandeep@srims.com | Inventory@123 | Inventory + Issue |

> **💡 Role Switcher:** A floating 🔄 button (bottom-right of every dashboard page) lets you switch between all four roles instantly without logging out — perfect for demoing the app.

### Available Scripts

```bash
npm run dev          # Start development server
npm run build        # Production build
npm run start        # Start production server
npm run lint         # ESLint check

npm run db:migrate   # Run Prisma migrations (requires DATABASE_URL)
npm run db:seed      # Seed the database (requires DATABASE_URL)
npm run db:studio    # Open Prisma Studio (requires DATABASE_URL)
```

---

## 👥 Roles & Permissions

| Feature | Admin | Approver | Inventory Mgr | User |
|---|:---:|:---:|:---:|:---:|
| Dashboard | ✅ | ✅ | ✅ | ❌ |
| New / My Requisitions | ✅ | ✅ | ✅ | ✅ |
| Pending Approvals | ✅ | ✅ | ❌ | ❌ |
| Approved / Rejected list | ✅ | ✅ | ❌ | ❌ |
| Stock Overview | ✅ | ❌ | ✅ | ❌ |
| Stock Inward (GRN) | ✅ | ❌ | ✅ | ❌ |
| Stock Outward / Adjust | ✅ | ❌ | ✅ | ❌ |
| Issue Items / Queue / History | ✅ | ❌ | ✅ | ❌ |
| Reports Overview | ✅ | ✅ | ✅ | ❌ |
| Requisition Analytics | ✅ | ❌ | ❌ | ❌ |
| Masters — Categories / Items / Suppliers | ✅ | ❌ | ✅ | ❌ |
| Masters — Departments / Users / Roles | ✅ | ❌ | ❌ | ❌ |
| Auto-Approval Rules | ✅ | ❌ | ✅ | ❌ |
| Audit Logs / System Settings | ✅ | ❌ | ❌ | ❌ |
| Profile | ✅ | ✅ | ✅ | ✅ |

---

## 📚 Modules

### 🏠 Dashboard

A live command-centre for Admins, Approvers, and Inventory Managers.

- **4 KPI Stat Cards** — Total Requisitions, Pending Approvals, Issued/Partial, Low Stock Items
- **Requisition Trend Area Chart** — dynamic granularity (day / week / month) based on actual data span; dual series (Submitted vs. Issued)
- **Low Stock Alerts Panel** — top 4 items below minimum threshold with stock severity badges
- **Recent Requisitions** — latest 5 with status pills and direct links
- **Recent Stock Transactions** — latest 5 with transaction-type colour coding
- **Date-range filter popover** — filters Recent Requisitions and Transactions by custom period

### 📋 Requisition Module

**New Requisition** (`/requisitions/new`) — A 3-step wizard:
1. **Category Selection** — visual category tile grid with search
2. **Item Selection** — filterable item list with live cart (quantity stepper, real-time total, stock availability guard)
3. **Details** — purpose, required date, remarks, priority selection; Save as Draft or Submit

Supports **edit mode** via `?edit=<id>` — pre-populates cart and all fields from an existing draft and updates it in place on save rather than creating a duplicate.

**My Requisitions** (`/requisitions/my`) — Full history with status filter pills (All / Draft / Pending / Approved / Rejected / Issued / Partial), search, pagination, and a detail modal showing item-level breakdown, approval history, and totals.

**Drafts** (`/requisitions/drafts`) — Edit, Submit, and Delete actions with confirmation dialogs.

### ✅ Approval Module

**Pending Approvals** (`/approvals/pending`):
- Stat strip showing pending count, approved today, rejected today
- Expandable detail panel with full item-level breakdown
- **Department and Priority filters**
- **Approve** — with optional approved-quantity adjustment per line item
- **Reject** — mandatory rejection reason modal
- **Approve & Issue** — atomic single-action: approves the requisition, deducts stock, creates an issuance record, writes the audit log, and fires notifications — all in one call

**Approved** / **Rejected** — read-only lists with detail modal (eye button) and expandable rejection reason.

### 📦 Inventory Module

**Stock Overview** (`/inventory/overview`) — Live donut chart of stock health distribution, filterable items table (by category, search, and stock status), kebab menu on each row deep-linking to Edit Item, Adjust Stock, or Stock Outward.

**Stock Inward — GRN Wizard** (`/inventory/inward`) — 3 steps:
1. Supplier + header details (invoice no., dates, delivery challan)
2. Item line entry (category-aware search, quantity and unit price per line)
3. Review + **drag-and-drop file attachments** (type/size validated client-side)

Draft persistence: partially-filled GRN wizards are saved as draft GRNs; a resume/discard banner greets users when they return. Submitting increments live stock and creates INWARD StockTransaction records.

**Stock Outward** (`/inventory/outward`) — Manual outward movement form with optional requisition linkage for traceability; full outward history table.

**Adjust Stock** (`/inventory/adjust`) — Signed delta correction (positive = increase, negative = decrease) with live resulting-stock preview and full adjustment history.

**Low Stock Alerts** (`/inventory/low-stock`) — Tabular view with shortfall calculation, severity badges (LOW / CRITICAL / OUT_OF_STOCK), and reorder links.

**All Transactions** (`/inventory/transactions`) — Unified INWARD + OUTWARD + ADJUSTMENT ledger.

### 🚚 Issue Module

**Issue Items Wizard** (`/issue/items`) — 3 steps:
1. Select an approved requisition from the queue
2. Set issued quantity per line (cannot exceed approved quantity; warns on partial)
3. Receiver details, reference note, confirm

Per-requisition draft persistence — a "Draft saved" badge appears on the issue list. Confirming deducts stock, creates an `Issuance` record and OUTWARD `StockTransaction` entries, updates requisition status to ISSUED or PARTIAL, and fires a notification to the requester.

**Issue Queue** (`/issue/queue`) — All approved requisitions awaiting issuance.

**Issued History** (`/issue/history`) — Full issuance ledger with reference numbers and detail modal.

### 🗂 Masters Module

Full CRUD for all reference data:

| Master | Key Features |
|---|---|
| **Categories** | Hierarchical (parent/child one level), 10-preset icon picker + custom image upload (JPG/PNG/WEBP/SVG, 1 MB), 8-colour picker, live item counts, promote-children-on-delete |
| **Items** | CRUD, active/inactive toggle, 25-preset icon picker + custom image upload, category/unit/price/minimum stock threshold |
| **Departments** | Admin-only; blocked from deletion if users are assigned |
| **Suppliers** | Contact + address fields, GRN count per supplier |
| **Auto-Approval Rules** | Enable/disable toggle, multi-select priority levels; changes take effect immediately; attributed to a synthetic "Auto-Approval System" actor in audit logs |
| **Users** | Admin-only; role + department assignment, active toggle, self-delete blocked |
| **Roles & Permissions** | Read-only permission matrix across all 4 roles |

### 📊 Reports Module

**Overview** (`/reports`) — Three report types:
- **Inventory Report** — full item list with stock levels and values; CSV + PDF export
- **Requisition History** — date-range filterable; CSV + PDF export
- **Audit Trail** — full activity log export; CSV + PDF export

All exports are **fully client-side** — no backend roundtrip, works in both mock and database modes.

**Requisition Analytics** (`/reports/requisition-analytics`) — Admin-only deep-dive:
- Highest and lowest demand items by total quantity requested
- Top and bottom requesters by requisition count
- **Grouped bar chart** — requested vs. issued quantity for the top 8 items
- **Horizontal bar chart** — requisitions by user
- Full searchable/sortable item demand table with fulfilment rate
- Full searchable/sortable user demand table with total value

### ⚙️ System Module

| Page | Access | Description |
|---|---|---|
| **Profile** | All roles | Name, email, department display; avatar upload (JPG/PNG/WEBP, 2 MB, stored as data URL) |
| **Settings** | Admin | Application settings |
| **Audit Logs** | Admin | Full chronological log of every system action with actor, entity, entity ID, and details |
| **Help & Support** | All roles | Contact cards (contacts@eduplex.in, csr@eduplex.in, +91 83370 56594), FAQ list |

---

## 🗄 Database Schema

14 Prisma models backed by MySQL 8:

```
Department        User              PasswordResetToken
Category          Item              Supplier
Requisition       RequisitionItem
Issuance
StockTransaction  GRN               GRNItem
AuditLog          Notification
```

### Key design decisions

- **Self-referential `User.approver`** — each user can have an assigned approver, enabling department-level approval hierarchies
- **Self-referential `Category.parent`** — one level of sub-categories; the app promotes children to top-level on parent deletion rather than orphaning them
- **Structured ID formats** — `REQ-YYYY-XXXXX`, `ISS-YYYY-XXXXX`, `GRN-YYYY-XXXX`, `ITM-XXXX` for human-readable references in every document
- **`AuditLog.before` / `.after`** — JSON Text columns for full diff storage on every entity mutation
- **`User.avatarUrl` as `LongText`** — stores Base64 data URLs today; the field is designed to accept an object-storage URL (S3, Cloudinary) without a schema migration
- **`GRN.attachments` as JSON Text** — stores an array of file references; switchable to object storage paths without schema change
- **Indexed hot columns** — `status`, `createdAt`, `date`, `userId`, `departmentId`, `itemId`, `token` for query performance at scale

### Enums

```prisma
enum Role               { ADMIN | USER | APPROVER | INVENTORY_MGR }
enum RequisitionStatus  { DRAFT | PENDING | APPROVED | REJECTED | ISSUED | PARTIAL }
enum StockTransactionType { INWARD | OUTWARD | ADJUSTMENT }
enum Priority           { LOW | NORMAL | URGENT }
```

---

## 🚢 Deployment

### Option A — Local MySQL (no Docker)

```bash
# 1. Set your database URL
echo 'DATABASE_URL="mysql://user:password@localhost:3306/srims"' >> .env.local

# 2. Generate client and create tables
npx prisma generate
npx prisma db push

# 3. Seed demo data (passwords are bcrypt-hashed by the seed script)
npx prisma db seed

# 4. Start
npm run build && npm run start
```

### Option B — Docker Compose (one command)

```bash
docker compose up --build
```

Starts MySQL 8, builds the Next.js app image, runs `prisma db push` + `prisma db seed` on container start, and serves the app on `localhost:3000`.

Override defaults via a `.env` file:

```env
MYSQL_ROOT_PASSWORD=your_root_password
MYSQL_USER=srims_user
MYSQL_PASSWORD=your_db_password
MYSQL_DATABASE=srims
NEXTAUTH_SECRET=<generate with: openssl rand -base64 32>
NEXTAUTH_URL=https://srims.yourcompany.com
```

### Environment Variables

| Variable | Required for DB mode | Notes |
|---|---|---|
| `DATABASE_URL` | Yes | `mysql://user:pass@host:3306/dbname` |
| `NEXTAUTH_SECRET` | Yes | Generate: `openssl rand -base64 32` |
| `NEXTAUTH_URL` | In production | Must be the deployed URL, not localhost |

### Pre-launch Security Checklist

- [ ] Rotate `NEXTAUTH_SECRET` — never reuse the one shipped in this repo
- [ ] Change or deactivate the 4 demo accounts once real users are onboarded
- [ ] Wire a real email provider for password resets (swap the `console.log` in `/api/auth/forgot-password/route.ts` for Resend / SES / Postmark)
- [ ] Move GRN attachments and profile avatars to object storage (S3, Cloudinary, etc.) — both currently use in-memory data URLs with no durable backend
- [ ] Tighten the MySQL user's permissions after initial migration (DDL rights are needed for `db push` and seeding but not for runtime queries)
- [ ] Add server-side role checks to any new API routes (following the `getServerSession` pattern in `/api/users`)

---

## 🔌 API Routes

| Method | Endpoint | Description | Auth |
|---|---|---|---|
| `GET` | `/api/users` | List all users | Session required |
| `POST` | `/api/users` | Create a new user | Admin only |
| `GET` | `/api/users/[id]` | Get user by ID | Session required |
| `PATCH` | `/api/users/[id]` | Update user | Admin only |
| `DELETE` | `/api/users/[id]` | Delete user | Admin only |
| `POST` | `/api/auth/forgot-password` | Generate password-reset token | Public |
| `POST` | `/api/auth/reset-password` | Validate token + set new password | Public |
| `*` | `/api/auth/[...nextauth]` | NextAuth handler | — |

All Prisma-backed routes return a clean `503` (not a crash) when no database is configured, so the app remains fully usable in mock mode regardless of which API calls the store fires as best-effort syncs.

---

## 🧩 Shared Components

| Component | Description |
|---|---|
| `DataTable` | Sortable, filterable, paginated table |
| `StatusPill` | Multi-variant coloured status badge (14 variants) |
| `StatCard` | KPI card with icon, tint colour, value, and optional delta |
| `WizardStepper` | Multi-step progress indicator for 3-step wizards |
| `RequisitionDetailModal` | Full item-level detail modal shared across 6+ pages |
| `QuantityStepper` | Quantity input with +/− controls and bounds enforcement |
| `CategoryTile` | Visual card for category selection in the new requisition wizard |
| `ToggleSwitch` | Accessible boolean toggle (used in Auto-Approval Rules) |
| `Sidebar` | RBAC-gated nav with push-layout on desktop + slide-over drawer on mobile (<1024px) |
| `Topbar` | Notification dropdown (mark as read / mark all), avatar menu, role switcher |
| `ItemIcon` | Resolves an `iconKey` to one of 25 SVG presets — or renders a `<img>` if the key is a `data:image` URL (custom upload path) |

---

## 🗺 Roadmap / Remaining Work

- [ ] **Email provider integration** — wire Resend / SES / Postmark into the forgot-password route
- [ ] **Object storage** — replace data-URL avatars and GRN attachments with S3 / Cloudinary
- [ ] **Server-side role enforcement** — add `getServerSession` guards to all future API routes (the `/api/users` routes already demonstrate the pattern)
- [ ] **Real-time notifications** — upgrade the current polling-based notification model to WebSockets or SSE
- [ ] **Print-friendly issuance slips** — PDF generation for issuance reference documents
- [ ] **Bulk item import** — CSV upload for seeding the Items master in production
- [ ] **Department-aware approval routing** — auto-assign approver based on the requester's department hierarchy

---

## 🤝 Built by ESPL

This system was designed and developed internally at **ESPL (Eduplex Solutions Pvt. Ltd.)** to digitise stationery and asset requisition workflows across departments.

> 📧 contacts@eduplex.in | csr@eduplex.in
> 📞 +91 83370 56594

---

<div align="center">

Made with ❤️ at **ESPL** · Licensed for internal use

</div>
