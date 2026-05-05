# Public demo v3.1 — handoff document

**Date:** 2026-05-04. **Repo:** `C:\AI_projects\pfi-engine\` (origin `dronuy/pfi-engine`, branch `main`).

## What v3.1 delivers (this commit)

- **Dual DICOM viewer** (SAG left, AX right) replacing the v3.0 text-only demo.
- **321 pre-rendered DICOM slices** (5 cases × ~32 SAG + ~33 AX), 512×512 PNG, percentile-norm 1–99 → 8-bit greyscale, AX canonical-flipped to Med-at-LOW-x. Sourced under `data/case_<label>/slices/{sag,ax}/<NN>.png`.
- **Slice scrubber** (◀ / ▶ + range slider + arrow keys + mouse wheel + AX index-jump buttons [Meas / LTI / PCL / Tilt / TT]).
- **ANALYZE button** → SVG overlay layer drawn directly over slice (no PNG overlay regeneration), in literature conventions (cyan = cart medial, red = lateral, orange = bone, white = PCL reference, yellow = tendon / projection / patella tilt axis, green = sulcus / baseline). Index value + verdict label inline next to its overlay anchor.
- **Per-slice overlay logic**: only landmarks relevant to current AX slice are drawn (LTI overlays only on LTI slice; SA/TD overlays only on Meas slice; etc.). PCL line drawn on PCL slice and projected dashed on slices that reference it.
- **4-domain results panel** below viewers: Patellar Height (IS/CDI/PTI), Trochlear Morphology (cart Tanaka 2023 + bone Pfirrmann/Carrillon), Lateralization (PT-TG Hinckel), Patellar Tilt (Sallay).
- **Discordance flags** with plain-language explanations (`is_only_alta_anatomic_variant`, `early_dysplasia`, `bony_only_consider_OA`).
- **Internal verification** section labels `Cart_TFA` + `sPT_TG_cart` as supplementary, NOT primary screening (per Andrei MD addendum).
- **Generate text report** modal with copy-to-clipboard.
- **Mobile-responsive**: viewers stack at <1000px width.
- **IP scrub PASS**: 0 hits across 16 files (`index.html` + 5 × `slice_metadata.json` + 5 × `overlays.json` + 5 × `index_v3.json` from v3.0).

Wallclock-grade UI: instant analyze (no model inference at run-time; pre-baked landmark JSON). Local server URL: `http://localhost:8000`.

## File inventory (post-v3.1, demo repo)

```
C:\AI_projects\pfi-engine\
├── index.html                                  ← v3.1 (rewritten)
├── index_v1_backup.html                        ← v1 archive (unchanged)
├── HANDOFF_v3.md                                ← v3.0 handoff (unchanged)
├── HANDOFF_v3.1.md                              ← THIS FILE
├── data\
│   ├── manifest_v3.json                         ← v3.0 (unchanged; not consumed by v3.1 UI)
│   ├── case_<label>\
│   │   ├── index_v3.json                        ← v3.0 IP-safe per-case JSON
│   │   ├── overlays.json                        ← v3.1 per-slice landmark + index JSON
│   │   ├── slice_metadata.json                  ← v3.1 SAG + AX cohort + slice metadata
│   │   └── slices\
│   │       ├── sag\01.png … NN.png             ← v3.1 SAG slices (~26-36 per case)
│   │       └── ax\01.png … NN.png              ← v3.1 AX slices (~32-36 per case, canonical)
└── …
```

5 cases × ~33 SAG + ~33 AX = **321 slices** (12.6 MB total).
5 × `overlays.json` (~5 KB each) = 25 KB.
5 × `slice_metadata.json` (~0.5 KB each) = 2.5 KB.

## Manual verification checklist (do this before pushing)

Open http://localhost:8000 in browser:

1. **5 case cards visible** at top with composite badges (Normal/Severe/Mild/Severe/Moderate).
2. **Case selection**: click each — viewers should swap slice images + reset overlays.
3. **Slice scroll** (try on case_severe_dysplasia):
   - SAG: scroll wheel, ◀ ▶ buttons, arrow keys, slider — slices change smoothly
   - AX: same + click [Meas] [LTI] [PCL] [Tilt] [TT] buttons — should jump to those specific slices
4. **Pre-Analyze**: SVG overlay layer is empty (no lines/dots on slices).
5. **Click ANALYZE**:
   - SAG best slice now shows IS+CDI+PTI overlays simultaneously (cyan/yellow lines, dot markers, value+verdict labels)
   - AX shows overlays only when current slice == predicted slice for an index (e.g. on the Meas slice, you see SA + TD + TFA overlays; on the LTI slice, you see Cart+Bone LTI overlays)
   - Scroll AX away from a predicted slice → overlays disappear (correct behavior)
   - Results panel slides in below with composite phenotype badge + 4 domains expanded
6. **Verify discordance flags** on:
   - case_anatomic_variant: blue card "IS-only alta — anatomic variant suspected" visible
   - case_early_dysplasia: blue card "Cart-bone discordance — early dysplasia pattern" visible
   - case_alta_confirmed: blue card "Cart-bone discordance — early dysplasia pattern" (this one happened to also trigger that flag)
7. **Click Generate text report**: modal opens with plain-text summary, Copy to clipboard works.
8. **Mobile width** (DevTools < 1000px): SAG and AX stack vertically.

## Push to gh-pages (only after manual verification passes)

```bash
cd C:\AI_projects\pfi-engine
git status                                    # confirm clean (all committed)
git push origin main                          # if main is the deploy target
# OR if gh-pages is separate:
# git push origin main:gh-pages
```

Then wait ~1-2 min for GitHub Pages rebuild and confirm live at https://dronuy.github.io/pfi-engine/.

## What's NOT in v3.1 (deferred to v3.2 or later)

- **Hover tooltips** on landmark dots (would show landmark name + literature ref)
- **Index-toggle checkboxes** below each viewer (currently auto-shows all overlays for current slice)
- **Animated overlay fade-in** (currently instant draw on Analyze)
- **Mobile-swipe SAG↔AX** (currently CSS stack; arrow buttons still work)
- **Pre-render PNG overlays** (per the original v3.1 spec's 70-PNG approach) — replaced by lighter-weight SVG drawing on the fly from JSON (faster, smaller, more flexible)
- **6 Playwright PR screenshots** — Playwright not installed in this env. Manual screenshot procedure: load each URL state, browser DevTools → Capture full-size screenshot. Save to `data/public_demo_screenshots/` if needed for PR description.

## SHA verification

| File | sha256 prefix |
|------|---|
| `data/axial_trochlea_annotations_v4_pilot.json` | `354000DBE10E7EE3` (unchanged) |
| `data/axial_trochlea_annotations_v5_bone_lti_addon.json` | `E1D356A2AF240C25` (unchanged) |
| `data/axial_trochlea_annotations_v4_5_extended.json` | `E8FA95287DA76F21` (unchanged) |

## Roll-back

If v3.1 needs to be reverted to v3.0:

```bash
cd C:\AI_projects\pfi-engine
git revert <v3.1-commit-hash>
# v3.0 demo (52299b4) was JSON-only with no slice viewers; v1 backup at index_v1_backup.html
```

Or to v1:

```bash
cp index_v1_backup.html index.html
```
