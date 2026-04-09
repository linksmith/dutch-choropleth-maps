# dutch-choropleth-maps Skill

AI agent skill for creating choropleth maps of Dutch statistical data using PDOK geographic boundaries.

## Project structure

- `SKILL.md` — Skill definition (5-step pipeline: detect → fetch → CRS → merge → render)
- `references/pdok-endpoints.md` — PDOK WFS URL templates, available years, field mappings
- `references/crs-guide.md` — CRS reference (EPSG:28992, EPSG:4326, EPSG:3857)
- `evals.json` — Evaluation prompts for skill testing

## External data sources

- **PDOK WFS**: `https://service.pdok.nl/cbs/gebiedsindelingen/` — official Dutch geographic boundaries
- **cartomap/nl**: `https://cartomap.github.io/nl/wgs84/` — simplified fallback boundaries (already in WGS84)

## Testing

Run the evals in `evals.json` manually. Pass threshold: all scoring_criteria met per eval.

Live endpoint health checks require network access and `geopandas`.
