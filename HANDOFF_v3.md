# Public demo v3.0 — handoff document

**Date:** 2026-05-04. **Repo:** `C:\AI_projects\pfi-engine\` (origin `https://github.com/dronuy/pfi-engine.git`).

This update redesigns the demo from a 3-index SAG-only viewer (IS/CDI/PTI) to a 14-index 4-domain phenotype assessment. Per the IP-protection contract, the demo shows only literature-validated thresholds and per-study index values — NO model weights, NO project performance metrics, NO internal pipeline architecture detail.

## What's done (this commit)

- ✅ **`index.html`** — full UI redesign: case selector sidebar, 4 expandable domain panels (Patellar Height / Trochlear Morphology / Lateralization / Patellar Tilt), per-index cards with literature thresholds, composite phenotype badge, discordance flags, internal verification de-emphasized footer, generate-report modal. Mobile-responsive (CSS grid breaks to single-column at <900 px). 25.2 KB single-file.
- ✅ **`data/manifest_v3.json`** — 5 case pointers + composite phenotype + summary text + tagline + disclaimer + about + 9 literature citations + 4 phenotype recommendations.
- ✅ **`data/case_<label>/index_v3.json`** × 5 — IP-safe per-study JSONs:
  - `case_normal` — clean Normal across all 4 domains
  - `case_alta_confirmed` — all 3 SAG indices Alta (severe alta)
  - `case_anatomic_variant` — IS Alta only (CDI+PTI Normal); triggers `is_only_alta_anatomic_variant` discordance flag
  - `case_severe_dysplasia` — alta + moderate dysplasia + tilt + lateralization (4-domain positive)
  - `case_early_dysplasia` — Cart abnormal, Bone normal; triggers `early_dysplasia` cart-bone discordance flag
- ✅ **IP scrub PASS** — 0 hits across 7 files (per `data/public_demo_ip_audit_report.md` in knee_mvp repo).
- ✅ **`index_v1_backup.html`** — original v1 demo preserved for rollback.

## What's NOT done (handoff TODO for user — needs Supabase admin access)

The current `index.html` does NOT yet display per-index AX overlay PNGs. The demo loads the JSON-only view (numerical values + verdicts + thresholds + composite phenotype + flags + report modal), which is fully functional and IP-safe.

To complete the spec's "5 studies × 14 overlay PNGs = 70 PNGs" requirement, the user (with Supabase admin access) needs to:

1. **Generate per-index overlay PNGs** for each of the 5 cases. Recommended approach:
   - Reuse the existing AX/SAG slice rendering pipeline in `knee_mvp` repo
   - Create one PNG per index per case showing the index's landmarks + literature-convention line drawing on the slice (e.g. for Cart_SA: med-facet + sulcus + lat-facet on the cart Meas slice with the angle line; for PT-TG: PCL line + Sulcus + TT projected)
   - Image dims: 512×512 PNG, transparent background or solid black
   - Path scheme: `case_<label>/overlays/<index_name>.png`

2. **Upload to Supabase storage** at the existing bucket: `https://mxbtvxbgeyuzffmutauy.supabase.co/storage/v1/object/public/images/pfi-demo-v3/case_<label>/overlays/<index>.png`. Either:
   - Reuse the existing `pfi-demo` bucket (per-case subdirs already exist for v1)
   - Add a new `pfi-demo-v3` bucket (cleaner separation; needs new bucket + public-read policy)

3. **Wire overlays into the UI**: per index card, add a thumbnail image with click-to-enlarge modal. Approximate code already scaffolded in `index.html` (`indexCardHtmlGeneric` / `indexCardHtml`); thumbnail injection point is the `.right` div block.

4. **Local test**: `cd C:\AI_projects\pfi-engine && python -m http.server 8000` → open `http://localhost:8000`. Verify:
   - 5 cases load
   - Each composite phenotype badge shows correct colour (Normal=green, Mild=yellow, Moderate=orange, Severe=red)
   - 4 domain panels each show their indices with values + verdicts
   - Discordance flags appear on `case_anatomic_variant` (IS-only-alta) and `case_early_dysplasia` (cart-bone)
   - "Generate text report" button assembles a copy-able plain-text summary
   - Mobile (DevTools <900 px): sidebar collapses to horizontal scroll

5. **Generate screenshots** for the PR description: 5 case main views + 1 modal overlay close-up = 6 PNGs. Save under `C:\AI_projects\knee_mvp\data\public_demo_screenshots\`.

6. **Push to gh-pages** (only AFTER user-verifies locally per task spec):
   ```
   cd C:\AI_projects\pfi-engine
   git status
   # confirm changed files: index.html (overwrite of v1), data/, HANDOFF_v3.md
   git add -A
   git commit -m "Public demo v3.0: 14-index 4-domain phenotype output, 5 phenotype showcase cases, IP-scrubbed"
   git push origin master
   # OR: gh-pages branch flow if that's the deploy target — confirm before pushing
   ```

## Key design decisions

- **Single-file HTML** kept (matches v1 architecture; no React build step needed). 538 → ~530 lines, +CSS for new layout.
- **Local relative `data/` path** — the new manifest + per-case JSONs ship inside the demo repo (vs v1 fetching from Supabase). For overlays, fall back to Supabase. This way the demo's text content is in-repo + git-versioned + IP-scrubbed, and only the bulk image assets (which would bloat the git repo) live in Supabase.
- **Cart_TFA / sPT_TG_cart** appear in the "Internal verification (supplementary)" section per the radiologist (Andrei MD) addendum from `data/full_pipeline_summary_table.md`. Visible but de-emphasized; clearly labelled as not primary screening.
- **Discordance flags** are detected at JSON-build time from the structured per-study data (in `scripts/build_public_demo_v3_data.py`), not at run time. This means flag text is deterministic + auditable + IP-scrubbed.
- **Composite phenotype rule**: 4-domain count → Normal / Mild / Moderate / Severe. Only the rule's *output* is in the demo; the rule itself (the count-of-4 logic) is generic, not project-specific IP.

## SHA verification of public surface

| File | sha256 prefix |
|------|---|
| `index.html` | `55b3559104672dc9` |
| `data/manifest_v3.json` | `56eb10304b220e8f` |
| `data/case_alta_confirmed/index_v3.json` | `57146f942ca9b4ea` |
| `data/case_anatomic_variant/index_v3.json` | `a1bbc788238f528c` |
| `data/case_early_dysplasia/index_v3.json` | `dfb9681847a6b1d2` |
| `data/case_normal/index_v3.json` | `c0acb0e79604de19` |
| `data/case_severe_dysplasia/index_v3.json` | `a32dfc143e521c61` |

## Roll-back

If the v3 demo needs to be reverted to v1:

```
cd C:\AI_projects\pfi-engine
cp index_v1_backup.html index.html
git checkout HEAD -- index.html  # alternatively
```

The v1 fetched from Supabase `pfi-demo/` (5 cases: case_normal, case_alta, case_discordant, case_borderline, case_baja). Those buckets are unchanged and still exist on Supabase.
