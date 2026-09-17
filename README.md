# APMEA — Alzheimer Pathology Morphological Embedding Atlas

Phase A starts with a **donor-level cohort lock** from the Emory BDSA folder `wsi_archive / APOLLO_NP`.

## Cohort export

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
# put DSA_API_KEY or DSA_TOKEN in .env

python scripts/export_apollo_np_cohort.py
# smoke test
python scripts/export_apollo_np_cohort.py --years 2022 --max-items 200
```

Create a BDSA API key under **Users → My account → API keys** (preferred over pasting a browser token).

Outputs in `data/` (gitignored; contains identifiable research fields):

| File | Grain | Use |
|---|---|---|
| `slides.csv` | one WSI | stain, region, block, item id, staging fields copied from the item |
| `donors.csv` | one case | stain panel, regions, Braak/CERAD/Thal/ABC, diagnoses |
| `phase_a_candidates.csv` | subset | HE + tau + Aβ, hippocampus or neocortex, and at least one staging field |
| `summary.json` | counts | lock the 40–60 donor pilot from this list |

## Inclusion rule (Phase A)

A donor is a candidate when all of these are true:

- at least one HE, one TAU/TAU3/TAU4, and one AB slide
- hippocampus and/or neocortex present
- Braak, ABC, CERAD, or Thal populated
- at least 3 SVS files

Split later **by donor**, never by patch.

## Local 20× tiles (Phase A lock)

SVS files live on this host at `/wsi_archive/APOLLO_NP/<year>/<case>/scanned images/`.
BDSA is used only for `npSchema` region names and clinical fields.

```bash
# optional: WSI_ARCHIVE_ROOT=/wsi_archive/APOLLO_NP in .env
python scripts/prepare_phase_a.py
# one case smoke test
python scripts/prepare_phase_a.py --cases E22-03 --max-preview-tiles 8
```

Writes (gitignored under `data/`):

| File | Use |
|---|---|
| `local_slides.csv` | every local SVS for the locked donors |
| `phase_a_selected_slides.csv` | HE + TAU + Aβ on the same hippocampus and neocortex block |
| `phase_a_slide_qc.csv` | OpenSlide size / MPP / tissue tile count |
| `phase_a_tile_index.csv` | 20× 256 level-0 coordinates (`read_px` ≈ 512 at 40×) |
| `previews/*_thumb.jpg` | slide thumbnail |
| `previews/*_tiles.jpg` | montage of tissue tiles |

There is no native 20× pyramid level (levels are 1 / 4 / 16). Tiles are read at level 0 as 512 px and resized to 256 px (0.50 µm/px).
