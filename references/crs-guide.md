# CRS Guide — Coördinaatreferentiestelsels voor Nederlandse kaarten

Beknopte referentie voor het werken met coördinaatreferentiestelsels (CRS) bij het maken van Nederlandse kaarten.

---

## De drie CRS-stelsels die je tegenkomt

### EPSG:28992 — Rijksdriehoekscoördinaten (RD New)

- **Wat**: Het Nederlandse nationale coördinatenstelsel, ontworpen speciaal voor Nederland.
- **Eenheden**: Meters (X/Y-coördinaten in meters t.o.v. een referentiepunt in Amersfoort)
- **Vervorming**: Minimaal voor Nederland — de beste keuze voor statische kaarten van NL
- **Bron**: PDOK levert alle geodata standaard in EPSG:28992
- **Gebruik**: Statische kaarten met geopandas (`.plot()`), oppervlakteberekeningen in m²

Voorbeeld coördinaten Amsterdam Centraal: **X ≈ 121.000 m, Y ≈ 487.000 m**

```python
# Controleer CRS na laden van PDOK-data
print(gdf.crs)  # -> EPSG:28992

# Oppervlakte berekenen in EPSG:28992 (in m²)
gdf['opp_m2'] = gdf.geometry.area
gdf['opp_km2'] = gdf['opp_m2'] / 1_000_000
```

---

### EPSG:4326 — WGS84 (breedte- en lengtegraad)

- **Wat**: Wereldwijd coördinatenstelsel op basis van breedte- en lengtegraden
- **Eenheden**: Graden (latitude/longitude)
- **Vervorming**: Aanzienlijk voor kleine-schaalkaarten (Nederland wordt iets breder dan het echt is bij hoge zoom)
- **Gebruik**: Verplicht voor Folium, Plotly Mapbox, Leaflet. Cartomap/nl-bestanden zijn al in WGS84.

Voorbeeld coördinaten Amsterdam Centraal: **lat ≈ 52.378°, lon ≈ 4.900°**

```python
# Converteer naar WGS84 voor webkaarten
gdf_wgs84 = gdf.to_crs(epsg=4326)
print(gdf_wgs84.crs)  # -> EPSG:4326
```

---

### EPSG:3857 — Web Mercator (Pseudo-Mercator)

- **Wat**: Het stelsel dat Google Maps, OpenStreetMap en vrijwel alle webtegels gebruiken intern
- **Eenheden**: Meters (maar niet nauwkeurig voor afstandsberekeningen)
- **Vervorming**: Sterk boven circa 60° breedte — niet geschikt voor kaartweergave van NL
- **Gebruik**: Je hoeft hier zelden expliciet naar te converteren. Folium, Plotly en Leaflet regelen dit intern. Gebruik het alleen als een specifieke bibliotheek het expliciet vereist.

```python
# Zelden nodig — alleen als een bibliotheek dit expliciet vereist
gdf_mercator = gdf.to_crs(epsg=3857)
```

---

## Beslisboom: welk CRS gebruik je?

```
Wat ga je maken?
│
├── Statische kaart met geopandas (.plot())
│   └── Gebruik EPSG:28992 (RD New) — minste vervorming voor NL
│
├── Interactieve kaart met Folium
│   └── Converteer naar EPSG:4326 (WGS84) — Folium vereist dit
│
├── Interactieve kaart met Plotly Mapbox
│   └── Converteer naar EPSG:4326 (WGS84) — Plotly vereist dit
│
├── Cartomap/nl GeoJSON-bestand geladen
│   └── Al in EPSG:4326 — geen conversie nodig voor Folium/Plotly
│
└── Oppervlakte- of afstandsberekeningen
    └── Gebruik EPSG:28992 of EPSG:3857 — niet EPSG:4326 (graden ≠ meters)
```

---

## CRS-conversie — code

```python
import geopandas as gpd

# Controleer huidig CRS
print(gdf.crs)

# Converteren
gdf_wgs84 = gdf.to_crs(epsg=4326)       # Naar WGS84 (voor webkaarten)
gdf_rd = gdf.to_crs(epsg=28992)          # Naar RD New (voor statische kaarten)
gdf_mercator = gdf.to_crs(epsg=3857)     # Naar Web Mercator (zelden nodig)

# CRS instellen als het ontbreekt (bijv. bij handmatig aangemaakte geodata)
gdf = gdf.set_crs(epsg=28992)            # Stel in zonder te converteren
```

---

## Veelgemaakte fouten

### Fout 1: Kaart is leeg of verschoven

**Oorzaak**: CRS van geodata en statistisch data komen niet overeen, of bibliotheek verwacht WGS84 maar krijgt RD New.

```python
# Diagnose: controleer CRS voor samenvoegen
print(f"Geodata CRS: {geo.crs}")
print(f"Na conversie: {geo.to_crs(epsg=4326).crs}")
```

### Fout 2: Oppervlakte berekend in graden i.p.v. meters

```python
# FOUT: oppervlakte in graden (zinloos)
gdf_wgs84['opp'] = gdf_wgs84.geometry.area  # Niet doen!

# JUIST: eerst converteren naar een metrisch stelsel
gdf_metrisch = gdf.to_crs(epsg=28992)
gdf_metrisch['opp_km2'] = gdf_metrisch.geometry.area / 1_000_000
```

### Fout 3: CRS ontbreekt na lezen van GeoJSON

```python
if gdf.crs is None:
    # PDOK-data is bijna altijd EPSG:28992
    # Cartomap/nl is altijd EPSG:4326
    gdf = gdf.set_crs(epsg=28992)  # Of 4326 voor cartomap/nl
    print(f"CRS handmatig ingesteld op: {gdf.crs}")
```

---

## Snelreferentie

| Stap | CRS |
|------|-----|
| PDOK WFS laden | EPSG:28992 (automatisch) |
| Cartomap/nl laden | EPSG:4326 (automatisch) |
| Statische kaart renderen (geopandas) | EPSG:28992 behouden |
| Folium-kaart renderen | Converteren naar EPSG:4326 |
| Plotly Mapbox renderen | Converteren naar EPSG:4326 |
| Oppervlakteberekening | EPSG:28992 of EPSG:3857 |
