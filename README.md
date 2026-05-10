# pfi-engine (archived)

Active development moved to **https://poluneev.com**. This repository is
retained for historical reference only; the live demo lives at
[poluneev.com/demo](https://poluneev.com/demo).

`index.html` here is a redirect page (meta-refresh + JS fallback) that
sends visitors to the new URL. The original demo (sample cases + dual
viewer + overlay rendering) was ported into the
[Dronuy/poluneev](https://github.com/Dronuy/poluneev) repo with these
additions:

* live DICOM upload (folder or ZIP) reaching `api.poluneev.com`
* server-side overlay rendering on uploaded slices
* sample case schema scrubbed of internal model identifiers
* roadmap + PFI Engine extended sections + subscribe form
* favicon + PWA manifest + brand teal `#0D9488`

The original assets (5 case slice PNGs, `index_v3.json`, `overlays.json`,
`slice_metadata.json`) remain in `data/` and are not served by the
redirect page, but they remain available via git history if needed for
provenance.

No new features will land here. File an issue on the
[Poluneev repo](https://github.com/Dronuy/poluneev/issues) instead.
