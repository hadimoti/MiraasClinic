# Miraas Clinic — Implementation Plan

**Version:** 1.0 · **Date:** Sunday 2026-09-20 · **First work session:** Monday 2026-09-21 ("tomorrow" in the planning conversation) · **Status:** execution checklist; no code/deployment changes claimed

> Filename intentionally matches the requested `Implentation-plan.md`. The entire scope covers a public redesign **and** a private CRM/integration system; it must not be represented as a one-day finished production build. Deliver a working, reviewable slice first, then enable sensitive features only after secure tests.

## 0. Scope freeze: what the user actually requested

| Requirement | Priority | Acceptance evidence |
|---|---|---|
| Mobile-first app-like public website | P0 | Responsive walkthrough on real phone + 320–430px emulation |
| Fixed lower-edge SVG nav for Home, Services, Results, Consultation, Account; admin entry for authorized user | P0 | Safe-area, visible active state, all destinations work |
| Compact layout and short media/slider sections | P0 | No full-screen pinned hero, no unexpectedly long horizontal/vertical scroll |
| White + muted/dark purple branding aligned to supplied Miraas logo | P0 | Approved token sheet and responsive screenshots |
| Visual smile-design hero with SVG + Anime.js | P0 | Path draws, design nodes appear, CTA opens form; reduced-motion fallback |
| Entry/scroll/modal microanimations via Anime.js | P0 | Smooth on mobile; no blocked content or scroll hijack |
| Modal/bottom sheet login/registration, blurred/dim background | P1 | Accessible sheet; **real** secure auth before promising accounts |
| Protected admin dashboard, client CRM with add/edit/view | P0 backend | Authorized CRUD, search/filter, audit; unauthorized requests rejected |
| Google Sheet editable ↔ admin CRM editable | P1 | Verified two-way field sync + displayed sync errors and conflicts |
| Excel prototype (~20 synthetic customers) and future real-data import | P1 | Downloadable template, preview, idempotent import, no silent loss |
| Five useful CRM tabs: customers, visits, payments, consultations, follow-ups | P1 | Relational IDs and Sheet columns match accepted schema |
| Consultation form, request queue, in-admin notifications | P0 backend | Request survives refresh and notifier failures |
| WhatsApp automatic alert to configurable admin number at zero added cost | **Feasibility gate** | Only mark complete after tested free, compliant method on **cPanel-only**; otherwise explicitly pending |
| No VPS, no mandatory Node runtime on server | Hard constraint | Deployment contains only supported cPanel/PHP/DB/Cron components |
| Exclusively aesthetic dental content: composite, laminate, smile design | Hard constraint | No skin, fillers, general dentistry claims or unrelated CTAs |

This plan supersedes earlier casual suggestions that an unofficial WhatsApp sender could simply be run for free on shared hosting. The user clarified that WhatsApp is a **notification only**, not a chat or data store. A permanent WhatsApp Web browser process is not an assumed cPanel capability.

## 1. Phase A — audit and prepare (before writing production features)

- [ ] Re-fetch GitHub `master` head and compare with audited commit `757a93391ef1b2446a93a131899b7f9520638f30`; do not assume the repository is unchanged.
- [ ] Back up currently deployed `public_html`, assets, relevant `.htaccess` and cPanel database if one exists. Confirm the real deployment process and obtain staging domain/path.
- [ ] Verify PHP version, extensions, Cron resolution, MySQL, outbound HTTPS to Google, TLS, cPanel resource limits and ability to keep private files outside webroot. **Stop/adjust integration plan** if hosting cannot support required security or outbound connectivity.
- [ ] Inspect current actual UI on 390×844 and desktop; capture visual baseline of hero, slider, result images and contact paths. Validate text/phone/address with the clinic; avoid blindly copying hard-coded values.
- [ ] Obtain approved transparent logo export and actual brand purple; verify whether currently tracked `logo/english-shape.png`, `logo/persian-shape.png`, `logo/extra/SHAPE.png` are current.
- [ ] Obtain consented/approved patient image set; confirm whether the supplied logo/image can be exported without background and whether a hero smile image is approved.
- [ ] Confirm clinic staff roles, Google account/editor list, fields/currency unit, contact numbers, data ownership/retention and what “customer account” should expose.

**Gate A:** hosting capabilities, design sources and staging strategy recorded. No credentials, patient data or real XLSX in a public branch.

## 2. Phase B — public UX foundation (first demonstrable slice)

- [ ] Create an isolated working branch such as `feature/miraas-app-redesign` and keep `master` untouched until reviewed. Separate `styles/`, `scripts/` and `assets/` from the monolithic HTML progressively; preserve deployed static functionality while migrating.
- [ ] Implement responsive tokens from `Design.md`, RTL typography, consistent containers, safe-area layout and subtle SVG icon system.
- [ ] Build fixed mobile bottom nav (Home / Services / Results / Consultation / Account). Integrate keyboard navigation and sensible desktop header. Reserve bottom padding so cards/forms are not obscured.
- [ ] Replace the existing 120–140vh pinned video hero with compact SVG smile design composition. Draw smile path, reveal design points, staged heading and CTA with Anime.js timeline. Use the approved logo, approved copy and optional approved photo.
- [ ] Build lightweight service cards and results carousel; retain suitable existing assets and require photo authorization. Defer or eliminate `BGNEW.webm` from critical load and keep `motion/teeth-new.webm` only if justified by real-device performance.
- [ ] Implement a reusable, accessible bottom-sheet/dialog manager; wire consultation CTA and account entry. Provide blur/dim backdrop plus fallback, focus trap, Escape/back, focus restoration and scroll lock.
- [ ] Add reveal animations with IntersectionObserver/Anime.js or Anime.js `onScroll` where needed, but ensure static/reduced-motion and no-JS fallbacks for critical copy/links.
- [ ] Update SEO title/description and structured content so the site accurately describes **aesthetic** dentistry only.

**Gate B:** approved public prototype on staging; navigation, compact hero, service/result sections and consultation sheet work. Visual review is mandatory before expanding the backend.

### Suggested first-day cut (2026-09-21)

Aim for a **reviewable public UI and backend feasibility proof**, not a promise of completed secure CRM, Sheet sync and WhatsApp. Feasible first-session target: audit + branch, tokens, bottom nav, compact hero prototype, working consultation sheet interface, cPanel capability checklist, and first DB/API scaffold **if environment access is ready**. If time runs out, keep the prototype on staging and leave production unchanged. Defer live patient data and external messaging until the later gates pass.

## 3. Phase C — backend baseline / data persistence

- [ ] Create schema migrations, `.env.example` (never real `.env`), private secrets store, config loader and PHP JSON API router; ensure `/api/*` cannot fall through to index.html on 404/500.
- [ ] Create MySQL tables listed in `Architecture.md`: users, customers, visits, payments, consultations, follow-ups, notifications, delivery jobs, sync/outbox, conflicts, audit and import batches.
- [ ] Define normalizers/validators for Iranian/international contact number, stable UUID, UTC timestamps vs clinic local display, controlled service labels, and explicit IRR/IRT representation.
- [ ] Implement public `POST /consultations` with transactional request + admin notification, spam/rate protection, duplicate-submit idempotency and useful errors. Return server-issued request ID.
- [ ] Implement protected admin login, role checks and CSRF protections. Seed admin privately; do not publish demo admin credentials. Keep patient `customers` distinct from public `users`.
- [ ] Implement protected dashboard request queue, unread/read notifications and new request status updates. Optionally add email fallback with only minimal content.
- [ ] Add audit events and bounded notification retries; test a failed email/WhatsApp job still leaves the request stored.

**Gate C:** real consultation is saved once, visible to admin on a fresh login, with access control verified and failure retry recorded. A public UI success toast must be tied to DB success, not just to a click.

## 4. Phase D — CRM and customer account

- [ ] Customers: list/detail/create/edit, filters, normalized contact information, customer UUID, source and restricted notes; safe duplicate candidate screen.
- [ ] Visits/sessions: customer relation, treatment/service, date, session sequence, status and follow-up schedule; do not confuse a consultation lead with an actual visit.
- [ ] Payments: amount, **explicit currency unit**, status, transaction reference and financial role permissions; no bank card/password data. No payment gateway requested for MVP.
- [ ] Follow-ups: due date, reason and status; overdue count on admin overview. Leave automated clinical follow-up messages for a separate approved feature; user requested only basic CRM and consultation alerts in this session.
- [ ] Admin mobile cards and desktop tables; search, loading, empty states, validation, confirmation for destructive actions and immutable audit log.
- [ ] Build optional customer registration/login + `My Account` sheet **only after** verification, recovery, security and privacy reviews. Never automatically expose archived CRM details based on the same typed phone number.

**Gate D:** all five CRM areas pass CRUD and permission tests with synthetic records; no real patient account access until identity linking is reviewed.

## 5. Phase E — Google Sheets two-way sync

- [ ] Clinic creates a private Sheet with five tabs exactly as defined in `Architecture.md`; protect ID/system columns and set access for authorized staff only.
- [ ] Enable Google Sheets API and create least-privilege service account; share **that specific Sheet** with the service-account email as Editor. Store key server-side only. Verify host outbound HTTPS.
- [ ] Implement connector reads/batch writes and `sync_outbox`, `sync_state`, deterministic field fingerprints and advisory lock. Configure allowed editable columns per tab.
- [ ] Configure cPanel Cron at allowed interval (target 1–5 minutes, but display real interval). Sync panel changes out and approved Sheet edits in. Include backoff on quota/network errors.
- [ ] Detect same-field conflicts; show a resolution screen; never silently last-write-wins or delete a client because a Sheet row disappeared.
- [ ] Sync test cases: panel edit → Sheet, Sheet edit → panel, new records on each side, sorted rows, transient Google outage, duplicate IDs, blank fields, concurrent edits, conflict resolution and repeated idempotent runs.

**Gate E:** successful bidirectional sync for 20 synthetic records, visible `last_sync_at` and error/conflict screens. Do not describe polling sync as instant real time.

## 6. Phase F — Excel prototype and controlled migration

- [ ] Generate a **synthetic 20-customer** workbook matching the five approved schema tabs. It is a test fixture, not the clinic’s actual customer list.
- [ ] Build template version check, upload validation, preview (new/update/duplicate/error/conflict), confirmation step, import batch ID, audit and backups.
- [ ] Support idempotent re-import and explicit conflict policy: existing stable UUID first; unknown/ambiguous matches reviewed manually. Blank spreadsheet cells must not wipe good data unexpectedly.
- [ ] Commit approved batch to MySQL transactionally and queue Sheet updates; never upload an unreviewed workbook directly into production Google Sheet.
- [ ] Test wrong headers, malformed dates, Iranian vs international number formatting, currency conversion ambiguity, duplicates, malicious formula cells, 20 records and a realistic larger sample.
- [ ] Replace fixture with actual clinic workbook only after field mapping and consent/authorization review; do not place sensitive exports on GitHub or in public directories.

**Gate F:** preview totals equal committed totals, repeat import does not duplicate rows, and new/changed values propagate to Sheet after the sync acknowledgment.

## 7. Phase G — WhatsApp feasibility decision and notification adapter

- [ ] Build interface `NotificationChannel` with `in_app` required; `email` optional; `whatsapp` optional/feature-flagged. Store a configurable **admin recipient number** in protected settings; avoid hard-coded numbers in client code.
- [ ] Validate the official Meta WhatsApp Cloud API sender/account setup, billing conditions, template rules and ability to deliver a **minimal admin alert** to the chosen recipient. Confirm actual cost for the clinic rather than assuming a permanent free allowance.
- [ ] Evaluate whether any zero-additional-cost, reliable, policy-compatible delivery is genuinely available **without a VPS or persistent browser process**. The existence of OpenWA is not evidence that it runs safely on this shared host.
- [ ] If automatic WhatsApp cannot meet the constraints, **do not mark it complete**. Keep the in-app notification (required) and optional email backup operational; retain a `wa.me` visitor shortcut only as a manual alternative with clear behavior.
- [ ] If a suitable solution is verified and expressly accepted, enable adapter on staging, test success/failure/retry/deduplication and show delivery status to admin. Never include sensitive medical or financial data in notification text.

**Gate G:** either validated automatic WhatsApp without unapproved expense/infrastructure, or documented pending integration and a reliable functioning in-panel fallback. No fictitious “message sent” toast.

## 8. Cross-cutting quality gates

| Area | Tests / acceptance |
|---|---|
| Visual/mobile | 320/360/390/430/768/1024/1440px, RTL, safe areas, no clipped hero or bottom-nav overlap |
| Accessibility | Keyboard focus, labels, dialog trap/return, contrast, reduced motion, alt text, error announcement |
| Performance | Measure actual LCP/CLS/INP on phone and staging, compare to targets in `Design.md`; optimize before claiming pass |
| Security | SQL injection/XSS/CSRF checks, role bypass, auth brute-force, IDOR on customer IDs, direct URL access to private files |
| Data integrity | FK checks, transaction rollback, repeated submit/import, sync outage, conflict and deterministic retry |
| Privacy | Synthetic fixtures in repo, private credentials, authorized Sheet editors, patient photo permissions, scrubbed logs |
| Hosting | cPanel Cron actually executes, private paths cannot be fetched, JSON API errors remain JSON, backup restore verified |
| Notifications | Request persists if all optional channels fail; admin unread count correct; no duplicate sends on retry |
| Deployment | Staging approval, deployment backup, health check, rollback and post-deploy smoke test |

Use focused automated tests for business-critical paths plus a manual device review; do not build an excessive CI matrix before functional MVP. Accept/merge the branch after review; **do not deploy/overwrite the existing live site by default**.

## 9. Deliverables and dependency order

```
A audit + hosting + assets
  ├── B visual system → mobile UI → hero/sheets → public staging review
  └── C PHP/MySQL + auth + consultation persistence → admin alerts
         └── D CRM (5 areas) + protected account (optional)
               └── E Google Sheets bidirectional sync
                     └── F Excel prototype + eventual real-data migration
C ────────────────→ G WhatsApp feasibility / optional notification adapter
All phases ───────→ security, tests, staging, approval, controlled deployment
```

| Deliverable | Completion artifact |
|---|---|
| Approved visual design | screenshots, final brand tokens, approved logo and copy |
| Public web app | responsive staging link, tested consultation entry UX |
| PHP + DB | versioned schema, API, auth, private config template, migrations |
| Protected CRM | admin screens for five entities, audit and notifications |
| Sheets integration | private Sheet, sync status dashboard, tested bidirectional flows |
| Excel workflow | synthetic workbook, template, validated preview/commit, migration instructions |
| WhatsApp decision | tested channel **or** explicit pending status and fallback |
| Operations | cPanel install/runbook, Cron config, backup/restore, release/rollback steps |

## 10. Questions requiring the clinic's approval

1. Which exact logo file, primary purple and original font lockup are final? Do we have approved smile imagery?
2. Which staff need edit rights in the Sheet and CRM? Which roles may see payment data?
3. Is a public **account** needed on first release, or is a consultation form plus private admin sufficient initially?
4. What is the source dataset's real field structure, how are amounts represented (IRR or toman), and are there existing stable customer IDs?
5. What is the clinic's real cPanel PHP/Cron/HTTPS configuration and what domain/staging path can be used?
6. Is free **automatic** WhatsApp a strict launch blocker, or can in-app notification + existing hosting email be used until a viable channel is verified?

### Reference documentation

- [Design.md](./Design.md) — visual behavior, navigation, hero and copy.
- [Architecture.md](./Architecture.md) — storage, data schema, synchronization, security and infrastructure.
- Anime.js SVG draw: https://animejs.com/documentation/svg/createdrawable/
- Anime.js scroll integration: https://animejs.com/documentation/events/onscroll/
- Google Sheets API usage limits: https://developers.google.com/workspace/sheets/api/limits
- Meta WhatsApp Cloud API: https://developers.facebook.com/docs/whatsapp/cloud-api/overview
