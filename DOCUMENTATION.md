# Technical Documentation - Logistics Management Hub

> **Project Status:** Production Ready (Showcase Version)  
> **Architecture:** Serverless + Real-time Hybrid

---

## 1. Data Flow Architecture

The system is designed for low latency and high consistency:

1.  **Frontend (React):** Communicates via REST for standard CRUD and WebSockets for telemetry.
2.  **Supabase Realtime:** Syncs database changes (tickets, messages) across all clients in <1s.
3.  **WebSocket Layer:** Handles high-frequency telemetry data from remote agents.
4.  **Edge Functions:** Process heavy logic (geocoding, mass notifications) outside the main thread.

---

## 2. Database Schema (Conceptual)

### Core Entities
- **Profiles:** Extended user metadata with role-based attributes.
- **Tickets (Chamados):** State-machine driven workflow with status transitions.
- **Assets (Ativos):** Real-time tracking of hardware specs and health metrics.
- **Sectors:** Hierarchical organization of business units.

### Real-time Implementation
The system leverages PostgreSQL `LISTEN/NOTIFY` through Supabase Realtime to push updates directly to the UI without polling.

---

## 3. Advanced Modules

### 3.1 Kanban Engine
A sophisticated drag-and-drop interface that handles:
- Optimistic UI updates.
- Automatic assignment upon transition.
- Mandatory justification modals for terminal states (Cancelled/Finalized).

### 3.2 Asset Telemetry System
Custom WebSocket implementation receiving:
- CPU/RAM/Disk usage trends.
- Online/Offline status heartbeats.
- Geographical mapping based on network metadata.

### 3.3 Dynamic Form Builder
A JSON-driven form engine that allows administrators to create complex ticket templates with conditional logic without writing new code.

---

## 4. Key Performance Indicators (KPIs)

The dashboard provides real-time insights into:
- **MTTR (Mean Time to Repair):** Average time taken to resolve tickets.
- **Volume Trends:** Weekly and monthly ticket distributions.
- **SLA Compliance:** Visual alerts for critical and high-priority items.

---

## 5. Security & Compliance

### Row-Level Security (RLS) Examples
```sql
-- Security policy: Users can only see their own tickets
CREATE POLICY "Users can view own tickets" 
ON tickets FOR SELECT 
USING (auth.uid() = requester_id);

-- Security policy: Operators can see tickets in their assigned sectors
CREATE POLICY "Operators view sector tickets" 
ON tickets FOR SELECT 
USING (
  EXISTS (
    SELECT 1 FROM user_sectors 
    WHERE user_id = auth.uid() AND sector_id = tickets.sector_id
  )
);
```

---

*This document is part of a technical portfolio. For inquiries regarding the full implementation, please contact the developer.*
