# Miraas Clinic — Architecture Specification

**Version:** 1.0 · **Date:** 2026-09-20 · **Status:** proposed, not deployed

## 1. Constraints and architecture decision

**Hard constraints from the conversation:** the website currently runs on **shared cPanel hosting**, not a VPS. No VPS may be introduced for this phase. The admin must be able to view/add/edit customers, sessions, payments, requests and follow-ups. A **Google Sheet must be editable and visible through the site's admin CRM**, with **two-way updates**. An admin can import an Excel workbook, initially with about 20 representative customers, and later import the clinic's actual dataset. A consultation request should persist on the site and create an in-app admin notification; WhatsApp to a configurable admin number is desired **at zero additional operating cost**, but never at the expense of reliability or by claiming an unverified workaround is production-ready.

**Recommended approach:** static/mobile-first frontend + PHP 8.x JSON API + MySQL/MariaDB on the same cPanel, with cPanel Cron for background jobs. Use the existing site as a content/assets reference; progressively modularize the frontend. MySQL is the **transactional system of record** for authentication, identifiers, notifications, histories, permissions and pending sync jobs. Google Sheets is an **editable, bi-directionally synchronized operational view** for an explicitly whitelisted set of CRM fields. The panel reads the database and includes sync status; manual Google Sheet edits are imported on a schedule. A Sheet is not suitable as the only source for concurrent edits, security-critical authorization or durable audit trails. The user's requirement is honored by allowing edits in either surface and by exposing the live Google Sheet through the admin workflow.

```
PUBLIC BROWSER (mobile/desktop)          ADMIN BROWSER (protected)
  landing | hero | results                 dashboard | customers | sessions
  consultation form | account              payments | requests | import | sync
                  \                             /
                   HTTPS /api/v1/ (PHP controllers)
                         auth + CSRF + validation + RBAC
                                      |
                               MySQL / MariaDB
                     customers | visits | payments | requests
                     notifications | audit | sync_outbox
                                      |
                   cPanel CRON (PHP CLI, if available)
                      /                  |               \
            Google Sheets API       mail fallback    WhatsApp adapter
             read/write delta         + alert        OPTIONAL / gated
                    |
              Private clinic Sheet (5 tabs)
```

**No public spreadsheet credentials, service account JSON, patient exports or real records may be committed to the public GitHub repository.** Keep database and service credentials in environment configuration outside `public_html` where possible; deny web access to config/logs/uploads and verify deployment behavior. The user-supplied logo is not a credential and may be committed only as an approved export.

## 2. Hosting and runtime

- Verify the actual PHP version, PHP extensions (`pdo_mysql`, `curl`, `openssl`, `mbstring`, `fileinfo`, optionally `zip`), MySQL access, SSH/CLI Cron availability, outbound HTTPS to Google, TLS and disk quotas through cPanel before coding production integrations.
- If Composer cannot run on the shared host, install dependencies locally and upload compatible `vendor/` files. No Node.js process is required in production: optional Vite/Anime.js build runs locally/CI and publishes static assets.
- Suggested paths: `/public_html/index.html`, `/public_html/assets/*`, `/public_html/api/v1/index.php`; place `/private/` config, app code, vendor, logs and backups outside the webroot if the host permits, otherwise deny them at the webserver and verify direct URL access fails.
- Create a separate environment for staging (subdomain or non-production cPanel path/database); never test sync/import against actual patient data. Daily encrypted DB backup plus spreadsheet export, restore drill and retention policy. Do not expose phpMyAdmin or raw DB exports publicly.
- The existing `.htaccess` has `ErrorDocument 404 /index.html`: update routing so `/api/*` errors return **real JSON 404 responses**, not HTML 200 from the app fallback; likewise avoid leaking private files through catch-all rewrites.

## 3. Frontend and API boundaries

Use accessible semantic HTML and modular CSS/TypeScript or modern JS. Recommended public routes or view identifiers: `/`, `/services`, `/results`, `/account` (optional), `/admin` (protected). Modal/bottom-sheet views are stateful UI for auth and consultation and should not require full-page reloads. PHP exposes versioned JSON endpoints, returns consistent errors, rate-limits sensitive operations and verifies roles **server-side**. The UI hiding an admin button is not authorization.

Minimum API contract (final response shapes are implementation-defined):

| Verb & path | Auth | Purpose |
|---|---|---|
| `POST /api/v1/consultations` | public, rate-limited | Validate and store lead/request; create notification and delivery job |
| `POST /api/v1/auth/login` / `logout` | public / session | Session-based authentication; regenerate session ID |
| `POST /api/v1/auth/register` | public, optional launch feature | New account; verified identity required before linking historical patient data |
| `GET /api/v1/me` | signed in | Own safe account data only |
| `GET /api/v1/admin/overview` | admin | KPI summary and unread count |
| `GET/POST /api/v1/admin/customers` | admin | Search/list/create; cursor/pagination and validation |
| `GET/PATCH /api/v1/admin/customers/{id}` | admin | Detail/update with optimistic version |
| `GET/POST /api/v1/admin/visits` | admin | Appointments/sessions |
| `GET/POST /api/v1/admin/payments` | privileged admin | Amount, status and references; no card credentials |
| `GET/PATCH /api/v1/admin/consultations` | admin | Request queue and statuses |
| `GET/PATCH /api/v1/admin/notifications` | admin | Unread/read state |
| `POST /api/v1/admin/imports/preview` | admin + CSRF | Validate workbook and show change preview |
| `POST /api/v1/admin/imports/commit` | admin + CSRF | Apply approved import, audit and queue sync |
| `GET /api/v1/admin/sync/status` | admin | Last success/errors/conflicts |
| `POST /api/v1/admin/sync/run` | admin | Authenticated, rate-limited manual sync trigger |

Use an authenticated same-site HTTP-only `Secure`, `SameSite=Lax` session cookie, CSRF protection for mutating routes, password hashes (`password_hash` Argon2id where available) and login throttling. Public consultation uses explicit spam controls (honeypot + IP-aware throttling; optional CAPTCHA if needed). No admin APIs should accept only a client-side role flag.

## 4. Data model and Google Sheet layout

A **customer/CRM record is not automatically a login identity**. Keep `users` (accounts, roles) separate from `customers` (clinic contacts) and add explicit verified `customer_user_links` only after admin-approved or strongly verified association. An unverified phone/email match may suggest a duplicate for staff review, but must never grant access to an existing record.

Core tables:

| Database table | Essential columns and responsibilities |
|---|---|
| `users` | `id`, `email`/verified phone where applicable, `password_hash`, `role`, `status`, timestamps |
| `customers` | immutable UUID `id`, name, normalized phone, optional email, source, notes (restricted), version, timestamps, deleted_at |
| `visits` | UUID, customer_id, scheduled_at, service, session_number, status, staff (optional), follow_up_at |
| `payments` | UUID, customer_id, visit_id optional, amount_minor or fixed decimal, currency (`IRR`/`IRT` explicitly), payment_status, paid_at, reference |
| `consultation_requests` | UUID, customer_id optional, name, phone, desired_service, preferred_time, note, status, created_at |
| `follow_ups` | UUID, customer_id, due_at, reason, status, completed_at, owner optional |
| `notifications` | UUID, admin user/role audience, request_id, type, read_at, created_at |
| `delivery_jobs` | UUID, notification_id, channel, state, attempts, last_error, idempotency_key |
| `sync_outbox` / `sync_state` | entity, id, version, pending action, last fingerprint/ack, retry, last successful run |
| `sync_conflicts` | entity/id/field, DB value, Sheet value, timestamps, resolution and actor |
| `audit_log` | actor, entity/id, changed fields, prior/new value as appropriate, timestamp, source (`panel`, `sheet`, `import`) |
| `import_batches` / `import_rows` | file digest, preview result, actor, row errors, applied/rolled-back state |

**Initial Google spreadsheet — five tabs:**

1. `Customers`: `customer_id`, `first_name`, `last_name`, `mobile`, `email`, `first_visit_date`, `reason_for_visit`, `lead_source`, `status`, `notes_admin`, `updated_at`, `row_version`.
2. `Visits`: `visit_id`, `customer_id`, `visit_date`, `service` (composite/laminate/smile_design), `session_number`, `visit_status`, `next_follow_up`, `updated_at`, `row_version`.
3. `Payments`: `payment_id`, `customer_id`, `visit_id`, `amount`, `currency`, `payment_status`, `paid_at`, `reference`, `updated_at`, `row_version`.
4. `Consultations`: `request_id`, `customer_id`, `name`, `mobile`, `desired_service`, `preferred_time`, `request_status`, `created_at`, `updated_at`, `row_version`.
5. `FollowUps`: `follow_up_id`, `customer_id`, `due_at`, `reason`, `status`, `completed_at`, `updated_at`, `row_version`.

Each sheet has a protected, immutable ID column and protected system columns; valid field values are validated via dropdowns where useful. Admins can edit approved cells, but **cannot create identity by editing IDs, passwords, roles, audit logs or job state**. Restrict the Sheet to clinic-authorized Google accounts, not public link-sharing. Limit patient-related content to operational minimum; detailed health records, identities and credentials should not be mirrored by default. Preserve a canonical date format (`YYYY-MM-DD` or UTC ISO timestamps), consistent phone normalization (e.g. `+98...`), and unambiguous payment currency units. Spreadsheet display may use Persian labels, but machine-readable keys must remain stable.

## 5. Deterministic two-way sync: panel ↔ Google Sheets

**Transport:** Google Sheets API v4 using a least-privilege service account with *Editor access to one designated clinic spreadsheet*. Clinic owner creates/owns the Sheet and shares it with the service account; credentials live only on server. Use Google-supported read and batch write operations. cPanel Cron polls Google Sheets every 1–5 minutes if permitted; actual propagation is **eventual**, not an instant push guarantee. If Cron minimum cadence differs, display the actual interval and `last_sync_at` in the admin UI. Panel changes are first committed to MySQL, then enqueued for Sheet delivery; the panel never waits on Google to accept the clinic’s request.

**Sync process (idempotent and conflict-aware):**

1. Acquire database advisory/lease lock, prevent overlapping runs, snapshot `sync_state`, and load sheet ranges. Match records by immutable UUID, **never row number** (sorts and insertions change positions).
2. Normalize and validate each editable Sheet row; check unique IDs, cell types, date/phone/currency, allowed status transitions and restricted columns. Produce explicit row errors for malformed changes; do not silently truncate or delete data.
3. Compare last acknowledged field fingerprints with current DB and Sheet values. If only Sheet changed, import validated editable fields transactionally into MySQL, audit `source=sheet`, and enqueue any derived admin notifications. If only DB changed, queue a batch Sheet update.
4. If both changed the **same field** since last sync, create a conflict entry and require admin resolution. If fields differ, perform a safe field-level merge; do not use unqualified last-write-wins. Apply the resolution with a new `row_version`.
5. Send batched writes, retry transient failures with bounded exponential backoff. Update acknowledgment/fingerprints **only after confirmed success**; provide `last_error`, failed row count and a retry action. Jobs survive network outages and process restarts.
6. New rows added manually in Sheet get UUIDs server-side after validation. Row removal does not physically delete a customer: flag as a proposed archive or conflict; only an authorized panel action archives after review.

**Example conflict:** staff changes a phone number in the panel while the Sheet editor changes that same phone before sync. Do not overwrite either silently; present old, panel and Sheet values, with a resolution button and audit entry. For <20 demo rows a full scan per tab is acceptable; re-evaluate batch size, quotas and sync method with real volume. Google Sheets quotas and failure modes require backoff; see official usage limits below.

## 6. Excel import (temporary demo → real data)

- Accept `.xlsx` (optionally `.csv` with explicit UTF-8 rules) only through authenticated admin upload; PHP `PhpSpreadsheet` or a comparable maintained parser. Enforce MIME/extension and strict size/row limits; scan and parse in private temp storage. No macros/embedded formula execution or external spreadsheet references.
- Supply **versioned templates** matching the five tab schemas. Demo workbook contains roughly 20 clearly labeled *synthetic* customers, never actual patients disguised as test data. Real workbook is imported into an approved empty/staging environment or via a reviewed migration batch.
- Preview shows counts: valid new, existing matched, changed fields, duplicate candidates, errors, conflicts and potential archives. Require admin confirmation, preserve original upload checksum, batch ID, backup and rollback/recovery plan.
- Upsert via immutable IDs when present. Without IDs, use normalized contact data only to **propose** potential matches; ambiguous names/phone collisions go to manual review. Do not overwrite non-empty database fields with Excel blanks by default. Offer explicit replace/null semantics if approved.
- Transactional commit updates DB, logs changes and queues Sheet writes; repeated import with the same checksum is idempotent or warned as already applied. Display per-row failures and never claim sync succeeded before the Sheet acknowledges it.
- Export to XLSX/CSV must escape spreadsheet formula prefixes (`=`, `+`, `-`, `@`) for user-supplied text; protect exports/temporary files from public access and delete on a defined schedule.

## 7. Consultation → admin notification → WhatsApp

**Reliable required path:** valid public form → `consultation_requests` INSERT → `notifications` INSERT and `delivery_jobs` INSERT within a DB transaction → HTTP success with request ID. Admin dashboard polls securely or uses short, bounded client polling for unread items; no WebSocket or persistent worker is needed on cPanel. Optional email to clinic via existing cPanel mail/SMTP provides a free-ish fallback within hosting quotas, with **minimal non-sensitive payload**.

**WhatsApp truth table:**

| Mode | Fully automatic to admin? | Cost / limitations | Launch decision |
|---|---|---|---|
| `wa.me` click-to-chat | **No**: the visitor/admin must actually press Send | Simple and normally no API charge | Optional contact shortcut; not a substitute for backend alerts |
| Official WhatsApp Business Platform (Cloud API) | Yes, with configured sender, eligibility and approved templates where needed | Pricing, onboarding and payment requirements may apply; no promise of universal free outbound messages | Adapter behind feature flag; enable only after real end-to-end test and cost approval |
| Unofficial OpenWA / WhatsApp Web automation | In principle, when continuously logged in | Requires persistent browser/session; shared cPanel may not support it; account restrictions, downtime and credential risk | **Do not deploy as mandatory path or claim it is free/reliable on current hosting** |

There is no built-in “free WhatsApp bot” on ordinary shared cPanel that guarantees unsolicited server-initiated notifications to the admin. A clinic-controlled sender number and configured recipient number would be separate, **but sender registration is not a free-delivery guarantee**. Do not put customer notes, treatment, finances or full patient details in messages. If a supported zero-cost outbound solution cannot be verified under actual hosting and account conditions, ship **in-panel notification + email**, keep WhatsApp disabled/pending, and document the gap clearly. Do not silently introduce the prohibited VPS.

## 8. Security, privacy and operational readiness

- HTTPS only; strong admin auth and optional 2FA, login/session expiry, scoped permissions for reception vs finance vs super-admin if needed. Admin accounts seeded privately, never checked into code.
- Data minimization, clinic-authorized access, staff training, consent/notice at registration and consultation. Define retention, deletion/archiving, controlled exports and authorization for patient photos. Determine local legal obligations with the clinic; this document is not a compliance certification.
- Validate input server-side, parameterized SQL, output encoding to prevent XSS, CSRF tokens, rate limits, anti-enumeration, file upload controls, strict API JSON responses and security headers. Logs must not store passwords, access tokens or sensitive request bodies.
- Audit panel/Sheet/import changes and notification attempts; display delivery failures separately from request creation. Treat Sheets outage as degraded sync, **not lost CRM data**.
- Daily backups with tested restore; protect DB, service account key and admin sessions against broad file permissions, web exposure and repository commits. Define monitoring for Cron age, failed queue count, storage and backup staleness.

## 9. Open decisions before deployment

1. Confirm actual cPanel runtime, outbound Google API connectivity and Cron interval; choose a viable service-account setup.
2. Confirm Google account that owns the clinic spreadsheet, staff editors and which specific columns can be edited in Sheets.
3. Confirm admin roles and whether public customer accounts are mandatory for first release.
4. Confirm fields, currency unit, service taxonomy, existing customer identifiers and Excel template format with the clinic.
5. Confirm whether *free, automatic* WhatsApp is a hard launch blocker. Under current constraints it is **unconfirmed**; never substitute an unreliable unofficial connector without explicit risk acceptance.

### Technical references (checked 2026-09-20)

- Google Sheets API: https://developers.google.com/workspace/sheets/api/guides/concepts
- Service account access via sharing: https://developers.google.com/workspace/guides/create-credentials
- Sheets quotas, retries and pricing caveat: https://developers.google.com/workspace/sheets/api/limits
- Official Meta WhatsApp Cloud API overview: https://developers.facebook.com/docs/whatsapp/cloud-api/overview
- WhatsApp Business Platform pricing: https://developers.facebook.com/docs/whatsapp/pricing/

Related specifications: [`Design.md`](./Design.md) · [`Implentation-plan.md`](./Implentation-plan.md).
