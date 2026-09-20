# Miraas Clinic — Design Specification

**Version:** 1.0 (planning baseline) · **Date:** 2026-09-20 · **Language:** Persian / RTL · **Target:** mobile-first, app-like website for an aesthetic dental clinic

> This document records the design decisions expressed in the planning conversation. It is a specification, not a claim that the redesigned UI has been implemented. The supplied logo image in the conversation is the visual reference; the original high-resolution/transparent asset must be provided or approved before final integration.

## 1. Product positioning and non-negotiables

- میراث یک **کلینیک دندان‌پزشکی زیبایی** است؛ خدمات محوری: **طراحی لبخند، لمینیت و کامپوزیت**. متن‌ها و کارت‌ها نباید کلینیک را به‌عنوان مرکز پوست، جوان‌سازی یا درمان عمومی دندان معرفی کنند.
- تجربهٔ موبایل باید به یک اپلیکیشن نزدیک باشد: bottom navigation ثابت، صفحات/بخش‌های کوتاه، modal/bottom sheet برای اقدامات، بازخورد لمسی و انیمیشن هدفمند. این سایت همچنان وب‌سایت است و نصب اپ اجباری نیست.
- رنگ اصلی **سفید + بنفش نسبتاً تیره و کنترل‌شده** است؛ بنفش نئونی فعلی `#C026FF` مبنای بازطراحی نیست.
- طرح هیرو تمام‌صفحه، ویدئوی طولانی و اسکرول اجباری انتخاب نشده است. پیام اصلی: **«ما طراحی دندان‌ها را بر اساس لبخند شما انجام می‌دهیم.»** با شکل و حرکت نشان داده شود، نه فقط یک پاراگراف تبلیغاتی.
- تمام آیکون‌های ناوبری **SVG مینیمال و هماهنگ**؛ عدم استفاده از آیکون ایموجی، مجموعه‌های نامتجانس یا آیکون‌های سه‌بعدی سنگین.
- اجرای انیمیشن‌های ورود، اسکرول و تعامل با **Anime.js**؛ نه انیمیشن سراسری دائم که خوانایی، باتری و سرعت را تخریب کند.

## 2. Existing repository: what to preserve / what to replace

Source reviewed: [`master/index.html`](https://github.com/hadimoti/MiraasClinic/blob/master/index.html) and repository tree at commit `757a93391ef1b2446a93a131899b7f9520638f30`. Currently the site is a single static HTML file (~62 KB) with inline CSS/JS, `.htaccess`, `_redirects`, `logo/`, `results/` (12 images), `BGNEW.webm` and `motion/teeth-new.webm`. It has an oversized sticky hero (140vh desktop, 120–130vh mobile), a 3D results slider, FAQ, aftercare, contact, a mobile hamburger menu and direct WhatsApp links. It does **not** currently include user authentication, a database, a CRM, Google Sheets sync or an automated notification service. Existing content is strongly composite-centric and uses several bright accent colors.

**Keep as reusable material:** approved result images (subject to patient permission), selected composite/aftercare content after clinical review, existing logos as reference, FAQ structure, metadata and any approved contact information. **Rebuild/rework:** hero, color tokens, bottom navigation, mobile layouts, forms, account surface, dynamic content flow, backend integration. The existing CSS/JS may be split into modules rather than retaining a monolithic file; there is no mandate to migrate to React or Next.js for this MVP.

## 3. Visual identity

### Design tokens (proposed, subject to approval against original logo)

| Token | Value | Usage |
|---|---|---|
| `--background` | `#FFFFFF` | Main canvas |
| `--surface` | `#F8F6FB` | Soft neutral panels |
| `--surface-tint` | `#EEE9F5` | Active backgrounds, subtle separators |
| `--primary` | `#5B3A75` | Primary actions, active nav, key headings |
| `--primary-deep` | `#382047` | Dark contrast panels and emphasis |
| `--primary-soft` | `#806499` | Secondary details |
| `--ink` | `#241A2C` | Primary text |
| `--muted` | `#716579` | Supporting text |
| `--stroke` | `#E9E3EE` | Borders |
| `--success` | `#26765F` | Form success, not main branding |

Purple should feel elegant and muted, rather than fluorescent. Use light surfaces, restrained gradients and generous whitespace. Avoid adding independent pink/cyan/yellow accent systems to every section. Check final text/button contrast with WCAG measurements before implementation.

**Typography:** Persian UI in Vazirmatn or Estedad; English microtype/wordmark only as needed. Keep `dir="rtl"` and isolate Latin text, phone numbers and prices with `dir="ltr"`. Prefer readable weights over ultra-thin Persian body text. Do not alter the supplied logo’s proportions or create an unapproved substitute.

**Shape language:** 16–24px rounded cards, pill CTAs, fine borders, soft realistic shadows. Do not overuse glassmorphism. The high-level impression should be clinical clarity + editorial luxury, not a generic beauty-salon landing page.

## 4. Mobile navigation and information architecture

A bottom bar is fixed to the lower viewport edge, with iOS/Android safe-area support: `padding-bottom: env(safe-area-inset-bottom)`. Suggested guest destinations:

| Item | Icon concept | Behavior |
|---|---|---|
| خانه | outline house | Go to landing top |
| خدمات | subtle tooth / sparkle | Service cards: composite, veneer, smile design |
| نمونه‌کارها | framed image | Before/after gallery |
| مشاوره | message/calendar outline | Open consultation bottom sheet |
| حساب من | person outline | Open login/register sheet or account screen |

For **admin accounts**, provide an additional visible **مدیریت** destination or an admin entry in the account area; do not show CRM controls to ordinary customers. Final decision on whether admin uses a separate `/admin` route is in `Architecture.md` (recommended). On desktop, preserve a conventional compact top navigation; bottom navigation may be hidden or adapted for wider screens.

Implementation rules: tap areas at least 44×44px, active state with both color and label, subtle press animation, no important content hidden behind the bar, keyboard/focus access, accessible names for all icons. Sections should be short, not a stack of 100vh carousels; default cards on phone should use compact ratios and horizontal scroll only where justified. Prefer a results carousel of manageable height (e.g. image ratio 4:5 or 1:1) instead of the current tall 9:16 3D stage. No mandatory autoplay; preserve manual swipe and clearly marked previous/next controls.

## 5. Hero — personalized smile-design visual

**Selected concept:** a compact interactive **SVG smile path → designed smile**. This explicitly replaces the full-screen cinematic-video proposal that was rejected.

Storyboard:

1. A clean white panel with a small MIRAAS logo, a thin curved SVG line and ample breathing room appears immediately. The main line visually evokes the silhouette of a smile.
2. With Anime.js, the line is gradually drawn over approximately 600–900 ms. Three or four unobtrusive anchor points appear across the line; short guide lines/contours suggest assessment and personalized tooth design. This is a **visual metaphor**, not a claim that an automatic medical assessment has occurred.
3. An approved smile photograph or abstract tooth/smile outline fades in behind/alongside the drawn line (subject to available asset and consent). Do not morph a patient’s real before/after image into a misleading synthetic clinical result.
4. Headline: **«لبخند شما، نقطهٔ شروع طراحی ماست.»** Supporting line: «طراحی لبخند، لمینیت و کامپوزیت با توجه به ویژگی‌های چهره و سلیقهٔ شما.» (final copy requires clinic approval).
5. Primary CTA: **«درخواست مشاوره»** opens the consultation sheet. Secondary CTA: **«دیدن نمونه‌کارها»** scrolls directly to results. A small trio of meaningful chips can follow: «طراحی لبخند»، «لمینیت»، «کامپوزیت». Tapping a chip reveals a short in-place detail or section; never a long full-screen slideshow.

**Sizing:** initial mobile hero should *ideally* fit within roughly 55–70dvh, with an ordinary scroll to the next section and no scroll pinning. Exact pixel height depends on content, keyboard, screen size and accessibility text scaling. At 360/390px widths the CTA remains visible without an additional hero-scroll requirement on common screen heights. On wider layouts, the illustration and copy form a balanced two-column layout.

**Fallbacks:** `prefers-reduced-motion` renders the finished line and content immediately; slow devices get a static SVG/optimized image. If the smile photo is not approved, use only original vector artwork and keep the design functional. The previous large `BGNEW.webm` is not required for the new hero.

## 6. Key public screens

**Home:** compact hero → 3 aesthetic dental service cards → featured consented results → concise methodology/trust section → FAQ excerpt → contact/consultation CTA. Remove unrelated medical service language. Clinic claims such as success rates, durability or «بهترین» must be reviewed, not fabricated.

**Services:** service overview for composite, veneers and smile design; each card opens an in-page detail or concise sheet with the process, suitability disclaimer and CTA. No skin, cosmetic injectables or unrelated dental treatments.

**Results:** clear and honest before/after labels if both stages are shown; patient authorization/approved images required. Use constrained card height and a lightbox, not a very tall page. Do not generate altered “after” images and present them as authentic outcomes.

**Consultation:** a bottom sheet containing name, contact number, desired service (composite/laminate/smile design/unsure), preferred time (optional), brief non-clinical note (optional), privacy acknowledgement and submit. On successful server confirmation show a request ID and acknowledgment; the notification channel’s failure must not erase the request. Avoid collecting full medical history in a public lead form.

**Login/register/account:** phone/email and password or a verifiable identity flow **only after backend support exists**. Registration and login open over the current view; background dims and blurs. Profile shows only the logged-in user's own permitted fields and consultation request statuses. Never infer that a matching phone number alone authorizes access to an old CRM/patient record. For first deployment, if customer authentication is incomplete, show consultation without pretending account management exists.

**Admin:** separate protected interface with overview tiles (new requests, pending follow-ups, recent changes), «اطلاعات مشتری‌ها», sessions, payments, consultation requests, follow-ups, import/export and sync status. Admin data tables should have useful mobile cards, search/filter, edit drawer and pagination. On desktop, use a proper table.

## 7. Overlay and micro-interaction system

- Bottom sheets rise from the lower edge with a small handle and rounded top corners; desktop can use a centered dialog. `aria-modal`, focus trap, Escape/back handling, focus restoration and background scroll lock are mandatory.
- Background blur: e.g. `backdrop-filter: blur(8px)` plus a translucent overlay; avoid excessive GPU blur and provide a non-blur fallback. Privacy-sensitive content should never remain visible just because a modal is open.
- Anime.js timelines for hero entrance, navigation active indicator, section reveals, staggered cards and successful form state. Use CSS for simple `:hover` and focus styles. Respect reduced motion, avoid scroll hijacking and large layout shifts.
- Skeleton only for genuine asynchronous data loading; a blank animated splash screen is not required. Prefer meaningful first paint and lazy-load below-fold media.
- Error states must say what happened and what to do (e.g. «درخواست ثبت نشد؛ دوباره تلاش کنید»), not just a decorative shake.

## 8. Responsive, performance and accessibility acceptance

- Test 320, 360, 390, 430, 768, 1024 and 1440px widths; no horizontal overflow or clipped CTAs. Account for soft keyboard and device safe areas.
- Target Core Web Vitals as engineering goals: LCP ≤2.5s, CLS ≤0.1 and INP ≤200ms at the 75th percentile where measurement is available; do not claim achievement before measuring.
- Local, optimized image variants (WebP/AVIF where compatible); use `srcset`, lazy-loading below the fold, explicit dimensions. Avoid eagerly loading multi-megabyte videos. If the 3D tooth motion is preserved, defer loading until its section is near view and provide a poster.
- Semantic headings, alt text, high contrast, visible focus, labeled form fields, status announcements and error summaries. Buttons are buttons, links are links.
- Protect patient privacy: no personal details or hidden treatment information in public gallery metadata, analytics events or client-side JavaScript.

## 9. Pending visual approvals

1. Obtain the **original logo with transparent background** and confirm which Persian/English lockups are approved; the image supplied in this conversation is the reference, not necessarily an export-ready web asset.
2. Obtain permission-cleared smile and before/after photography; determine whether hero should use approved photo or pure vector artwork.
3. Confirm exact brand purple from source brand file, final hero copy, service names and clinic contact details.
4. Confirm whether customer login is required on day one; keep it functionally gated until authentication and backend protections pass testing.

Related specifications: [`Architecture.md`](./Architecture.md) · [`Implentation-plan.md`](./Implentation-plan.md).
