# buku-induk-siswa - Work Plan

## TL;DR (For humans)

**Who this is for and what changes for them:** Staf admin, wali kelas, wali kamar, asatidz, dan pimpinan pesantren SMP yang saat ini mengelola data santri manual via spreadsheet/kertas. Setelahnya: satu web app internal dengan login role-based, dashboard real-time, CRUD santri lengkap (foto, dokumen KK/Akta/Ijazah/SKK/SKTM ke Google Drive), manajemen kelas & kamar (ikhwan/akhwat), tracking status kesehatan (sakit/pulang/izin dengan detail), audit log otomatis semua perubahan, dan workflow assignment wali dengan persetujuan TU — semua diakses via browser desktop/mobile.

**What you'll get:** Aplikasi SvelteKit 3 + Flowbite Svelte + Tailwind v4 + Supabase (PostgreSQL + Auth + Realtime) + Google Drive API, deploy ke Cloudflare Pages. Termasuk: 14 tabel database dengan RLS/trigger, 5 role + RBAC dinamis kelola lewat UI, UI-first development (prototype dummy → review → backend), light/dark theme, responsive, CI/CD GitHub Actions.

**Why this approach:** (1) **UI-first per fitur** memastikan tampilan disetujui sebelum backend — menghindari rework; (2) **Supabase database-only + Google Drive** menghemat storage gratis Supabase (500MB) sambil memanfaatkan 100GB Google Workspace for Education; (3) **Dynamic RBAC + audit log via DB triggers** memberikan accountability penuh tanpa hardcode permissions.

**What it will NOT do:** Multi-tenancy (sekolah lain), portal santri/orang tua, nilai/absensi/jadwal/keuangan, notifikasi WA/email, mobile app native, OAuth/SSO, soft delete, i18n, Supabase Storage, field-level permissions.

**Effort:** XL
**Risk:** Medium - Google Drive service account setup & domain-wide delegation memerlukan akses Google Workspace admin; SvelteKit 3 masih beta/RC (adapter-cloudflare support OK); audit log volume perlu retention policy ketat.

**Decisions to sanity-check:** UI Library: Flowbite Svelte (bukan shadcn-svelte/DaisyUI); Auth: Supabase Email/Password only; File storage: Google Drive service account; Folder structure: per-tahun-ajaran; Audit log: ALL tables via triggers, retention 1yr; Realtime: status_siswa only; Assignment: checklist + TU approval; Data change proposal: wali propose → TU diff review → approve.

Your next move: approve to execute via `$ulw-execute`.

---

> TL;DR (machine): XL effort, Medium risk, SvelteKit3+Flowbite+Supabase+GoogleDrive admin app with 14 tables, 5 roles, dynamic RBAC, audit triggers, UI-first workflow, Cloudflare Pages deploy

## Scope

### Affected user and ideal state

**Affected user:** 7 personas: Superadmin (full control), Staff TU (opsional data entry + approval), Wali Kelas (kelas + assignment + propose changes), Wali Kamar (kamar + kesehatan + assignment + propose changes), Asatidz (read-only assigned), Pimpinan (dashboard strategic), Developer (maintainable codebase)

| Row | Statement | Reason |
| --- | --- | --- |
| IS-1 | Superadmin: login → kelola user, permission matrix UI, audit log viewer, system settings (school name, logo, Google Drive config), semua CRUD, tanpa hardcode | Single source of truth permissions & accountability |
| IS-2 | Staff TU: login → dashboard stats, siswa/kelas/kamar CRUD, Excel import/export, PDF export, approve assignment proposals, approve data change proposals, kelola tahun ajaran | Operational efficiency, compliance, gatekeeper accuracy |
| IS-3 | Wali Kelas: login → view assigned kelas (checklist-assign siswa), siswa list dengan kamar, export class list, update status kesehatan, view audit log own kelas, propose data changes untuk TU approve | Class management tanpa dependency admin, field-level accuracy |
| IS-4 | Wali Kamar: login → view assigned kamar (ikhwan/akhwat), siswa in kamar, status kesehatan real-time (sakit/pulang/izin + deskripsi + tanggal), log entry, view audit log own kamar, checklist-assign siswa, propose data changes | Room supervision, student welfare, field-level accuracy |
| IS-5 | Asatidz: login → view assigned kelas/siswa, search, export, audit log read-only | Teaching support tanpa admin overhead |
| IS-6 | Pimpinan: login → dashboard total aktif per kelas/kamar/status kebutuhan/angkatan, audit log summary | Strategic visibility untuk decision making |
| IS-7 | Developer: Type-safe SvelteKit 3 codebase, modular structure, Zod validation, Supabase RLS, dynamic RBAC, Google Drive integration, CI/CD automated | Maintainable, secure, scalable, zero-config deployment |

| Row | Statement | Reason |
| --- | --- | --- |
| GAP-1 | No authentication system | Need secure login for all 5 roles | Todo 1-3 |
| GAP-2 | No database schema | Greenfield project | Todo 4 |
| GAP-3 | No UI component library + theme | Need admin-grade components | Todo 5-6 |
| GAP-4 | No mock service layer contract | UI-first workflow requires interface | Todo 7 |
| GAP-5 | No Dashboard UI prototype | Visual approval before backend | Todo 8 |
| GAP-6 | No Dashboard backend integration | Connect to real data | Todo 9 |
| GAP-7 | No Siswa CRUD UI prototype | Visual approval before backend | Todo 10 |
| GAP-8 | No Siswa CRUD backend integration | Connect to real data + Google Drive docs | Todo 11 |
| GAP-9 | No Kelas Management UI prototype | Visual approval before backend | Todo 12 |
| GAP-10 | No Kelas Management backend integration | Connect to real data + assignment workflow | Todo 13 |
| GAP-11 | No Kamar Management UI prototype | Visual approval before backend | Todo 14 |
| GAP-12 | No Kamar Management backend integration | Connect to real data + assignment workflow | Todo 15 |
| GAP-13 | No Tahun Ajaran UI prototype | Visual approval before backend | Todo 16 |
| GAP-14 | No Tahun Ajaran backend integration | Connect to real data + drives Drive folders | Todo 17 |
| GAP-15 | No Status Kesehatan UI prototype | Visual approval before backend | Todo 18 |
| GAP-16 | No Status Kesehatan backend integration + Realtime | Connect to real data + live updates | Todo 19 |
| GAP-17 | No Dokumen/Google Drive UI prototype | Visual approval before backend | Todo 20 |
| GAP-18 | No Dokumen/Google Drive backend integration | Connect to Drive API + per-tahun-ajaran folders | Todo 21 |
| GAP-19 | No Audit Log UI prototype | Visual approval before backend | Todo 22 |
| GAP-20 | No Audit Log backend integration (triggers + all tables) | Database-enforced audit trail | Todo 23 |
| GAP-21 | No RBAC/Settings UI prototype | Visual approval before backend | Todo 24 |
| GAP-22 | No RBAC/Settings backend integration | Dynamic permissions + Google Drive config | Todo 25 |
| GAP-23 | No Assignment Workflow UI prototype | Visual approval before backend | Todo 26 |
| GAP-24 | No Assignment Workflow backend integration | Proposal → TU approval flow | Todo 27 |
| GAP-25 | No Data Change Proposal UI prototype | Visual approval before backend | Todo 28 |
| GAP-26 | No Data Change Proposal backend integration | Propose → diff review → approve | Todo 29 |
| GAP-27 | No Excel Import/Export + PDF Export | Bulk data & reporting | Todo 30-31 |
| GAP-28 | No responsive polish + E2E tests + deploy config | Production readiness | Todo 32-34 |

### Must have

- SvelteKit 3 + Svelte 5 + TypeScript strict
- Flowbite Svelte + Tailwind v4 + CSS variables theming (light/dark toggle, SSR-safe)
- Supabase: PostgreSQL, Auth (email/password), Realtime (status_siswa only)
- Database: 14 tables with PK identity, FK indexes, CHECK constraints, RLS policies, triggers
- Auth: Login, protected routes, 5 roles, password change, profile update
- Dynamic RBAC: Resource-action matrix, role-permission mapping, superadmin UI, middleware enforcement
- Dashboard: Stats cards, audit summary, quick actions
- Siswa CRUD: DataTable (search, filter, cursor pagination), modal form (nested orang_tua, kamar, dokumen), detail view (timeline + dokumen tabs)
- Kelas/Kamar CRUD + Assignment Proposal Workflow (checklist → TU approve)
- Tahun Ajaran CRUD (drives Google Drive folder structure)
- Status Kesehatan: sakit (deskripsi+tanggal), pulang (deskripsi+tanggal_mulai+tanggal_selesai), izin (deskripsi+tanggal), aktif — history + Realtime
- Dokumen Siswa: 6 jenis (foto/kk/akta/ijazah/skk/sktm), Google Drive API (service account), per-tahun-ajaran folders
- Audit Log: DB triggers ALL mutating tables + auth events, filterable UI, cursor pagination, Excel export
- Wali Assignment: Checklist UI → proposal → TU approve → create assignment records
- Data Change Proposal: Wali propose → TU diff review → approve → apply + audit
- Excel Import (drag-drop, Zod validate, batch upsert NISN, error report), Excel Export, PDF Export (A4 portrait)
- Light/Dark toggle persisted, all pages responsive (desktop-first, mobile-usable)
- Cloudflare Pages deploy via adapter-cloudflare, GitHub Actions CI/CD

### Must NOT have (guardrails, anti-slop, scope boundaries)

- Multi-tenancy / multi-school
- Student/parent portal
- Academic grades, attendance tracking, schedule/timetable
- Financial/billing/payment processing
- SMS/WhatsApp/Email notifications
- Mobile app (React Native/Flutter)
- OAuth/SSO (Google, Microsoft)
- Soft delete / trash
- Data versioning beyond audit_log + status_siswa history
- Multi-language (i18n)
- Supabase Storage (explicitly excluded)
- Field-level permissions (resource-action only)
- Google Drive folder management beyond per-tahun-ajaran categories

## Verification strategy

> Zero human intervention - all verification is agent-executed.

- Test decision: **tests-after** + **Vitest** (unit/utils/Zod/RBAC), **@testing-library/svelte** (component), **Playwright** (E2E critical flows)
- Evidence: `.omo/evidence/ulw/<session>/<goalId>/a<attempt>/task-<N>-buku-induk-siswa.<ext>`

## Execution strategy

### Parallel execution waves

Target 5-8 todos per wave. Each feature follows UI-first: **Prototype (dummy) → Review Gate → Backend Integration → Polish**. Waves group by dependency layer.

### Dependency matrix

| Todo | Depends on | Blocks | Can parallelize with |
| --- | --- | --- | --- |
| 1-3 (Foundation: scaffold, Flowbite, Supabase client) | — | 4-34 | — |
| 4 (Database schema migration) | 3 | 5-34 | — |
| 5 (Auth system) | 3,4 | 6-34 | — |
| 6 (Mock service layer contract) | 4 | 7-34 | — |
| 7 (RBAC middleware) | 5 | 8-34 | — |
| 8-9 (Dashboard: UI → Backend) | 5,6,7 | 10-34 | — |
| 10-11 (Siswa CRUD: UI → Backend) | 5,6,7,4 | 12-34 | 12-17 |
| 12-13 (Kelas: UI → Backend) | 5,6,7,4 | 14-34 | 10-11, 14-17 |
| 14-15 (Kamar: UI → Backend) | 5,6,7,4 | 16-34 | 10-13, 16-17 |
| 16-17 (Tahun Ajaran: UI → Backend) | 5,6,7,4 | 18-34 | 10-15 |
| 18-19 (Status Kesehatan: UI → Backend + Realtime) | 5,6,7,4 | 20-34 | 10-17 |
| 20-21 (Dokumen/Drive: UI → Backend) | 5,6,7,4,16 | 22-34 | 18-19 |
| 22-23 (Audit Log: UI → Backend triggers) | 5,6,7,4 | 24-34 | 18-21 |
| 24-25 (RBAC/Settings: UI → Backend) | 5,6,7 | 26-34 | 22-23 |
| 26-27 (Assignment Workflow: UI → Backend) | 5,6,7,12,14 | 28-34 | 24-25 |
| 28-29 (Data Change Proposal: UI → Backend) | 5,6,7,10 | 30-34 | 26-27 |
| 30-31 (Excel/PDF Export) | 5,6,7,10 | 32-34 | 28-29 |
| 32 (Responsive polish all pages) | 8-31 | 33-34 | — |
| 33 (Playwright E2E critical flows) | 32 | 34 | — |
| 34 (Deploy config + CI/CD) | 3 | 33 | — |

## Todos

> Implementation + Test = ONE todo. Never separate.

### Wave 1: Foundation

- [ ] 1. **SvelteKit 3 Scaffold + TypeScript + Tailwind v4 + ESLint + Prettier + Vitest**
  What to do / Must NOT do: `pnpm dlx sv create buku-induk-siswa --yes --template skeleton --add tailwindcss,eslint,prettier,vitest` → verify Svelte 5 runes, `@tailwindcss/vite` plugin, `tsconfig.json` strict mode. NO Supabase Storage config.
  Closes: GAP-1, GAP-2 (partial)
  Parallelization: Wave 1 | Blocked by: — | Blocks: 2-34
  References: SvelteKit 3 migration guide; shadcn-svelte Tailwind v4 migration (for `@tailwindcss/vite` pattern)
  Acceptance criteria: `pnpm run dev` starts on localhost:5173; `pnpm run check` passes (tsc --noEmit); `pnpm run lint` passes; `pnpm run test` runs Vitest
  QA scenarios: `pnpm run build` succeeds; `pnpm run preview` serves production build; Evidence: `attemptDir/task-1-buku-induk-siswa.log`
  Commit: Y | feat(scaffold): initialize SvelteKit 3 project with TS, Tailwind v4, linting, testing

- [ ] 2. **Flowbite Svelte Installation + Theme Setup (CSS variables, light/dark toggle)**
  What to do / Must NOT do: `pnpm add flowbite-svelte flowbite` → configure `tailwind.config.cjs` (Flowbite plugin), `app.css` with `@import "tailwindcss"; @plugin "flowbite"; @custom-variant dark (&:where(.dark, .dark *));` → CSS variables for theme (OKLCH) → `ThemeToggle.svelte` component (localStorage + cookie for SSR) → `+layout.svelte` imports theme. NO DaisyUI, NO shadcn-svelte.
  Closes: GAP-3
  Parallelization: Wave 1 | Blocked by: 1 | Blocks: 5-34
  References: Flowbite Svelte install guide; Tailwind v4 dark mode docs
  Acceptance criteria: Flowbite components render (Button, Card, Table, Modal, DataTable, Form, Select, DatePicker, Tabs, Checkbox, FileInput); light/dark toggle persists reload; SSR no flash; `pnpm run check` passes
  QA scenarios: Toggle theme → persist → reload → correct theme; Evidence: `attemptDir/task-2-buku-induk-siswa.log`
  Commit: Y | feat(ui): install Flowbite Svelte, configure Tailwind v4, implement light/dark theme toggle

- [ ] 3. **Supabase SSR Client + Environment Config**
  What to do / Must NOT do: `pnpm add @supabase/supabase-js @supabase/ssr` → `src/hooks.server.ts` createServerClient → `src/routes/+layout.server.ts` load session → `src/routes/+layout.ts` createLoadClient → `src/routes/+layout.svelte` auth state listener → `src/lib/supabase.ts` typed client → `.env` (PUBLIC_SUPABASE_URL, PUBLIC_SUPABASE_ANON_KEY) → `src/env.ts` defineEnvVars. NO Storage client init.
  Closes: GAP-1
  Parallelization: Wave 1 | Blocked by: 1 | Blocks: 4-34
  References: Supabase SvelteKit SSR guide; @supabase/ssr package docs
  Acceptance criteria: `event.locals.supabase` available in hooks; session persists navigation; `pnpm run check` passes; login page accessible
  QA scenarios: Login → session cookie set → navigate protected route → session valid; Evidence: `attemptDir/task-3-buku-induk-siswa.log`
  Commit: Y | feat(auth): setup Supabase SSR client with cookie-based sessions

### Wave 2: Database & Auth Foundation

- [ ] 4. **Database Schema Migration (14 tables, PK identity, FK indexes, CHECK, RLS, triggers)**
  What to do / Must NOT do: Create Supabase migration SQL files for 14 tables: `siswa`, `orang_tua`, `kelas`, `kamar`, `tahun_ajaran`, `users`, `roles`, `permissions`, `role_permissions`, `audit_log`, `status_siswa`, `dokumen_siswa`, `wali_kelas_assignment`, `wali_kamar_assignment`. All PK: `bigint generated always as identity`. FK indexes explicit. CHECK constraints: `status_kebutuhan` (yatim/piatu/yatim-piatu/dhuafa/umum), `kamar.tipe` (ikhwan/akhwat), `status_siswa.status` (aktif/sakit/pulang/izin). RLS policies per role. Trigger function `audit_trigger()` on ALL mutating tables. Auth events via Supabase Auth hooks. NO Supabase Storage buckets.
  Closes: GAP-2
  Parallelization: Wave 2 | Blocked by: 3 | Blocks: 5-34
  References: supabase-postgres-best-practices: schema-primary-keys, schema-foreign-key-indexes, schema-constraints, security-rls-basics; PostgreSQL trigger docs
  Acceptance criteria: `supabase db push` applies clean; `supabase gen types typescript --local > src/lib/database.types.ts` generates types; all tables queryable; RLS blocks unauthorized; triggers fire on INSERT/UPDATE/DELETE
  QA scenarios: Insert siswa → audit_log row created; Update siswa → old_data/new_data captured; Delete → audit_log action=DELETE; RLS: wali_kelas only sees own kelas; Evidence: `attemptDir/task-4-buku-induk-siswa.sql`
  Commit: Y | feat(db): create 14-table schema with PK identity, FK indexes, CHECK constraints, RLS, audit triggers

- [ ] 5. **Auth System: Login Page + Protected Routes + 5 Roles + Profile/Password**
  What to do / Must NOT do: `/login` page (Flowbite Form + Zod) → `src/hooks.server.ts` handle hook guard routes by role → `/auth/callback` for email confirm → `/profile` page (update name, avatar) → `/change-password` page → role stored in `users.role_id` FK to `roles`. NO OAuth providers.
  Closes: GAP-1
  Parallelization: Wave 2 | Blocked by: 3,4 | Blocks: 6-34
  References: Supabase SvelteKit auth tutorial; Flowbite Form + Zod resolver pattern
  Acceptance criteria: Login redirects to `/dashboard`; invalid creds shows error; role guard blocks unauthorized routes; password change works; profile update works; `pnpm run test` passes auth unit tests
  QA scenarios: Login as superadmin → access /settings; Login as wali_kamar → blocked from /settings; Evidence: `attemptDir/task-5-buku-induk-siswa.log`
  Commit: Y | feat(auth): implement login, role guards, profile, password change for 5 roles

- [ ] 6. **Mock Service Layer Contract (TypeScript interfaces for all domain services)**
  What to do / Must NOT do: Define `src/lib/services/` interfaces: `SiswaService`, `KelasService`, `KamarService`, `TahunAjaranService`, `StatusSiswaService`, `DokumenService`, `AuditLogService`, `AssignmentService`, `ProposalService`, `RbacService`, `GoogleDriveService`. Each with methods matching real backend. Return dummy data (TypeScript constants). NO implementation — only interfaces + mock implementations for UI prototype phase.
  Closes: GAP-4
  Parallelization: Wave 2 | Blocked by: 4 | Blocks: 7-34
  References: Contract-first development; Svelte 5 runes for service state
  Acceptance criteria: All interfaces defined; mock implementations return typed dummy data; UI components import from service interfaces; `pnpm run check` passes
  QA scenarios: Import SiswaService in component → type checks; Evidence: `attemptDir/task-6-buku-induk-siswa.ts`
  Commit: Y | feat(services): define mock service layer contracts for UI-first development

- [ ] 7. **RBAC Middleware + Permission Matrix Seed**
  What to do / Must NOT do: `src/lib/rbac.ts` — `can(user, resource, action)` check via `role_permissions` → `+layout.server.ts` inject permissions → `PermissionGate.svelte` component for UI gating → seed SQL: permissions (siswa:read/create/update/delete/export, kelas:read/create/update/delete, kamar:..., users:..., audit_log:read/export, settings:read/update, assignment:propose/approve, proposal:create/approve, dokumen:read/create/delete) → role_permissions per seed matrix (superadmin=all, staff_tu=siswa/kelas/kamar/dokumen CRUD+export+assignment:approve+proposal:approve, wali_kelas=siswa:read+status_siswa:create/update+assignment:propose+proposal:create, wali_kamar=siswa:read+status_siswa:CRUD+assignment:propose+proposal:create, asatidz=siswa:read). NO hardcoded role checks in components.
  Closes: GAP-10 (partial)
  Parallelization: Wave 2 | Blocked by: 5 | Blocks: 8-34
  References: Dynamic RBAC pattern; Supabase RLS with JOIN roles/permissions
  Acceptance criteria: `can(user, 'siswa', 'create')` returns true for staff_tu, false for wali_kelas; PermissionGate hides/shows UI correctly; seed SQL inserts correct matrix
  QA scenarios: Login wali_kelas → cannot see "Tambah Siswa" button; Login staff_tu → can see; Evidence: `attemptDir/task-7-buku-induk-siswa.log`
  Commit: Y | feat(rbac): implement permission matrix, middleware, seed data, PermissionGate component

### Wave 3: Dashboard (UI-First)

- [ ] 8. **Dashboard UI Prototype (Dummy Data)**
  What to do / Must NOT do: `/dashboard` page with Flowbite: stats cards (total aktif, per kelas, per kamar, per status kebutuhan, per angkatan), recent activity table (dummy audit_log), quick actions. Use mock `DashboardService` returning dummy data. Responsive grid. Light/dark theme. NO real data fetching.
  Closes: GAP-5
  Parallelization: Wave 3 | Blocked by: 5,6,7 | Blocks: 9
  References: Flowbite Card, Table, Badge components; mock DashboardService from task 6
  Acceptance criteria: Page renders all cards/tables with dummy data; responsive on mobile (stack cards); theme toggle works; no console errors
  QA scenarios: Visual review — user approves layout/colors/spacing; Evidence: `attemptDir/task-8-buku-induk-siswa.png` (screenshot)
  Commit: Y | feat(dashboard): build UI prototype with dummy data

- [ ] 9. **Dashboard Backend Integration**
  What to do / Must NOT do: Replace mock `DashboardService` with real Supabase queries: `siswa` count by status/kelas/kamar/tahun_ajaran, `audit_log` recent 10, `status_siswa` summary. Server load in `+page.server.ts`. Cache 30s. Keep UI identical.
  Closes: GAP-6
  Parallelization: Wave 3 | Blocked by: 8,4 | Blocks: —
  References: Supabase query patterns; SvelteKit load functions
  Acceptance criteria: Real data matches dummy structure; stats update on data change; loading skeletons; error boundary shows toast
  QA scenarios: Add siswa → dashboard total updates; Evidence: `attemptDir/task-9-buku-induk-siswa.log`
  Commit: Y | feat(dashboard): connect to real Supabase data

### Wave 4: Siswa Management (UI-First)

- [ ] 10. **Siswa CRUD UI Prototype (Dummy Data)**
  What to do / Must NOT do: `/siswa` list page: Flowbite DataTable (search, filter kelas/kamar/angkatan/status_kebutuhan/status_kesehatan, cursor pagination, column visibility), "Tambah" button → Modal Form (Flowbite Form + Zod schema: nested orang_tua, kamar select, dokumen tabs), row actions: Detail (tabs: Profile, Dokumen, Activity Timeline), Edit, Delete (confirm). `/siswa/[id]` detail page. All dummy data via mock `SiswaService`. Responsive: table → card stack on mobile.
  Closes: GAP-7
  Parallelization: Wave 4 | Blocked by: 5,6,7 | Blocks: 11
  References: Flowbite DataTable, Dialog, Form, Select, DatePicker, Tabs, FileInput; TanStack Table v8 patterns
  Acceptance criteria: All CRUD UI flows work with dummy data; form validation shows errors; modal opens/closes; tabs switch; responsive mobile card view; theme consistent
  QA scenarios: Visual review — user approves form layout, table columns, modal UX; Evidence: `attemptDir/task-10-buku-induk-siswa.png`
  Commit: Y | feat(siswa): build CRUD UI prototype with dummy data

- [ ] 11. **Siswa CRUD Backend Integration + Google Drive Dokumen**
  What to do / Must NOT do: Real `SiswaService` with Supabase CRUD (Zod validated server actions). `dokumen_siswa` CRUD → Google Drive API (service account from settings): upload (resumable, max 5MB, resize images), delete, list, thumbnail. Per-tahun-ajaran folders. `foto_url` → Drive `webViewLink`. Excel import: drag-drop .xlsx → `xlsx` parse → Zod validate array → batch `upsert` on `nisn` → error report download. Excel export: filtered query → `xlsx` workbook. PDF export: filtered query → `pdf-lib` A4 portrait. NO Supabase Storage.
  Closes: GAP-8
  Parallelization: Wave 4 | Blocked by: 10,4,18 | Blocks: —
  References: Google Drive API v3 resumable upload; xlsx SheetJS; pdf-lib; Zod array validation; data-batch-inserts
  Acceptance criteria: Create siswa with dokumen → files in Drive correct folder; Edit updates Drive; Delete removes Drive file; Import 100 rows → all inserted + error report; Export Excel/PDF matches filter; `pnpm run test` passes
  QA scenarios: Upload KK → appears in Drive `Buku Induk Siswa/2024-2025/KK/`; Import Excel with invalid NIK → error report downloadable; Evidence: `attemptDir/task-11-buku-induk-siswa.log`
  Commit: Y | feat(siswa): backend CRUD + Google Drive dokumen + Excel/PDF import/export

### Wave 5: Kelas Management (UI-First)

- [ ] 12. **Kelas Management UI Prototype (Dummy Data)**
  What to do / Must NOT do: `/kelas` list: DataTable (search, filter tingkat/tahun_ajaran, pagination), "Tambah" modal (nama, tingkat 7/8/9, tahun_ajaran select, wali_kelas select users with role wali_kelas), row actions: Edit, Delete, "Kelola Assignment" → checklist siswa unassigned → submit proposal. Dummy data via mock `KelasService`.
  Closes: GAP-9
  Parallelization: Wave 4 | Blocked by: 5,6,7 | Blocks: 13
  References: Flowbite DataTable, Dialog, Form, Select, Checkbox; mock KelasService
  Acceptance criteria: All UI flows work dummy; checklist multi-select works; proposal submit modal; responsive
  QA scenarios: Visual review — user approves assignment workflow UI; Evidence: `attemptDir/task-12-buku-induk-siswa.png`
  Commit: Y | feat(kelas): build management UI prototype with assignment checklist

- [ ] 13. **Kelas Management Backend Integration + Assignment Workflow**
  What to do / Must NOT do: Real `KelasService` CRUD. `wali_kelas_assignment` table: proposal (user_id, kelas_id, proposed_by, status: pending/approved/rejected, approved_by, approved_at) → TU dashboard `/assignments` shows pending → approve/reject → on approve: insert assignment, notify. Email/notification out of scope — UI toast only.
  Closes: GAP-10
  Parallelization: Wave 5 | Blocked by: 12,4 | Blocks: 26
  References: Proposal pattern; Supabase RLS for assignments
  Acceptance criteria: Wali kelas proposes assignment → TU sees pending → approve → assignment created → wali sees assigned kelas; audit_log captures
  QA scenarios: Propose 5 siswa to kelas 7A → TU approve → all 5 assigned; Evidence: `attemptDir/task-13-buku-induk-siswa.log`
  Commit: Y | feat(kelas): backend CRUD + assignment proposal workflow

### Wave 6: Kamar Management (UI-First)

- [ ] 14. **Kamar Management UI Prototype (Dummy Data)**
  What to do / Must NOT do: `/kamar` list: DataTable (search, filter tipe/tahun_ajaran, pagination), "Tambah" modal (nomor 1-8, tipe ikhwan/akhwat, kapasitas, tahun_ajaran, wali_kamar select), row actions: Edit, Delete, "Kelola Assignment" → checklist siswa (filter by gender match) → submit proposal. Dummy data via mock `KamarService`.
  Closes: GAP-11
  Parallelization: Wave 4 | Blocked by: 5,6,7 | Blocks: 15
  References: Flowbite components; gender-separated room logic
  Acceptance criteria: UI enforces gender match in checklist (ikhwan only sees putra, akhwat only putri); proposal flow works dummy
  QA scenarios: Visual review — user approves gender-filtered checklist; Evidence: `attemptDir/task-14-buku-induk-siswa.png`
  Commit: Y | feat(kamar): build management UI prototype with gender-filtered assignment

- [ ] 15. **Kamar Management Backend Integration + Assignment Workflow**
  What to do / Must NOT do: Real `KamarService` CRUD. `wali_kamar_assignment` proposal flow identical to kelas. Capacity check on assign. TU approval same dashboard.
  Closes: GAP-12
  Parallelization: Wave 5 | Blocked by: 14,4 | Blocks: 26
  References: Kamar capacity logic; assignment proposal pattern
  Acceptance criteria: Capacity exceeded → proposal rejected; gender mismatch → blocked; audit_log captures
  QA scenarios: Assign 10 siswa to kapasitas 8 → rejects 2; Evidence: `attemptDir/task-15-buku-induk-siswa.log`
  Commit: Y | feat(kamar): backend CRUD + assignment proposal with capacity/gender validation

### Wave 7: Tahun Ajaran (UI-First)

- [ ] 16. **Tahun Ajaran UI Prototype (Dummy Data)**
  What to do / Must NOT do: `/tahun-ajaran` list: CRUD (nama, mulai, selesai, aktif radio — only one active), badge active. Dummy data via mock `TahunAjaranService`.
  Closes: GAP-13
  Parallelization: Wave 4 | Blocked by: 5,6,7 | Blocks: 17, 20
  References: Simple CRUD pattern; single active constraint
  Acceptance criteria: Only one `aktif` at a time; date picker works; responsive
  QA scenarios: Visual review; Evidence: `attemptDir/task-16-buku-induk-siswa.png`
  Commit: Y | feat(tahun-ajaran): build UI prototype

- [ ] 17. **Tahun Ajaran Backend Integration**
  What to do / Must NOT do: Real `TahunAjaranService` CRUD. `aktif` constraint via partial unique index. On change active → drives Google Drive folder structure for new tahun_ajaran (pre-create folders).
  Closes: GAP-14
  Parallelization: Wave 5 | Blocked by: 16,4 | Blocks: 20
  References: Partial unique index; Drive folder creation API
  Acceptance criteria: Switch active → new tahun_ajaran folders created in Drive; siswa/kelas/kamar filter by active tahun_ajaran default
  QA scenarios: Create 2025-2026 → set active → Drive folders exist; Evidence: `attemptDir/task-17-buku-induk-siswa.log`
  Commit: Y | feat(tahun-ajaran): backend CRUD + Drive folder provisioning

### Wave 8: Status Kesehatan (UI-First + Realtime)

- [ ] 18. **Status Kesehatan UI Prototype (Dummy Data)**
  What to do / Must NOT do: `/status-kesehatan` page (or tab in siswa detail): List status_siswa for assigned kamar/kelas (filter by wali), "Tambah" modal: status select (sakit/pulang/izin/aktif), deskripsi textarea, tanggal (date), tanggal_mulai/tanggal_selesai (for pulang), siswa select (checkbox multi for batch log). Real-time indicator (green dot). Dummy data via mock `StatusSiswaService` + mock Realtime subscription.
  Closes: GAP-15
  Parallelization: Wave 4 | Blocked by: 5,6,7 | Blocks: 19
  References: Flowbite Form, DatePicker, Checkbox; Supabase Realtime patterns
  Acceptance criteria: Form validates (pulang requires tanggal_mulai + tanggal_selesai); batch log multiple siswa; realtime dot pulses dummy
  QA scenarios: Visual review — user approves form fields for each status type; Evidence: `attemptDir/task-18-buku-induk-siswa.png`
  Commit: Y | feat(kesehatan): build status tracking UI prototype with batch logging

- [ ] 19. **Status Kesehatan Backend Integration + Supabase Realtime**
  What to do / Must NOT do: Real `StatusSiswaService` CRUD. Supabase Realtime subscription on `status_siswa` table filtered by `kamar_id`/`kelas_id` → live updates to UI (new entry appears without refresh). RLS: wali_kamar sees own kamar, wali_kelas sees own kelas, superadmin all. History retained (no auto-delete).
  Closes: GAP-16
  Parallelization: Wave 5 | Blocked by: 18,4 | Blocks: —
  References: Supabase Realtime channel filter; RLS with JOIN
  Acceptance criteria: Wali kamar logs sakit → instantly appears on other wali_kamar same kamar screen; Realtime reconnects on network change; `pnpm run test` passes
  QA scenarios: Open two browsers wali kamar same kamar → log in one → appears in other <1s; Evidence: `attemptDir/task-19-buku-induk-siswa.log`
  Commit: Y | feat(kesehatan): backend CRUD + Supabase Realtime live updates

### Wave 9: Dokumen/Google Drive (UI-First)

- [ ] 20. **Dokumen Siswa UI Prototype (Dummy Data)**
  What to do / Must NOT do: Siswa detail → Dokumen tab: 6 card slots (Foto, KK, Akta, Ijazah, SKK, SKTM) — each shows thumbnail (Drive `thumbnailLink`) or upload button. Click → FileInput (Flowbite) → preview → upload progress. "Lihat di Drive" link. Dummy data via mock `DokumenService` + mock `GoogleDriveService`.
  Closes: GAP-17
  Parallelization: Wave 4 | Blocked by: 5,6,7,16 | Blocks: 21
  References: Flowbite FileInput, Card, Progress; Google Drive thumbnailLink
  Acceptance criteria: 6 slots render; upload shows progress; thumbnail displays; "Lihat di Drive" opens correct URL; responsive grid
  QA scenarios: Visual review — user approves dokumen card layout; Evidence: `attemptDir/task-20-buku-induk-siswa.png`
  Commit: Y | feat(dokumen): build dokumen management UI prototype

- [ ] 21. **Dokumen/Google Drive Backend Integration**
  What to do / Must NOT do: Real `GoogleDriveService`: service account JWT (from settings), domain-wide delegation, resumable upload (max 5MB), create per-tahun-ajaran folders if not exist (`Buku Induk Siswa/{tahun_ajaran}/{Foto,KK,Akta,Ijazah,SKK,SKTM}/`), store metadata in `dokumen_siswa` (drive_file_id, webViewLink, thumbnailLink, jenis, tahun_ajaran_id). Delete → Drive `files.delete`. Settings page: superadmin paste service account JSON + root folder ID → validate → save encrypted.
  Closes: GAP-18
  Parallelization: Wave 5 | Blocked by: 20,4,17,24 | Blocks: —
  References: Google Drive API v3 service account; resumable upload; folder creation; Drive thumbnailLink
  Acceptance criteria: Upload KK → file in correct Drive folder; metadata in dokumen_siswa; thumbnail loads; delete removes Drive file; settings save/validate works
  QA scenarios: Upload 5MB PDF Ijazah → success; Upload 6MB → rejected; Evidence: `attemptDir/task-21-buku-induk-siswa.log`
  Commit: Y | feat(dokumen): Google Drive integration with service account, per-tahun-ajaran folders

### Wave 10: Audit Log (UI-First)

- [ ] 22. **Audit Log UI Prototype (Dummy Data)**
  What to do / Must NOT do: `/audit-log` page: DataTable (search, filter user/table/action/date range, cursor pagination), columns: timestamp, user, table, record_id, action, diff preview (old→new), IP. Export Excel button. Dummy data via mock `AuditLogService`. Superadmin only (RBAC gate).
  Closes: GAP-19
  Parallelization: Wave 4 | Blocked by: 5,6,7 | Blocks: 23
  References: Flowbite DataTable, DateRangePicker, Export button; audit_log JSONB diff display
  Acceptance criteria: Filters work dummy; diff shows old/new values; export button triggers download; pagination cursor
  QA scenarios: Visual review — user approves diff display format; Evidence: `attemptDir/task-22-buku-induk-siswa.png`
  Commit: Y | feat(audit): build audit log UI prototype with diff viewer

- [ ] 23. **Audit Log Backend Integration (Triggers + All Tables + Auth Events)**
  What to do / Must NOT do: PostgreSQL trigger function `audit_trigger()` on ALL mutating tables (siswa, orang_tua, kelas, kamar, tahun_ajaran, users, status_siswa, dokumen_siswa, wali_kelas_assignment, wali_kamar_assignment) → inserts to `audit_log` (user_id, table_name, record_id, action, old_data JSONB, new_data JSONB, ip, user_agent). Supabase Auth hooks for login/password_change/profile_update → audit_log. Real `AuditLogService` query with filters + cursor pagination. Retention: cron job delete >1yr (configurable).
  Closes: GAP-20
  Parallelization: Wave 5 | Blocked by: 22,4 | Blocks: —
  References: PostgreSQL triggers; Supabase pg_cron; JSONB diff
  Acceptance criteria: Any INSERT/UPDATE/DELETE on tracked tables → audit_log row; Auth events logged; Query filters work; Export Excel matches filter; Cron deletes old logs
  QA scenarios: Update siswa.nama → audit_log shows old/new; Login → audit_log action=LOGIN; Evidence: `attemptDir/task-23-buku-induk-siswa.log`
  Commit: Y | feat(audit): implement DB triggers on all tables + auth events + retention cron

### Wave 11: RBAC & Settings (UI-First)

- [ ] 24. **RBAC & Settings UI Prototype (Dummy Data)**
  What to do / Must NOT do: `/settings` (superadmin only): Tabs: Users (CRUD table), Permissions (matrix grid: resources × actions × roles, checkbox toggle), System (school name, logo upload → Drive, default tahun_ajaran, Google Drive config: service account JSON textarea, root folder ID input, "Test Connection" button). Dummy data via mock `RbacService` + `SettingsService`.
  Closes: GAP-21
  Parallelization: Wave 4 | Blocked by: 5,6,7 | Blocks: 25
  References: Flowbite Tabs, Table, Checkbox grid, Textarea; Permission matrix UI pattern
  Acceptance criteria: Matrix grid toggles permissions; Users CRUD works; Drive config test button dummy; responsive
  QA scenarios: Visual review — user approves matrix UX; Evidence: `attemptDir/task-24-buku-induk-siswa.png`
  Commit: Y | feat(settings): build RBAC matrix + system settings UI prototype

- [ ] 25. **RBAC & Settings Backend Integration**
  What to do / Must NOT do: Real `RbacService`: CRUD roles/permissions/role_permissions, `can()` middleware. `SettingsService`: upsert `system_settings` key-value (school_name, logo_drive_file_id, default_tahun_ajaran_id, gdrive_service_account_json_encrypted, gdrive_root_folder_id). Encrypt service account JSON (libsodium). Drive config test → API call `drive.about.get`. RLS on settings table.
  Closes: GAP-22
  Parallelization: Wave 5 | Blocked by: 24,4 | Blocks: 26
  References: Dynamic RBAC; libsodium encryption; Google Drive API test
  Acceptance criteria: Toggle permission → immediate effect (no reload); Drive config test → success/fail toast; Encrypted JSON not readable in DB
  QA scenarios: Revoke wali_kelas `siswa:read` → UI hides siswa menu; Evidence: `attemptDir/task-25-buku-induk-siswa.log`
  Commit: Y | feat(rbac): backend permission matrix + encrypted Drive config + settings persistence

### Wave 12: Assignment Workflow (UI-First)

- [ ] 26. **Assignment Workflow UI Prototype (Dummy Data)**
  What to do / Must NOT do: `/assignments` page: TU view — tabs: "Pending Kelas", "Pending Kamar", "History". Each: DataTable (proposal info, siswa count, proposed_by, actions: View Detail → modal with siswa list + Approve/Reject buttons). Wali view — "Ajukan Assignment" button → checklist siswa (filter by kamar/kelas/angkatan, gender match for kamar) → submit → "Proposals Saya" tab shows status. Dummy data via mock `AssignmentService`.
  Closes: GAP-23
  Parallelization: Wave 4 | Blocked by: 5,6,7,12,14 | Blocks: 27
  References: Flowbite DataTable, Dialog, Checkbox, Badge; proposal status flow
  Acceptance criteria: TU sees pending proposals; wali sees own proposals; detail modal shows siswa list; approve/reject buttons; responsive
  QA scenarios: Visual review — user approves both TU and wali views; Evidence: `attemptDir/task-26-buku-induk-siswa.png`
  Commit: Y | feat(assignment): build assignment proposal UI for TU and wali views

- [ ] 27. **Assignment Workflow Backend Integration**
  What to do / Must NOT do: Real `AssignmentService`: `proposeAssignment(userId, targetIds, type: 'kelas'|'kamar', proposedBy)` → inserts proposal rows (status=pending). `approveAssignment(proposalId, approvedBy)` → transaction: insert assignment records, update proposal status, audit_log. `rejectAssignment(proposalId, approvedBy, reason)` → update proposal. TU dashboard queries pending. Notifications: UI toast only.
  Closes: GAP-24
  Parallelization: Wave 5 | Blocked by: 26,13,15 | Blocks: —
  References: Transaction pattern; proposal status machine
  Acceptance criteria: Wali proposes 10 siswa → TU approves → 10 assignment rows created; reject → proposal status=rejected; audit_log captures both
  QA scenarios: Propose duplicate siswa → handled gracefully; Evidence: `attemptDir/task-27-buku-induk-siswa.log`
  Commit: Y | feat(assignment): backend proposal workflow with TU approval

### Wave 13: Data Change Proposal (UI-First)

- [ ] 28. **Data Change Proposal UI Prototype (Dummy Data)**
  What to do / Must NOT do: Siswa detail page → "Ajukan Perubahan" button → modal: field select (multi), current value, proposed value, alasan. Submit → "Proposals Saya" tab (status badge). TU view: `/proposals` DataTable (filter status), row action: View Diff (side-by-side current vs proposed), Approve/Reject. Dummy data via mock `ProposalService`.
  Closes: GAP-25
  Parallelization: Wave 4 | Blocked by: 5,6,7,10 | Blocks: 29
  References: Flowbite Modal, Form, Diff view pattern; proposal status
  Acceptance criteria: Field multi-select works; diff view shows changes clearly; TU approve/reject flow dummy
  QA scenarios: Visual review — user approves diff UI; Evidence: `attemptDir/task-28-buku-induk-siswa.png`
  Commit: Y | feat(proposal): build data change proposal UI with diff viewer

- [ ] 29. **Data Change Proposal Backend Integration**
  What to do / Must NOT do: Real `ProposalService`: `createProposal(siswaId, changes: Record<string, {from, to}>, reason, proposedBy)` → inserts `data_change_proposal` (id, siswa_id, changes JSONB, reason, proposed_by, status, approved_by, approved_at, applied_at). TU approve → transaction: apply changes to `siswa` (and related), update proposal status, audit_log captures. Reject → status=rejected.
  Closes: GAP-26
  Parallelization: Wave 5 | Blocked by: 28,11 | Blocks: —
  References: JSONB changes; transaction apply; audit_log integration
  Acceptance criteria: Propose nama+alamat → TU approves → siswa updated + audit_log; Reject → no changes; `pnpm run test` passes
  QA scenarios: Propose invalid NIK → Zod rejects; Evidence: `attemptDir/task-29-buku-induk-siswa.log`
  Commit: Y | feat(proposal): backend data change proposal with diff apply + audit

### Wave 14: Excel/PDF Export (Shared)

- [ ] 30. **Excel Import/Export Implementation**
  What to do / Must NOT do: Server actions: `importSiswa(formData)` → `xlsx` parse → Zod array validate → batch `upsert` on `nisn` (onConflict) → return {success: n, errors: []} → client downloads error report .xlsx. `exportSiswa(filters)` → query → `xlsx` workbook → Indonesian headers → download. Shared utility `src/lib/excel.ts`.
  Closes: GAP-27 (partial)
  Parallelization: Wave 5 | Blocked by: 4,11 | Blocks: —
  References: xlsx SheetJS streaming; Zod array; data-batch-inserts
  Acceptance criteria: Import 500 rows <10s; error report has row numbers; Export matches current filter; `pnpm run test` passes
  QA scenarios: Import with 10 invalid rows → 490 inserted, 10 in error report; Evidence: `attemptDir/task-30-buku-induk-siswa.log`
  Commit: Y | feat(excel): implement import/export with Zod validation + error reporting

- [ ] 31. **PDF Export Implementation**
  What to do / Must NOT do: Server action `exportSiswaPdf(filters)` → query filtered → `pdf-lib` generate A4 portrait: header (school name/logo), table (columns: NISN, Nama, Kelas, Kamar, Status Kebutuhan, Status Kesehatan), footer (page X of Y, date). Font: Noto Sans (supports Indonesian). Stream response.
  Closes: GAP-27 (partial)
  Parallelization: Wave 5 | Blocked by: 4,11 | Blocks: —
  References: pdf-lib table generation; Unicode font embedding
  Acceptance criteria: PDF opens in browser; 100 rows → 3 pages; headers/footers correct; Indonesian chars render
  QA scenarios: Export filtered 50 siswa → PDF correct; Evidence: `attemptDir/task-31-buku-induk-siswa.pdf`
  Commit: Y | feat(pdf): implement A4 portrait PDF export with school branding

### Wave 15: Polish, Testing, Deploy

- [ ] 32. **Responsive Polish + Accessibility + Empty/Error States**
  What to do / Must NOT do: All pages: mobile breakpoint (<640px) table→card, modal→fullscreen, form stacked. WCAG AA: color contrast, focus visible, ARIA labels, keyboard nav. Empty states (illustration + action). Error boundaries + toast (Sonner). Loading skeletons. NO new features.
  Closes: GAP-28 (partial)
  Parallelization: Wave 6 | Blocked by: 8-31 | Blocks: 33
  References: Flowbite responsive; WCAG AA checklist; Sonner toast
  Acceptance criteria: Lighthouse accessibility >90; mobile viewport no horizontal scroll; keyboard tab order logical; error toast on network fail
  QA scenarios: Mobile Chrome DevTools → all pages usable; Evidence: `attemptDir/task-32-buku-induk-siswa.log`
  Commit: Y | feat(polish): responsive, accessibility, empty/error states, loading skeletons

- [ ] 33. **Playwright E2E Tests (Critical Flows)**
  What to do / Must NOT do: `pnpm add -D @playwright/test` → tests: login (all 5 roles), dashboard stats, siswa CRUD, kelas/kamar assignment propose→approve, status kesehatan log+realtime, dokumen upload, audit log filter+export, settings RBAC toggle. CI runs on Chromium/Firefox/WebKit. NO visual regression.
  Closes: GAP-28 (partial)
  Parallelization: Wave 6 | Blocked by: 32 | Blocks: 34
  References: Playwright SvelteKit patterns; test data isolation
  Acceptance criteria: All 10 tests pass in CI; test data cleaned up; runs <5min
  QA scenarios: CI pipeline green; Evidence: `attemptDir/task-33-buku-induk-siswa.xml`
  Commit: Y | test(e2e): add Playwright tests for 10 critical user flows

- [ ] 34. **Cloudflare Pages Deploy Config + GitHub Actions CI/CD**
  What to do / Must NOT do: `pnpm add -D @sveltejs/adapter-cloudflare wrangler` → `vite.config.ts` adapter-cloudflare(config) → `wrangler.jsonc` (name, main, assets binding, compatibility_date, vars) → `.github/workflows/ci.yml` (lint, check, test, build, deploy preview on PR, deploy production on main) → Cloudflare Pages project linked → env vars in dashboard (PUBLIC_SUPABASE_URL, PUBLIC_SUPABASE_ANON_KEY, GDRIVE_SERVICE_ACCOUNT_JSON_ENCRYPTED, GDRIVE_ROOT_FOLDER_ID, ENCRYPTION_KEY).
  Closes: GAP-28 (partial)
  Parallelization: Wave 6 | Blocked by: 1,34 | Blocks: —
  References: SvelteKit adapter-cloudflare; wrangler.jsonc; GitHub Actions deploy
  Acceptance criteria: Push to main → preview URL works; Push to main → production deploy; Env vars accessible; `wrangler pages deploy` works locally
  QA scenarios: PR deploy → preview URL accessible; Production deploy → custom domain works; Evidence: `attemptDir/task-34-buku-induk-siswa.yml`
  Commit: Y | chore(deploy): configure Cloudflare Pages + GitHub Actions CI/CD

## Final verification wave

> Runs in parallel after ALL todos. ALL must APPROVE. Surface results and wait for the user's explicit okay before declaring complete.

- [ ] F1. **Plan compliance audit** — Verify every todo maps to GAP/IS, no scope creep, all Must NOT have respected
- [ ] F2. **Code quality review** — `pnpm run check` clean, `pnpm run lint` clean, TypeScript strict, no `any`, proper error handling
- [ ] F3. **Real manual QA** — Playwright report green; manual test: login each role, full CRUD flows, assignment propose→approve, proposal diff, dokumen upload Drive, audit log filters, theme toggle, mobile responsive
- [ ] F4. **Ideal-state fidelity** — Each IS row verified: Superadmin manages permissions/UI; TU approves assignments/proposals; Wali kelas/kamar checklist-assign + propose changes; Asatidz read-only; Pimpinan dashboard stats; Developer: `pnpm run check` + `pnpm run test` + `pnpm run build` all pass

## Commit strategy

- Conventional commits: `feat(scope): summary`, `fix(scope): summary`, `chore(scope): summary`, `test(scope): summary`
- One commit per todo (or logical group within wave)
- Branch per wave: `wave-1-foundation`, `wave-2-db-auth`, etc.
- PR per wave with description linking todos
- Squash merge to main after review

## Success criteria

> One row per IS row. The plan is complete only when every IS row has a delivering todo and a proving QA scenario; F4 checks the delivered behavior against these rows 1:1, and a shortfall becomes new `- [ ] N.` rows, never a note.

| IS | Delivering todo(s) | Proving QA scenario | Evidence |
| --- | --- | --- | --- |
| IS-1 | 7, 24, 25 | Superadmin creates role, toggles permission, saves Drive config, views audit log | `task-7,24,25` |
| IS-2 | 9, 11, 13, 15, 17, 27, 29, 30, 31 | TU approves assignment/proposal, imports Excel, exports PDF | `task-13,15,17,27,29,30,31` |
| IS-3 | 10, 11, 12, 13, 18, 19, 26, 28, 29 | Wali kelas assigns siswa via checklist, proposes change, TU approves | `task-12,13,26,28,29` |
| IS-4 | 14, 15, 18, 19, 26, 28, 29 | Wali kamar logs sakit/pulang realtime, assigns siswa, proposes change | `task-14,15,18,19,26,28,29` |
| IS-5 | 9, 10, 11 | Asatidz views assigned kelas, exports list | `task-9,10,11` |
| IS-6 | 9, 22, 23 | Pimpinan sees dashboard stats + audit summary | `task-9,22,23` |
| IS-7 | 1-34 | `pnpm run check && pnpm run test && pnpm run build` all pass | `task-1,33,34` |