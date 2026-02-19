# Changelog

All notable changes to this project will be documented in this file.  
Todas as mudanças notáveis neste projeto serão documentadas aqui.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.0.7] — 2026-02-17

### Fixed / Corrigido
- Fixed sidebar color inconsistencies in the Users page (`UserFolderSidebar`)
- Added missing "Import" button to the Admin Users toolbar
- Corrected dark mode token usage across multiple UI components
- Fixed notification messages incorrectly displaying "Urgency" instead of the ticket opener's name

---

## [1.0.6] — 2026-02-16

### Added / Adicionado
- Webhook payload now includes the group email address alongside the group name
- Default Kanban permissions (Operational/Billing, View Mode) for users created under the "Filial" sector
- Asset dropdown in user management now populates correctly from the database

### Fixed / Corrigido
- Resolved 403 Forbidden error when creating users with `dev` profile
- Resolved 409 Conflict error when deleting users with transfer history records
- Fixed chat responsiveness on mobile screen sizes across all chat interfaces

---

## [1.0.5] — 2026-02-14

### Added / Adicionado
- Internal real-time chat integrated directly into ticket workflows
- Notification system with external webhook support
- Group email management for department notifications

---

## [1.0.4] — 2026-02-13

### Added / Adicionado
- Executive Analytics dashboard with KPIs, MTTR, SLA compliance charts
- Dynamic form builder for administrators (JSON-driven, no-code ticket templates)
- Drag-and-drop Kanban with mandatory justification modals for terminal states

---

## [1.0.3] — 2026-02-01

### Added / Adicionado
- IT Asset telemetry system via custom WebSocket agent (CPU, RAM, Disk)
- Geographical mapping based on network metadata
- Online/Offline heartbeat status for all registered assets

---

## [1.0.2] — 2026-01-20

### Added / Adicionado
- Role-Based Access Control (RBAC) with Admin, Operator, Dev, and User roles
- Row-Level Security (RLS) policies enforced at the PostgreSQL level
- Multi-sector hierarchy and user-sector association

---

## [1.0.1] — 2026-01-10

### Added / Adicionado
- Full dark/light mode support with theme persistence
- Responsive layout for mobile and tablet devices

---

## [1.0.0] — 2026-01-05

### Added / Adicionado
- Initial release of the Logistics Management Hub
- Support ticketing system with Kanban board
- User and department management
- Inventory control module
- Supabase integration (PostgreSQL, Auth, Realtime, Edge Functions)
