# Arogya — Health data sharing app

A working implementation of the `Health App.dc.html` Claude Design prototype
("Health data sharing app redesign"). It reads bloodwork lab reports and explains
them in plain language, with progress trends, a ranked action plan, a Stripe
paywall, and Google Calendar export.

## Run it

The app is a single self-contained `index.html` that pulls React from a CDN, so it
must be served over HTTP (opening the file directly with `file://` won't run the ES
modules).

```bash
cd "arogya-app"
python3 -m http.server 8777
```

Then open http://localhost:8777 — or use any static server (`npx serve`, etc.).

## What it does

- **Sign in** — Google Sign-In (falls back to a demo login if no client ID is set).
- **Upload** — drag & drop or browse lab reports (PDF/image). Seeded with 3 sample
  reports; uploaded files are dated from their `lastModified` and merged in.
  **PDF reports are parsed** (via pdf.js) into real marker values — see below.
- **Dashboard**
  - Plain-language hero summary of the whole panel.
  - Trend sparklines for key markers across reports.
  - "Needs your attention" and "Your action plan" — blurred until unlocked.
  - "Everything, by system" — every metric with a healthy-zone bar and one-line
    explanation, grouped into collapsible panels.
- **Paywall** — a $12 one-time unlock (demo Stripe flow; reveals the blurred content).
- **Google Calendar** — after unlocking, pushes the action-plan tasks to the user's
  calendar (uses the Google OAuth token client, with a `render?action=TEMPLATE`
  fallback that opens pre-filled event links).
- **Keys config** — Google OAuth Client ID and Stripe publishable key, stored in
  `localStorage` (`bp_gid`, `bp_spk`). Session/paid state is also in `localStorage`
  (`bp_user`, `bp_paid`).

## Connecting real keys

Click "Connect Google / Stripe keys" on the sign-in screen. For Google Sign-In and
Calendar to work, add the origin you serve from (e.g. `http://localhost:8777`) as an
authorized JavaScript origin on the OAuth client. Without keys, the demo login and
the calendar `TEMPLATE` fallback still work.

## Reading real reports (PDF parsing)

When you upload a **PDF**, the app extracts its text with pdf.js and matches each line
against a catalog of ~60 markers (with common aliases, e.g. `SGPT`/`ALT`,
`25-OH Vitamin D`, `Glycated Haemoglobin`). Recognised values replace the sample
values, and status (in range / borderline / out of range), the healthy-zone bars, the
"Needs attention" list, and the action-plan gating all recompute from your numbers. A
banner on the dashboard says how many markers were read and from which report; any
marker not found in your PDF falls back to its sample value.

Try it with the included [`sample-report.pdf`](sample-report.pdf) — a minimal synthetic
panel — or any text-based lab PDF.

**Limitations (it's a heuristic, not a certified parser):**

- **Text-based PDFs only.** Scanned/image PDFs and uploaded images aren't OCR'd yet —
  they're listed but keep sample values.
- Matching is line-based and picks the first standalone number after the test name.
  Unusual layouts (multi-column, values before names, footnote markers) can mis-read or
  miss a marker. Values are never a substitute for reading the lab's own report.
- Status thresholds for parsed values are computed heuristically (out if clearly past a
  bound, borderline if within ~15% of the range span). Sample rows keep their curated
  status from the original design.
- Trend sparklines use real history once **two or more** uploaded reports contain the
  same marker; otherwise they show the sample series.

The catalog + reference ranges live in the `data()` method, aliases in `aliases()`, and
the extract/parse/classify pipeline in `extractPdfText()` / `parseMarkers()` /
`classify()` in `index.html`.

## Notes

This is an educational explainer, not medical advice or a diagnosis.
