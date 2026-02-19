<p align="right">
  <a href="./DOCUMENTATION.pt-br.md">🇧🇷 Versão em Português</a>
</p>

# Technical Documentation — Logistics Management Hub

> **Project Status:** Production Ready (Showcase Version)  
> **Architecture:** Serverless + Real-time Hybrid  
> **Version:** 1.0.7

---

## 1. Data Flow Architecture

The system is designed for **low latency** and **high consistency**:

1. **Frontend (React 18):** Communicates via REST for standard CRUD and WebSockets for telemetry.
2. **Supabase Realtime:** Syncs database changes (tickets, messages) across all clients in <1s using PostgreSQL `LISTEN/NOTIFY`.
3. **WebSocket Layer:** Handles high-frequency telemetry data from remote hardware agents.
4. **Edge Functions:** Process heavy logic (geocoding, mass notifications, webhook dispatch) outside the main thread.

### Data Flow Diagram
```
User Action
    │
    ├──► REST (PostgREST) ──► PostgreSQL ──► RLS Validation
    │                                │
    │                                └──► Realtime ──► All connected clients
    │
    └──► Edge Function ──► External APIs / Webhooks
```

---

## 2. Database Schema (Conceptual)

### Core Entities

| Entity | Description |
|---|---|
| **Profiles** | Extended user metadata with role-based attributes (Admin, Operator, Dev, User) |
| **Tickets (Chamados)** | State-machine driven workflow with status transitions and SLA tracking |
| **Assets (Ativos)** | Real-time tracking of hardware specs, health metrics, and telemetry |
| **Sectors** | Hierarchical organization of business units and department associations |
| **Groups** | Email groups for notification routing |
| **Inventory** | Materials, equipment, and resources with quantity control |

### Real-time Implementation
The system leverages PostgreSQL `LISTEN/NOTIFY` through Supabase Realtime to push updates directly to the UI without polling — ensuring instant synchronization for Kanban boards, chat messages, and ticket status changes.

### Row-Level Security Examples

```sql
-- Users can only view their own tickets
CREATE POLICY "Users can view own tickets"
ON tickets FOR SELECT
USING (auth.uid() = requester_id);

-- Operators can view tickets in their assigned sectors
CREATE POLICY "Operators view sector tickets"
ON tickets FOR SELECT
USING (
  EXISTS (
    SELECT 1 FROM user_sectors
    WHERE user_id = auth.uid() AND sector_id = tickets.sector_id
  )
);

-- Admins have full access
CREATE POLICY "Admins have full access"
ON tickets FOR ALL
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);
```

---

## 3. Edge Functions

Edge Functions are deployed as Deno serverless functions via Supabase:

| Function | Purpose |
|---|---|
| `send-notification` | Dispatches push notifications + external webhook calls |
| `admin-create-user` | Creates users with RBAC role assignment (bypasses anon restrictions) |
| `telemetry-processor` | Aggregates and stores hardware telemetry data |

---

## 4. Advanced Modules

### 4.1 Kanban Engine

A sophisticated drag-and-drop interface built with `@hello-pangea/dnd` that handles:
- **Optimistic UI updates** — Immediate visual feedback before server confirmation.
- **Automatic assignment** upon ticket transition to "In Progress".
- **Mandatory justification modals** for terminal states (Cancelled / Finalized).
- **Access-controlled columns** — Role-based restrictions on which status transitions are allowed.

### 4.2 Asset Telemetry System

Custom WebSocket implementation receiving high-frequency data from hardware agents:
- **CPU / RAM / Disk** usage trends with historical charts.
- **Online/Offline status** heartbeats with configurable timeout thresholds.
- **Geographical mapping** based on network metadata (IP-based location).
- **Alert thresholds** — Automated notifications when metrics exceed defined limits.

### 4.3 Dynamic Form Builder

A JSON-driven form engine that allows administrators to create complex ticket templates with conditional logic **without writing any new code**. Supports:
- Text fields, dropdowns, checkboxes, file attachments.
- Conditional visibility rules (show field X only if field Y equals Z).
- Required field validation with custom error messages.

### 4.4 Internal Chat (Real-time)

Real-time messaging integrated into each ticket's context:
- Messages stored in PostgreSQL with Realtime subscriptions.
- Supports file attachments and read receipts.
- Scoped per ticket — each conversation is isolated to its ticket context.

---

## 5. Key Performance Indicators (KPIs)

The analytics dashboard provides real-time insights into:

| KPI | Definition |
|---|---|
| **MTTR** | Mean Time to Repair — average resolution time per ticket |
| **Volume Trends** | Weekly and monthly ticket distribution charts |
| **SLA Compliance** | Visual alerts for tickets breaching priority-based SLA thresholds |
| **Asset Health Score** | Aggregate health metric across all monitored hardware |
| **Open / In Progress / Closed** | Real-time ticket status distribution |

---

## 6. Security & Compliance

### Authentication Flow
1. User submits credentials → Supabase Auth validates and issues JWT.
2. JWT included in all subsequent API requests.
3. PostgreSQL RLS policies evaluate `auth.uid()` on every query — **zero bypass possible** from the client.

### Permission Matrix

| Action | User | Operator | Dev | Admin |
|---|---|---|---|---|
| View own tickets | ✅ | ✅ | ✅ | ✅ |
| View sector tickets | ❌ | ✅ | ✅ | ✅ |
| Create tickets | ✅ | ✅ | ✅ | ✅ |
| Manage users | ❌ | ❌ | ✅ | ✅ |
| Access analytics | ❌ | ❌ | ✅ | ✅ |
| System configuration | ❌ | ❌ | ❌ | ✅ |

---

*This document is part of a technical portfolio. For inquiries regarding the full implementation, please contact the developer via [GitHub](https://github.com/Tcordeir0).*
