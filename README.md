# Miraas Clinic

Mobile-first public prototype for Dental Miraas Clinic.

## Included in this package

- RTL Persian interface focused exclusively on smile design, laminate and composite services.
- Compact SVG smile-design hero animated with Anime.js.
- Responsive service cards, selected results, methodology, FAQ and consultation CTA.
- Mobile bottom navigation with Home, Services, Results, Consultation and Account surfaces.
- Accessible consultation bottom sheet with reduced-motion support.
- Existing logo, result photography and motion assets preserved.
- Previous landing page preserved at [`legacy/index.html`](./legacy/index.html).
- Product specifications included in [`docs/`](./docs/).

## Current status

This package contains the reviewable public UX foundation only. The consultation form is intentionally marked as prototype mode and does not claim to send data. PHP/MySQL persistence, protected admin CRM, Google Sheets sync and notification adapters require the cPanel environment and credentials described in the architecture documents.

## Local preview

From the repository root:

```bash
python3 -m http.server 8080
```

Then open `http://localhost:8080/`.

## Push with GitHub CLI

```bash
unzip MiraasClinic-prepared.zip
cd MiraasClinic-prepared
git init
git add .
git commit -m "Add Miraas mobile redesign foundation"
gh repo create hadimoti/MiraasClinic --source=. --remote=origin --push
```

If the repository already exists locally, add the files to a new branch and open a pull request instead of force-pushing `master`.
