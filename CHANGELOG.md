# Changelog

All notable changes to this project will be documented in this file.  
Todas as mudanças notáveis neste projeto serão documentadas aqui.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.0.7] — 2026-02-18

### Fixed / Corrigido
- Fixed sidebar color inconsistencies in the Users page (`UserFolderSidebar`)
- Added missing "Import" button to the Admin Users toolbar
- Corrected dark mode token usage across multiple UI components (replaced hardcoded hex values with MUI theme variables)
- Fixed notification messages incorrectly displaying "Urgency" instead of the ticket opener's name

---

## [1.0.6] — 2026-02-16

### Added / Adicionado
- Webhook payload now includes the group email address alongside the group name
- Default Kanban permissions (Operational/Billing, View Mode) for users created under the branch sector
- Asset dropdown in user management now populates correctly from the database

### Fixed / Corrigido
- Resolved 403 Forbidden error when creating users with `dev` profile
- Resolved 409 Conflict error when deleting users with transfer history records
- Fixed chat responsiveness on mobile screen sizes across all chat interfaces

---

## [1.0.5] — 2026-02-06

### Added / Adicionado
- **Automated SLA System** — Replaced manual "Urgency" field with automatic time-based SLA: ✅ On Time (<3h), 🔴 Delayed (≥3h)
- **Search by License Plate & CT-e** — Universal search across Kanban boards using normalized in-memory index (removes accents, hyphens, spaces for flexible matching)
- **Excel Export for Attendance History** — Auditable file with username and export date in filename; includes ticket ID, title, requester, sector, status, SLA, license plate, CT-e, timestamps, and total time
- **Kanban Mobile Optimization** — Fluid column transition animations with scale/opacity effects, native swipe gestures, and stable scroll-snap without mid-scroll freezing
- **Login Toast** — Migrated from shadcn/radix to Sonner with native swipe-to-dismiss and 3-second auto-dismiss; no longer blocks the profile completion modal
- **Version Indicator** — "v1.0.5" text displayed in Sidebar (desktop and mobile) below the logout button

### Fixed / Corrigido
- Dashboard SLA chart incorrectly showed 100% "On Time" — fixed missing `created_at` in query causing `Date.now()` fallback
- Unauthorized route access — users without operator permission are silently redirected to Home

---

## [1.0.4] — 2026-02-04

### Added / Adicionado
- **Operational Kanban** — Dedicated route and board for operational team with separate access control (`kanban_acesso = 'OPERACIONAL'`)
- **Dynamic Form Persistence** — Form data saved to `dados_formulario` (JSONB) column for structured querying and card display
- **Smart Field Extraction** — Intelligent extraction of license plate, CT-e, freight note, and client from both structured JSONB and legacy description bullets
- **View-only Mode** — Chat and drag-and-drop disabled with lock indicator for `visualizar_kanban = true` users
- **Secure Public Profiles** — Edge Function resolves requester/responsible names safely without exposing PII (no email/phone in RLS bypass)
- **Status Mapping Fix** — Operational Kanban now maps to correct database enum values: `novo → Em Fila`, `em_andamento → Em Andamento`, `aguardando_retorno → Pendente`

### Fixed / Corrigido
- Operational sector lookup error (PGRST116) when multiple sectors matched partial name — resolved with normalization + best-match selection
- License plate not appearing on Operational Kanban cards — `extractCamposChamado()` now correctly handles bullet-format descriptions

---

## [1.0.3] — 2026-01-22

### Fixed / Corrigido
- **Chat & Realtime Stability (Hotfix)** — Subscription loop (CLOSED/SUBSCRIBED) fixed by stabilizing `useRef` for message and presence channels; `useEffect` now depends only on `id`, not on unstable `user` object reference
- **Transfer Modal** — 15s safety timeout to unblock UI; transferred ticket now sets `status: 'novo'` (was `'transferido'`, which was invisible on Kanban); `sendNotification` called asynchronously to avoid blocking UI
- **Kanban Debounce** — 500ms debounce on Realtime refetch prevents request storms from multiple rapid UPDATE events

---

## [1.0.2] — 2026-01-21

### Added / Adicionado
- **Floating Onboarding Tips** — Context-aware tips per route (Dashboard, New Ticket, Profile, Sectors, My Tickets) with 30s display, slide+fade animation
- **User Cancel Modal** — Users can now cancel their own tickets with mandatory reason field (min 10 characters)
- **Operator Availability in Transfer Modal** — Shows availability badges (🟢 On-call, 🔵 Working hours, 🔴 Off) and sorts available operators first
- **Responsive Charts** — Dashboard pie chart refactored with adaptive container heights and responsive legend grid

### Fixed / Corrigido
- Operators marked as "on-call" incorrectly appearing as "off" — updated SQL function `is_user_available()` with timezone support (`America/Sao_Paulo`) and overnight shift handling
- Toast notifications overflowing mobile screens — `max-width: calc(100vw - 24px)`, iOS safe-area support, `word-break: break-word`
- PWA icons standardized to official app icon

---

## [1.0.1] — 2026-01-19

### Added / Adicionado
- **Full Mobile Compatibility** — Adaptive sidebar (drawer on mobile, fixed on desktop), hamburger button, responsive layouts across all pages
- **Touch Swipe Gestures** — `useSwipeGesture` hook: swipe from left edge (40px threshold) to open sidebar; swipe left to close; passive event listeners for performance
- **Progressive Web App (PWA)** — Web App Manifest, manual Service Worker (network-first + cache fallback), iOS meta tags, safe-area support for Dynamic Island
- **Mobile Kanban** — Horizontal scroll with 800px minimum container for drag-and-drop usability on touch devices
- **iOS Layout Fixes** — `env(safe-area-inset-*)` padding, no `100vw` or `w-screen` usage, GPU-accelerated sidebar animation via CSS `transform`

---

## [1.0.0] — 2026-01-05

### Added / Adicionado
- Initial release of the IntraNet
- Support ticketing system with Kanban board
- User and department management
- Inventory control module
- Supabase integration (PostgreSQL, Auth, Realtime, Edge Functions)
- Executive analytics dashboard

---

*Developed by **Talys Matheus Cordeiro Silva (Tcordeiro)***
