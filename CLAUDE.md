# Everflow Website Audit

AI-powered audit of everflow.io — full-page screenshots + Gemini 2.5 Flash messaging/design analysis.

## Live Report
**https://everflow-audit.vercel.app**

## Screenshots CDN
All screenshots are hosted publicly on Supabase Storage:
```
https://ysmcphkranfuljhrjiwz.supabase.co/storage/v1/object/public/website-audit/{filename}.jpg
```
Example: `https://ysmcphkranfuljhrjiwz.supabase.co/storage/v1/object/public/website-audit/homepage.jpg`

137 full-page screenshots, 1440px wide JPEGs.

## Quick Start
```bash
# Deploy changes to Vercel
cd ~/Projects/everflow-audit && npx vercel deploy --prod --yes
# (May need multiple retries due to SSL upload errors on large batches)

# Re-run audit on specific pages
cd ~/Library/Mobile Documents/com~apple~CloudDocs/Work/Projects/Google\ Colab
export GOOGLE_API_KEY="<key>"
.venv/bin/python everflow_website_audit.py

# Rebuild report HTML from results
.venv/bin/python build_report.py
```

## Architecture
- **Audit engine:** `~/Projects/Google Colab/everflow_website_audit.py` — Selenium CDP full-page screenshots + Gemini 2.5 Flash analysis
- **Report builder:** `~/Projects/Google Colab/build_report.py` — merges result JSONs + screenshots into static HTML
- **Report UI:** `public/index.html` — wrapped tab pills, grade badges, expandable screenshots, lightbox, gallery grid
- **Screenshots:** `public/screenshots/` — 1440px wide JPEGs
- **Results data:** `audit_v3_results.json` + `audit_batch2_results.json` + `audit_batch3_results.json` in Google Colab project
- **Mapping files** (in Google Colab project):
  - `screenshot_mapping.json` — full mapping (category, label, URL, type, Supabase URL)
  - `screenshot_mapping.csv` — for Google Sheets / Figma data plugins
  - `figma_import_urls.txt` — comma-separated Supabase URLs

## What's Covered
**~235 pages** across 17 categories:

### Full Audit (119 pages — AI analysis + screenshot)
| Category | Count |
|---|---|
| Homepage & Conversion | 8 |
| Platform | 9 |
| Action Plans | 4 |
| Solutions (Verticals) | 10 |
| Partners (Static) | 9 |
| Competitor Pages | 7 |
| Onboarding Landers | 5 |
| Amplify (Influencer) | 9 |
| Content | 2 |
| Boost (Referral) | 7 |
| Explore (SEO Landers) | 19 |
| Features | 5 |
| Chats (Webinars) | 8 |
| Compare (Competitor Alt) | 7 |
| Enablement (Sales Decks) | 10 |

### Screenshot Gallery (116 pages — no AI, thumbnail grid with click-to-expand)
| Category | Count |
|---|---|
| Chats Gallery | 41 |
| Enablement Gallery | 75 |

## Figma Import for Design Team

**The Figma REST API is read-only — it cannot create frames or upload images programmatically.**

### Approaches (in order of recommendation):

1. **Batch Image Import plugin (local files)** — Install "Batch Image Import" from Figma Community. Click "Choose Files", navigate to `~/Projects/everflow-audit/public/screenshots/`, select all JPGs, import. This is the most reliable method as it uses local files (no CORS/URL issues).

2. **Bulk Image Importer plugin (URLs)** — Install "Bulk Image Importer" from Figma Community. Paste comma-separated URLs from `figma_import_urls.txt`. Note: URL-based import may silently fail due to CORS or plugin bugs — test with 5 URLs first.

3. **Google Drive import** — Upload screenshots folder to a shared Google Drive, then use Figma's native file import or the "data.to.design" plugin with `screenshot_mapping.csv`. ~30 min manual setup.

4. **Custom Figma plugin** — Build a plugin using `figma.createImageAsync(url)` + `getSizeAsync()` to create named frames from the mapping JSON. ~2-4 hours dev effort. Max image dimension: 4096x4096px (our full-page screenshots exceed this vertically, so they'd need cropping).

5. **Web report as annotation layer** — Skip Figma entirely, add comment/pin functionality to the web report at everflow-audit.vercel.app. ~3-4 hours dev effort.

### Known Issues with Figma Plugins
- "Bulk Image Importer" (URL-based): tested Apr 1 2026, imports show "success" but 0 images actually appear on canvas. May be a plugin bug or Figma sandbox limitation.
- "Batch Image Import" (local files): shows 100% progress bar but images may not appear on canvas. Zoom out (Cmd+0) to check if they were placed far from viewport.
- Full-page screenshots are very tall (1440x8000+ px). Figma's plugin API has a 4096x4096 max for `createImageAsync`. Images may need to be cropped or split.

## Gemini Analysis Criteria
Each audited page is scored on:
1. ICP Alignment (1-10)
2. Messaging Clarity (1-10)
3. FOMO Factor (1-10)
4. Call to Action effectiveness
5. Case Study usage
6. AI/Innovation messaging
7. Competitor differentiation
8. Visual Design (1-10)
9. Above the Fold content
10. Top 3 improvements
11. Overall Grade (A-F)

## Screenshot Capture Details
- Headless Chrome via Selenium CDP `Page.captureScreenshot`
- 2x device scale factor (Retina)
- Full-page: scrolls to bottom (triggers lazy loading), scrolls back to top, captures entire scrollable area
- Output: 2880px wide PNGs, converted to 1440px JPEGs for web/Supabase
- PIL `MAX_IMAGE_PIXELS` set to 300M (handles very long blog/enablement pages)

## Collaboration
- **Dasha** — project lead, audit engine, report builder
- **Ceno Pant** (`cp-everflow`) — design team, UX review, subtask creation for page cleanup
- **Ticket:** DESGN board (Jira) — to be created
- **GitHub:** https://github.com/resourcehub40-create/everflow-audit

## Known Limitations
- Cookie consent banners appear on many screenshots (covers hero content)
- Carousels/sliders only capture one state
- Collapsed/accordion sections may not be expanded in screenshots
- Some pages return 404 (tracked in audit results as F grade)
- Vercel deploy has intermittent SSL upload errors — retry until success (caches progress incrementally)
- Compare pages exist at BOTH `/compare/cake` and `/competitor/cake` — different content/design

## Not Yet Audited
- **86 Enablement pages** (75 screenshotted in gallery, 10 fully audited, 1 remaining)
- **~800+ blog posts** (individual articles — same template, only index audited)
- **~600+ sitemap pages** beyond the 200 we crawled (long-tail SEO pages)

## Future Ideas
- Figma API integration (if they ever add write endpoints)
- Custom Figma plugin for bulk import with named frames
- Comment/annotation system on the web report
- Diff mode (re-run audit, compare before/after screenshots)
- Mobile viewport screenshots (375px width)
- Lighthouse performance scores alongside messaging audit
- Auto-expand carousels/accordions before screenshotting
