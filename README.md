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

... (README content continues)
