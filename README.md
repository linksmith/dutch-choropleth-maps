# dutch-choropleth-maps

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

An AI agent skill for creating choropleth maps of Dutch statistical data at gemeente, wijk, or buurt level. Compatible with Claude Code, Open Code, Kilo Code, Cursor, Windsurf, Cline, Aider, and 40+ other AI coding tools.

Designed to work with output from [cbs-statline-skill](https://github.com/linksmith/cbs-statline-skill) and [data-cleaning-dutch](https://github.com/linksmith/data-cleaning-dutch).

## Overview

This skill transforms an AI agent into a knowledgeable Dutch cartography partner that can:

- Auto-detect the geographic level (gemeente, wijk, buurt) from CBS region code prefixes (GM, WK, BU)
- Fetch official boundaries from PDOK WFS (the authoritative Dutch geodata service)
- Fall back to cartomap/nl simplified boundaries when PDOK times out
- Convert CRS from EPSG:28992 (RD New / Rijksdriehoekscoördinaten) to EPSG:4326 (WGS84) for web maps
- Merge statistical data with geodata on `statcode`, with quality checking and mismatch detection
- Render with geopandas (static PNG/SVG), Folium (interactive HTML), or Plotly (dashboard HTML)
- Apply journalism rules: finding-first titles, correct colour scales, grey areas for missing data

A key concept: **year-matching between CBS statistical data and PDOK boundaries**. The Netherlands regularly merges municipalities (herindelingen), causing GM-codes to change. Using boundary data from a different year than your statistical data is the most common cause of a grey or nearly empty map.

## Installation

### Dependencies

No Python helper module — this is a pure prompt skill. You will need:

```bash
pip install geopandas folium plotly requests matplotlib
```

Or with uv:

```bash
uv pip install geopandas folium plotly requests matplotlib
```

### Quick Install (Any Agent)

If you use the [Vercel Skills CLI](https://github.com/vercel-labs/skills), this works across 40+ agents:

```bash
npx skills add linksmith/dutch-choropleth-maps
```

See below for tool-specific instructions.

### Cursor

**Option 1: Clone to Cursor rules directory**

```bash
git clone https://github.com/linksmith/dutch-choropleth-maps.git ~/.cursor/rules/dutch-choropleth-maps
```

**Option 2: Add to project `.cursorrules`**

```bash
curl -L https://raw.githubusercontent.com/linksmith/dutch-choropleth-maps/main/SKILL.md -o .cursorrules
```

**Option 3: Project-level installation**

```bash
git clone https://github.com/linksmith/dutch-choropleth-maps.git .cursor/dutch-choropleth-maps
```

Then reference in `.cursorrules`:
```
Use the dutch-choropleth-maps skill in .cursor/dutch-choropleth-maps/ for all Dutch choropleth map requests.
```

### Windsurf (Codeium)

**Option 1: Global rules**

```bash
git clone https://github.com/linksmith/dutch-choropleth-maps.git ~/.windsurf/rules/dutch-choropleth-maps
```

**Option 2: Project-level**

Create `.windsurf/rules/dutch-choropleth-maps.md` in your project:
```bash
curl -L https://raw.githubusercontent.com/linksmith/dutch-choropleth-maps/main/SKILL.md -o .windsurf/rules/dutch-choropleth-maps.md
```

### Claude Code / Open Code / Kilo Code

All three tools support the same plugin format:

**Option 1: Install as a plugin** (recommended, no npm/node required)

```bash
claude plugin install --from https://github.com/linksmith/dutch-choropleth-maps
```

Replace `claude` with `open` or `kilo` depending on your tool.

**Option 2: Add as a project skill**

```bash
mkdir -p .claude/skills
git clone https://github.com/linksmith/dutch-choropleth-maps.git .claude/skills/dutch-choropleth-maps
```

**Option 3: Add as a slash command**

```bash
mkdir -p .claude/commands
curl -L https://raw.githubusercontent.com/linksmith/dutch-choropleth-maps/main/SKILL.md \
  -o .claude/commands/dutch-choropleth-maps.md
```

### Cline (VS Code Extension)

**Option 1: Add to .clinerules**

```bash
curl -L https://raw.githubusercontent.com/linksmith/dutch-choropleth-maps/main/SKILL.md -o .clinerules
```

**Option 2: Workspace settings**

1. Clone the skill to your workspace:
```bash
git clone https://github.com/linksmith/dutch-choropleth-maps.git .cline/dutch-choropleth-maps
```

2. In VS Code settings, add to `cline.customInstructions`:
```
Use dutch-choropleth-maps skill for Dutch geographic visualisations. See .cline/dutch-choropleth-maps/SKILL.md
```

### Roo Code (VS Code Extension)

**Option 1: Add to .roorules**

```bash
curl -L https://raw.githubusercontent.com/linksmith/dutch-choropleth-maps/main/SKILL.md -o .roorules
```

**Option 2: Custom instructions**

In VS Code, open Roo Code settings and add to Custom Instructions:
```
For Dutch choropleth maps with PDOK geodata, reference the skill at:
https://github.com/linksmith/dutch-choropleth-maps

Download SKILL.md for the full pipeline.
```

### Aider

**Option 1: Add as read-only context**

```bash
git clone https://github.com/linksmith/dutch-choropleth-maps.git ~/skills/dutch-choropleth-maps

aider --read ~/skills/dutch-choropleth-maps/SKILL.md \
      --read ~/skills/dutch-choropleth-maps/references/pdok-endpoints.md \
      --read ~/skills/dutch-choropleth-maps/references/crs-guide.md
```

**Option 2: Add to .aider.conf.yml**

```yaml
read:
  - ~/skills/dutch-choropleth-maps/SKILL.md
  - ~/skills/dutch-choropleth-maps/references/pdok-endpoints.md
  - ~/skills/dutch-choropleth-maps/references/crs-guide.md
```

### OpenHands

**Option 1: Add to workspace**

```bash
git clone https://github.com/linksmith/dutch-choropleth-maps.git .openhands/dutch-choropleth-maps
```

**Option 2: Custom instructions**

Add to `.openhands/instructions.md`:
```
For Dutch choropleth maps (gemeente, wijk, buurt):
1. Read .openhands/dutch-choropleth-maps/SKILL.md for the 5-step pipeline
2. Reference pdok-endpoints.md for PDOK WFS URLs by year
3. Reference crs-guide.md for coordinate system conversion
```

### Goose (Block)

**Option 1: Add to Goose extensions**

```bash
git clone https://github.com/linksmith/dutch-choropleth-maps.git ~/.goose/extensions/dutch-choropleth-maps
```

**Option 2: Add instruction file**

```bash
curl -L https://raw.githubusercontent.com/linksmith/dutch-choropleth-maps/main/SKILL.md -o ~/.goose/instructions/dutch-choropleth-maps.md
```

### GitHub Copilot

**Option 1: Add to .github/copilot-instructions.md**

```bash
mkdir -p .github
curl -L https://raw.githubusercontent.com/linksmith/dutch-choropleth-maps/main/SKILL.md -o .github/copilot-instructions.md
```

**Option 2: Reference in VS Code settings**

In `.vscode/settings.json`:
```json
{
  "github.copilot.chat.codeGeneration.instructions": [
    {
      "file": ".github/copilot-instructions.md"
    }
  ]
}
```

### Generic AI Assistants

For any AI assistant that supports custom instructions or context files:

1. Download the SKILL.md file:
```bash
curl -L https://raw.githubusercontent.com/linksmith/dutch-choropleth-maps/main/SKILL.md -o dutch-choropleth-maps.md
```

2. Paste the contents into your AI assistant's custom instructions or system prompt.

## Usage

The skill responds in Dutch if the user writes in Dutch. It is designed for mixed audiences (journalists, data analysts, and coders) and generates complete, runnable Python code.

### Example Prompts

**Basic map creation:**
```
Maak een choroplethkaart per gemeente van zonnepanelen
```

**Troubleshooting a grey or empty map:**
```
Mijn kaart is bijna leeg, hoe los ik dat op?
```
The skill will diagnose merge quality, check for trailing spaces in join keys, and warn about year mismatches caused by herindelingen.

**Choosing a renderer:**
```
Gebruik een interactieve Folium-kaart
```

**Colour scale guidance:**
```
Welk kleurschema gebruik ik voor afwijking van het gemiddelde?
```
The skill recommends diverging scales (RdBu_r) for deviation from average and sequential scales (YlOrRd) for absolute quantities.

### Five-Step Pipeline

1. **Detect** — auto-detect geographic level from region code prefix (GM=gemeente, WK=wijk, BU=buurt)
2. **Fetch** — download official boundaries from PDOK WFS; fall back to cartomap/nl for timeouts
3. **CRS** — convert from EPSG:28992 (RD New) to EPSG:4326 (WGS84) for web maps
4. **Merge** — join statistical data with geodata on `statcode`; report match percentage and flag mismatches
5. **Render** — output with geopandas (static PNG/SVG), Folium (interactive HTML), or Plotly (dashboard HTML)

### Journalism Rules

- **Title states the finding**, not the variable: "Noord-Groningen loopt achter op energietransitie" — not "Aardgasaansluitingen per gemeente 2022"
- **Sequential colour scales** (YlOrRd, YlGnBu) for quantities and counts
- **Diverging colour scales** (RdBu_r) for deviation from average — centred at zero
- **Grey areas** signal missing data — never hidden, always labelled "Geen data"
- **Source line**: "Bron: CBS StatLine, PDOK gebiedsindelingen {jaar}"
- **Herindelingen warning**: when using historical data, the skill warns that gemeente mergers may cause year-mismatch problems and instructs you to use the PDOK endpoint matching your statistical year

## Skill Structure

```
dutch-choropleth-maps/
├── SKILL.md                        # Main skill definition (5-step pipeline)
├── evals.json                      # Evaluation prompts for skill testing
└── references/
    ├── pdok-endpoints.md           # PDOK WFS URL templates, available years, field mappings
    └── crs-guide.md                # CRS reference (EPSG:28992, EPSG:4326, EPSG:3857)
```

## External Data Sources

- **PDOK WFS**: `https://service.pdok.nl/cbs/gebiedsindelingen/` — official Dutch geographic boundaries, year-specific endpoints
- **cartomap/nl**: `https://cartomap.github.io/nl/wgs84/` — simplified fallback boundaries already in WGS84 (EPSG:4326)

## Related Skills

- [cbs-statline-skill](https://github.com/linksmith/cbs-statline-skill) — fetch and clean CBS StatLine data; provides the DataFrame that this skill maps
- [data-cleaning-dutch](https://github.com/linksmith/data-cleaning-dutch) — Dutch-specific data cleaning patterns, including region code normalisation

## License

MIT License — see [LICENSE](LICENSE) for details.

## Contributing

Contributions welcome! Please feel free to submit issues or pull requests.

- **Issue Tracker:** https://github.com/linksmith/dutch-choropleth-maps/issues
- **Pull Requests:** https://github.com/linksmith/dutch-choropleth-maps/pulls

## Resources

- [PDOK (Publieke Dienstverlening Op de Kaart)](https://www.pdok.nl/)
- [CBS Gebiedsindelingen WFS](https://service.pdok.nl/cbs/gebiedsindelingen/)
- [cartomap/nl](https://github.com/cartomap/nl)
- [geopandas documentation](https://geopandas.org/)
- [Folium documentation](https://python-visualization.github.io/folium/)
