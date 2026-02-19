<p align="right">
  <a href="./README.pt-br.md">🇧🇷 Versão em Português</a>
</p>

<h1 align="center">IntraNet</h1>
<p align="center"><em>A professional-grade corporate intranet platform for high-scale logistics operations.</em></p>

<p align="center">
  <img src="https://img.shields.io/badge/Status-Production%20Ready-brightgreen?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Version-1.0.7-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/License-Proprietary-red?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Showcase-Portfolio-orange?style=for-the-badge" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/React-18-61DAFB?style=flat-square&logo=react&logoColor=white" />
  <img src="https://img.shields.io/badge/TypeScript-5.x-3178C6?style=flat-square&logo=typescript&logoColor=white" />
  <img src="https://img.shields.io/badge/Supabase-BaaS-3ECF8E?style=flat-square&logo=supabase&logoColor=white" />
  <img src="https://img.shields.io/badge/Vite-5.x-646CFF?style=flat-square&logo=vite&logoColor=white" />
  <img src="https://img.shields.io/badge/TailwindCSS-3.x-38BDF8?style=flat-square&logo=tailwindcss&logoColor=white" />
  <img src="https://img.shields.io/badge/PWA-Ready-5A0FC8?style=flat-square&logo=pwa&logoColor=white" />
  <img src="https://img.shields.io/badge/Mobile-Responsive-green?style=flat-square&logo=android&logoColor=white" />
</p>

---

## 📌 Overview

**IntraNet** is a professional-grade corporate platform designed for high-scale operations management, centralizing integrated workflows for:

- 🎫 **Support Ticketing System** — Complete ticket management with visual Kanban workflows and priority-based SLA tracking.
- 💻 **IT Asset Monitoring** — Real-time hardware and device monitoring with telemetry via WebSocket agents.
- 👥 **User & Department Management** — Centralized management of employees, roles, and sector hierarchies.
- 📦 **Inventory Control** — Real-time tracking of materials, equipment, and resources.
- 📊 **Executive Analytics** — Performance dashboards and operational KPI metrics (MTTR, SLA, Volume Trends).
- 🔔 **Integrated Notifications** — Native push notifications and external webhooks for cross-system alerts.
- 💬 **Internal Chat** — Real-time messaging integrated directly into ticket workflows.

### 🎯 Key Objective

To serve as a **unified hub** for communication and operations management, consolidating multiple workflows into an intuitive and responsive interface — eliminating the need for multiple disconnected tools.

---

## 📱 Mobile & PWA Support

The platform was built with a **mobile-first mindset** and is fully installable as a Progressive Web App:

| Feature | Details |
|---|---|
| **PWA Installable** | Works as a native app on Android & iOS via browser prompt |
| **Offline Support** | Service Worker with network-first + cache fallback strategy |
| **Touch Gestures** | Swipe from left edge to open/close sidebar |
| **iOS Safe Area** | Supports notch & Dynamic Island via `env(safe-area-inset-*)` |
| **Responsive Layout** | Adaptive sidebar (drawer on mobile, fixed on desktop) |
| **Swipe Kanban** | Column-to-column navigation via touch swipe with scroll-snap |
| **Mobile Toasts** | Swipe-to-dismiss notifications with Sonner |
| **Adaptive Cards** | Ticket list switches to card layout on mobile vs table on desktop |

> The system runs seamlessly across **Android, iOS (iPhone/iPad), and all screen sizes from 320px to 1920px+**.

---

## 🖼️ Visual Preview

The platform features a modern, clean interface with full **dark/light mode** and **mobile responsiveness**.

### 🖥️ Desktop

<p align="center">
  <img src="screenshots/preview-1.jpg" width="400" alt="Dashboard Overview" />
  <img src="screenshots/preview-2.jpg" width="400" alt="Attendance History & SLA" />
</p>
<p align="center">
  <img src="screenshots/preview-3.jpg" width="400" alt="Reports & Analytics" />
  <img src="screenshots/preview-4.jpg" width="400" alt="Asset Monitoring" />
</p>
<p align="center">
  <img src="screenshots/preview-5.jpg" width="400" alt="New Ticket Form" />
  <img src="screenshots/preview-6.jpg" width="400" alt="User Management" />
</p>
<p align="center">
  <img src="screenshots/preview-7.jpg" width="400" alt="Changelog Modal" />
  <img src="screenshots/preview-8.jpg" width="400" alt="User Profile" />
</p>
<p align="center">
  <img src="screenshots/preview-9.jpg" width="400" alt="IT Room Inventory" />
</p>

### 📱 Mobile (iPhone — PWA)

> All views captured running as an installed PWA on iPhone, demonstrating dark mode, responsive layouts, and touch-optimized navigation.

<p align="center">
  <img src="screenshots/mobile-1.jpg" width="180" alt="Login Screen — Dark Mode" />
  <img src="screenshots/mobile-2.jpg" width="180" alt="Sidebar Drawer — Swipe Navigation" />
  <img src="screenshots/mobile-3.jpg" width="180" alt="Dashboard — KPI Cards" />
</p>
<p align="center">
  <img src="screenshots/mobile-4.jpg" width="180" alt="Dashboard — Weekly Trend & SLA Donut" />
  <img src="screenshots/mobile-5.jpg" width="180" alt="New Ticket Form — Mobile" />
  <img src="screenshots/mobile-6.jpg" width="180" alt="Asset Monitoring — Mobile" />
</p>

---

## 🏗️ System Architecture

The platform follows a modern **serverless-first** approach combined with real-time capabilities:

```
┌─────────────────────────────────────────────────────┐
│                  React 18 + TypeScript               │
│         (Vite + Tailwind + shadcn/ui + PWA)          │
└────────────────────────┬────────────────────────────┘
                         │
           ┌─────────────┼─────────────┐
           │             │             │
    ┌──────▼──────┐ ┌────▼────┐ ┌─────▼──────┐
    │  REST API   │ │Realtime │ │  WebSocket  │
    │  (PostgREST)│ │(LISTEN/ │ │  (Telemetry)│
    └──────┬──────┘ │NOTIFY)  │ └─────┬──────┘
           │        └────┬────┘       │
    ┌──────▼──────────────▼───────────▼──────┐
    │         Supabase (PostgreSQL + Auth)    │
    │              + Edge Functions           │
    └────────────────────────────────────────┘
```

| Layer | Technology |
|---|---|
| Frontend | React 18, TypeScript, Vite |
| Styling | Tailwind CSS, shadcn/ui, Material-UI |
| State | React Query (TanStack) |
| Backend (BaaS) | Supabase (PostgreSQL, Auth, Storage, Realtime) |
| Compute | Edge Functions (Deno Runtime) |
| Telemetry | Custom Node.js WebSocket Server |
| Charts | Recharts |
| Drag-and-Drop | @hello-pangea/dnd |
| Mobile / PWA | Service Worker, Web App Manifest |

---

## 💻 Technical Stack

### Frontend
- **React 18 & TypeScript:** For robust and type-safe UI development.
- **React Query:** Advanced server-side state management with caching, background refetching, and optimistic updates.
- **Tailwind CSS & shadcn/ui:** Consistent, accessible, and high-performance component library.
- **Recharts:** Complex data visualization for operational metrics and KPI dashboards.
- **@hello-pangea/dnd:** Fluid drag-and-drop experience for Kanban boards.
- **Sonner:** Toast notifications with native swipe-to-dismiss on mobile.

### Backend & Infrastructure
- **Supabase Cloud:** Integrated database, authentication, storage, and realtime engine.
- **PostgreSQL RLS:** Row-Level Security enforcing multi-tenant data isolation at the database level.
- **Real-time Engine:** Supabase Realtime with debounced subscriptions for instant UI updates across all clients.
- **Edge Computing:** Deno-based serverless functions for critical business logic (notifications, geocoding, secure profile resolution).
- **WebSocket Server:** Custom Node.js agent for high-frequency hardware telemetry data.
- **Service Worker:** Manual implementation with network-first + cache fallback for offline support.

---

## 🔐 Security Features

| Feature | Implementation |
|---|---|
| **RBAC** | Granular permissions for Admin, Operator, Dev, and User roles |
| **JWT Authentication** | Secure sessions managed by Supabase Auth |
| **Row-Level Security** | Policies enforced directly at the PostgreSQL level |
| **Data Encryption** | Secure handling of sensitive telemetry and user data |
| **Input Validation** | Zod schema validation on all form inputs |
| **View-only Mode** | Chat and drag-and-drop locked for read-only Kanban users |

---

## 📋 Core Modules

| Module | Description | Status |
|---|---|---|
| 🎫 Ticketing / Kanban | Full lifecycle ticket management with SLA | ✅ Production |
| 💻 Asset Telemetry | Real-time hardware monitoring via WebSocket | ✅ Production |
| 👥 User Management | RBAC + department hierarchy | ✅ Production |
| 📊 Analytics Dashboard | KPIs, MTTR, SLA compliance charts | ✅ Production |
| 🔔 Notifications | Push + Webhook integration | ✅ Production |
| 💬 Internal Chat | Real-time messaging per ticket | ✅ Production |
| 📦 Inventory | Materials and resources tracking + Excel export | ✅ Production |
| 📱 PWA | Installable app with offline support | ✅ Production |

---

## 🖼️ Portfolio Note

> **Note:** This repository is a **showcase** of architectural design and technical proficiency. The source code for core business logic, proprietary algorithms, and private server implementations are kept in a **private repository** to protect intellectual property.

**What's included here:**
- ✅ Detailed technical documentation
- ✅ System architecture diagrams
- ✅ Technology stack specifications
- ✅ Visual previews and screenshots
- ✅ Configuration files demonstrating environment setup expertise

**What's in the private repository:**
- 🔒 Full application source code
- 🔒 Edge Functions implementations
- 🔒 Database migration scripts
- 🔒 WebSocket server implementation

---

## 📞 Contact

> Interested in the full implementation or have any questions?

**Talys Matheus Cordeiro Silva (Tcordeiro)**  
Feel free to reach out via [GitHub](https://github.com/Tcordeir0) or connect on [LinkedIn](https://www.linkedin.com/in/thalescordeiro/).

---

## 📜 License

Copyright © 2026 Talys Matheus Cordeiro Silva — All rights reserved.  
This project is for **demonstration purposes only**. Unauthorized copying, modification, distribution, or use of this project, in part or in whole, is strictly prohibited.

See [LICENSE](./LICENSE) for details.
