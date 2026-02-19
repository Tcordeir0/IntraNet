# Technical Documentation

<p align="right">
  <a href="./DOCUMENTATION.pt-br.md">🇧🇷 Versão em Português</a>
</p>

> **Developed by:** Talys Matheus Cordeiro Silva (Tcordeiro) — Professional Portfolio Showcase

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Data Flow](#2-data-flow)
3. [Security Model (RBAC + RLS)](#3-security-model-rbac--rls)
4. [Core Modules](#4-core-modules)
5. [Mobile & PWA Implementation](#5-mobile--pwa-implementation)
6. [Real-time & Performance](#6-real-time--performance)
7. [Edge Functions](#7-edge-functions)
8. [Permission Matrix](#8-permission-matrix)

---

## 1. Architecture Overview

The system follows a **serverless-first** architecture leveraging Supabase as the primary Backend-as-a-Service (BaaS) layer:

```
┌─────────────────────────────────────────────────────────────┐
│              Client Layer (React 18 + TypeScript)            │
│           Vite · Tailwind · shadcn/ui · React Query          │
└──────────────────────────┬──────────────────────────────────┘
                           │
            ┌──────────────┼──────────────┐
            │              │              │
     ┌──────▼──────┐ ┌─────▼──────┐ ┌────▼───────┐
     │  REST API   │ │  Realtime  │ │  WebSocket  │
     │ (PostgREST) │ │  (LISTEN/  │ │ (Telemetry) │
     └──────┬──────┘ │  NOTIFY)   │ └────┬───────┘
            │        └─────┬──────┘      │
     ┌──────▼──────────────▼─────────────▼──────┐
     │           Supabase Platform               │
     │  PostgreSQL · Auth · Storage · Edge Fn   │
     └───────────────────────────────────────────┘
```

### Tech Stack Summary

| Layer | Technology | Purpose |
|---|---|---|
| Frontend | React 18 + TypeScript | Typed, reactive UI |
| Build | Vite 5 | Fast HMR, ESM output |
| Styling | Tailwind CSS + shadcn/ui | Design system |
| State | React Query (TanStack) | Server state + caching |
| Database | PostgreSQL (Supabase) | Primary data store |
| Auth | Supabase Auth (JWT) | Session management |
| Realtime | Supabase Realtime | WebSocket subscriptions |
| Edge | Deno (Edge Functions) | Serverless compute |
| Telemetry | Node.js WebSocket Agent | Hardware monitoring |
| Charts | Recharts | Data visualization |
| DnD | @hello-pangea/dnd | Kanban drag-and-drop |
| Mobile | Service Worker + Manifest | PWA support |

---

## 2. Data Flow

### Ticket Lifecycle Flow

```
User → New Ticket Form → Sector/Topic → Structured JSONB
                                          ↓
                                   PostgreSQL RLS
                                          ↓
                        +----------------+----------------+
                        |                                 |
                 Realtime Notify                    Edge Function
                 (UI Push Update)              (send-notification)
                        |                                 |
                 Kanban Board                  Push + Webhook Dispatch
                 auto-updates                  (group email + external)
```

### Smart Field Extraction

Dynamic form data is persisted in a `dados_formulario` (JSONB) column, enabling:

```typescript
// Priority 1: Structured JSONB from dynamic form
const placa = dados_formulario?.placa
           || dados_formulario?.placaVeiculo

// Priority 2: Intelligent regex on legacy description bullets
|| extractFromBullets(descricao, 'Placa')

// Normalized for search (removes accents, hyphens, spaces)
|| normalized(rawText)
```

---

## 3. Security Model (RBAC + RLS)

### Role Hierarchy

| Role | Code | Access Level |
|---|---|---|
| Admin | `admin` | Full — all data, all users |
| Developer | `dev` | Full + technical panels |
| Operator | `operador` | Own tickets + assigned tickets |
| User | `usuario` | Own submissions only |
| Viewer | `visualizar_kanban` | Read-only Kanban access |

### Row-Level Security (RLS) Concept

All tables enforce RLS policies that evaluate the authenticated JWT to determine row visibility:

- **Users** see only their own tickets unless assigned as operator.
- **Operators** see tickets in their sector or directly assigned to them.
- **Admins** bypass all restrictions with full read/write access.
- **Viewers** access Kanban read-only — chat and drag-and-drop are locked in the UI.

### Secure Profile Resolution

To resolve user display names without exposing private data (email/phone), an **Edge Function** bypasses RLS using the service role key, returning only `{ id, nome }` — never full profile data.

---

## 4. Core Modules

### 4.1 Ticketing & Kanban

The ticket system supports two independent Kanban boards with separate access controls:

- **Standard Kanban** — for general support operators
- **Operational Kanban** — for logistics/freight operators (`kanban_acesso = 'OPERACIONAL'`)

Cards display dynamically extracted fields:

| Field | Source |
|---|---|
| License Plate | `dados_formulario.placa` or bullet extraction |
| CT-e / Freight Note | `dados_formulario.cte` or bullet extraction |
| Client | `dados_formulario.cliente` or bullet extraction |
| SLA Status | Computed from `created_at` vs 3h threshold |
| Responsible | Assigned operator name with avatar |

### 4.2 SLA System

Replaced the legacy manual "Urgency" field with an automatic time-based metric:

```typescript
export const calcularStatusPrazo = (
  created_at: string,
  finalizado_at: string | null
): 'No Prazo' | 'Em Atraso' => {
  const prazoLimite = new Date(
    new Date(created_at).getTime() + 3 * 60 * 60 * 1000 // 3h
  );
  if (finalizado_at) return 'No Prazo';
  return new Date() > prazoLimite ? 'Em Atraso' : 'No Prazo';
};
```

Displayed as a colored badge (green/red) on all Kanban cards and in the reporting dashboard.

### 4.3 Asset Monitoring

Real-time hardware telemetry via a hybrid approach:

- **PostgreSQL**: Persistent asset metadata (specs, location, ownership)
- **WebSocket Server**: Live CPU/RAM/Disk usage pushed from local Node.js agents
- **UI**: Folder-organized tree view with online/offline counters per sector

### 4.4 Analytics & Reports

KPI metrics computed via server-side PostgreSQL queries and displayed via Recharts:

- Daily evolution chart (created vs finalized tickets)
- SLA compliance donut chart (On Time / Delayed distribution)
- Demand per sector/branch bar charts
- Most active users per branch
- MTTR (Mean Time To Resolution) calculation

### 4.5 Inventory

Materials and IT assets tracking with:

- Room-based organization (server rooms, storage, offices)
- Real-time quantity tracking
- Excel export with user-stamped filename (`InventarioSala_YYYY-MM-DD_<user>.xlsx`)

---

## 5. Mobile & PWA Implementation

### Service Worker Strategy

```
Request → Network?
           Yes → Serve + Cache update
           No  → Serve from Cache
                   No Cache → Fallback page
```

### Web App Manifest

```json
{
  "name": "Logistics Management Hub",
  "short_name": "IntraNet",
  "display": "standalone",
  "orientation": "portrait",
  "theme_color": "#0f172a",
  "background_color": "#0f172a",
  "icons": [
    { "src": "/icons/icon-192.png", "sizes": "192x192", "type": "image/png" },
    { "src": "/icons/icon-512.png", "sizes": "512x512", "type": "image/png" }
  ]
}
```

### iOS Safe Area Support

```css
/* iPhone notch / Dynamic Island */
padding-top: env(safe-area-inset-top);
padding-bottom: env(safe-area-inset-bottom);
padding-left: env(safe-area-inset-left);
padding-right: env(safe-area-inset-right);
```

> No `100vw` or `w-screen` — uses `100%` with `overflow: hidden` to prevent iOS bounce scroll bleed.

### Swipe Gesture Implementation

```typescript
// Custom hook: useSwipeGesture
// - Touch start: record X position if within 40px of left edge
// - Touch move: calculate delta, update CSS transform for visual feedback
// - Touch end: if delta > 80px, open sidebar; else snap back
// Passive listeners for 60fps scrolling performance
```

### Responsive Kanban

- Horizontal scroll container with `min-width: 800px` columns
- `scroll-snap-type: x mandatory` for fluid column navigation on touch
- Drag-and-drop remains functional on touch (hello-pangea/dnd touch support)

---

## 6. Real-time & Performance

### Subscription Stability

Chat subscriptions are stabilized using `useRef` to prevent loop caused by object identity changes:

```typescript
// Problematic: unstable reference causes useEffect to re-run
useEffect(() => { subscribe(user) }, [user])

// Stable: extract primitive, use ref for non-primitive
const userIdRef = useRef(user?.id)
useEffect(() => { subscribe(userIdRef.current) }, [user?.id])
```

### Kanban Debounce

Realtime updates debounced at 500ms to prevent request storms from rapid consecutive DB events:

```typescript
const debouncedRefetch = useCallback(
  debounce(() => refetchChamados(), 500), []
);
channel.on('postgres_changes', { event: '*', table: 'chamados' }, debouncedRefetch);
```

### Safety Timeouts

Critical operations (message send, attachment upload, ticket transfer) include 15-second safety timeouts to unblock the UI if the server fails to respond.

---

## 7. Edge Functions

Deployed on Deno runtime, invoked via `supabase.functions.invoke()`:

| Function | Purpose |
|---|---|
| `send-notification` | Dispatches push notifications + external webhook with group email |
| `admin-create-user` | Creates users with service role — supports all profile types including `dev` |
| `get-public-profile` | Resolves user display name safely without exposing PII |

All functions require a valid JWT in the `Authorization` header (verify_jwt = true).

---

## 8. Permission Matrix

| Feature | Admin | Dev | Operator | User | Viewer |
|---|---|---|---|---|---|
| View all tickets | ✅ | ✅ | Own sector | Own only | ✅ (read) |
| Create tickets | ✅ | ✅ | ✅ | ✅ | ❌ |
| Assign/transfer tickets | ✅ | ✅ | ✅ | ❌ | ❌ |
| Finalize tickets | ✅ | ✅ | ✅ | ❌ | ❌ |
| Drag Kanban cards | ✅ | ✅ | ✅ | ❌ | ❌ |
| Access Admin panels | ✅ | ✅ | ❌ | ❌ | ❌ |
| View analytics | ✅ | ✅ | ✅ | ❌ | ❌ |
| Export Excel | ✅ | ✅ | ✅ | ❌ | ❌ |
| Manage users | ✅ | ✅ | ❌ | ❌ | ❌ |
| Asset monitoring | ✅ | ✅ | ✅ | ❌ | ❌ |

---

*Documentation maintained by **Talys Matheus Cordeiro Silva (Tcordeiro)***
