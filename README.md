# Logistics Management Hub (Showcase)

## 📌 Overview

**Logistics Management Hub** is a professional-grade corporate platform designed for high-scale operations management, centralizing integrated workflows for:

- **Support Ticketing System** - Complete ticket management with visual Kanban workflows.
- **IT Asset Monitoring** - Real-time hardware and device monitoring with telemetry.
- **User & Department Management** - Centralized management of employees and sectors.
- **Inventory Control** - Real-time tracking of materials, equipment, and resources.
- **Executive Analytics** - Performance dashboards and operational metrics.
- **Integrated Notifications** - Native push notifications and external webhooks.

### 🎯 Key Objective
To serve as a **unified hub** for communication and operations management, consolidating multiple workflows into an intuitive and responsive interface.

### 🖼️ Visual Preview

The platform features a modern, clean interface with dark/light mode support.

<p align="center">
  <img src="screenshots/preview-1.jpg" width="400" />
  <img src="screenshots/preview-2.jpg" width="400" />
</p>
<p align="center">
  <img src="screenshots/preview-3.jpg" width="400" />
  <img src="screenshots/preview-4.jpg" width="400" />
</p>

---

## 🏗️ System Architecture

The platform follows a modern serverless-first approach combined with real-time capabilities:

- **Frontend:** React 18, TypeScript, Vite, Tailwind CSS, shadcn/ui.
- **Backend (BaaS):** Supabase (PostgreSQL, Auth, Storage, Realtime).
- **Compute:** Edge Functions (Deno).
- **Real-time Telemetry:** Node.js WebSocket Server.

---

## 💻 Technical Stack

### Frontend
- **React 18 & TypeScript:** For robust and type-safe UI development.
- **React Query:** Advanced server-side state management.
- **Tailwind CSS & shadcn/ui:** Consistent, accessible, and high-performance styling.
- **Recharts:** Complex data visualization for operational metrics.
- **@hello-pangea/dnd:** Fluid drag-and-drop experience for Kanban boards.

### Backend & DevOps
- **Supabase Cloud:** Integrated database and authentication.
- **PostgreSQL RLS:** Row-Level Security ensuring data isolation at the database level.
- **Real-time Engine:** Subscriptions for instant UI updates.
- **Edge Computing:** Deno-based serverless functions for critical business logic.

---

## 🔐 Security Features

- **RBAC (Role-Based Access Control):** Granular permissions for Admin, Operator, and User roles.
- **JWT Authentication:** Secure sessions handled by Supabase Auth.
- **Row-Level Security (RLS):** Policies enforced directly at the SQL level.
- **Data Encryption:** Secure handling of sensitive telemetry data.

---

## 🖼️ Portfolio Purpose

**Note:** This repository is a **showcase** of architectural design and technical proficiency. The source code for core business logic, proprietary algorithms, and private server implementations are kept in a **private repository** to protect intellectual property.

### What's included here:
- Detailed technical documentation.
- System architecture diagrams.
- Technology stack specifications.
- Configuration files demonstrating environment setup expertise.

---

## 📜 License

Copyright © 2026. All rights reserved.
This project is for **demonstration purposes only**. Unauthorized copying, modification, distribution, or use of this project, in part or in whole, is strictly prohibited.
