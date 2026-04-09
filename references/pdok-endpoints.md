# PDOK Endpoints Reference

Complete reference for fetching Dutch geographic boundary files from PDOK.

---

## 1. WFS URL template — nieuw formaat (2023–heden)

Vanaf 2023 gebruikt PDOK een nieuw endpoint-formaat. Gebruik **altijd** het nieuwe formaat.

```
https://service.pdok.nl/cbs/gebiedsindelingen/{JAAR}/wfs/v1_0
  ?request=GetFeature
  &service=WFS
  &version=2.0.0
  &typeName={LAAG}
  &outputFormat=json
```

### Variabelen

| Variabele | Opties | Toelichting |
|-----------|--------|-------------|
| `{JAAR}` | 2013–2025 | Jaar van de gebiedsindeling; gebruik het jaar van je statistische data |
| `{LAAG}` | zie onderstaande tabel | Geef voorkeur aan `_gegeneraliseerd`-varianten |

### Beschikbare lagen

| Geografisch niveau | Laag (gegeneraliseerd) | Laag (exact) | Typisch aantal polygonen |
|--------------------|----------------------|-------------|--------------------------|
| Gemeente | `gemeente_gegeneraliseerd` | `gemeente` | ~340 |
| Wijk | `wijk_gegeneraliseerd` | `wijk` | ~2.500 |
| Buurt | `buurt_gegeneraliseerd` | `buurt` | ~13.000 |
| Corop | `corop_gegeneraliseerd` | `corop` | 40 |
| Provincie | `provincie_gegeneraliseerd` | `provincie` | 12 |

**Gebruik altijd de `_gegeneraliseerd`-variant voor kaarten** — vereenvoudigde geometrie, veel kleinere bestandsgrootte, aanzienlijk snellere rendering.

### Python-code

```python
import geopandas as gpd

def haal_pdok_grenzen(niveau: str, jaar: int = 2024) -> gpd.GeoDataFrame:
    """
    Haal gebiedsgrenzen op van PDOK WFS.
    
    Parameters
    ----------
    niveau : str — 'gemeente', 'wijk', 'buurt', 'corop', 'provincie'
    jaar : int   — Jaar van gebiedsindeling (2013–2025)
    
    Returns
    -------
    GeoDataFrame in EPSG:28992 (RD New)
    """
    laag = f"{niveau}_gegeneraliseerd"
    url = (
        f"https://service.pdok.nl/cbs/gebiedsindelingen/{jaar}/wfs/v1_0"
        f"?request=GetFeature&service=WFS&version=2.0.0"
        f"&typeName={laag}&outputFormat=json"
    )
    print(f"Laden van PDOK: {url}")
    gdf = gpd.read_file(url)
    print(f"Geladen: {len(gdf)} gebieden | CRS: {gdf.crs}")
    return gdf

# Gebruik:
gemeenten = haal_pdok_grenzen('gemeente', jaar=2024)
```

---

## 2. Oud PDOK-formaat (vóór 2023 — niet meer aanbevolen)

Het oude endpoint (`geodata.nationaalgeoregister.nl`) is vervangen. Gebruik het niet voor nieuwe projecten. Als je toch met oude code werkt:

```
# OUD FORMAAT — niet meer gebruiken
https://geodata.nationaalgeoregister.nl/cbsgebiedsindelingen/wfs
  ?request=GetFeature
  &service=WFS
  &version=2.0.0
  &typeName=cbsgebiedsindelingen:cbs_gemeente_2022_gegeneraliseerd
  &outputFormat=json
```

---

## 3. Veldnamen映射 (field name mapping)

| Veld | Inhoud | Voorbeeld |
|------|--------|-----------|
| `statcode` | Regiocode (CBS-formaat, met spaties gestript) | `GM0363` |
| `statnaam` | Regionaam in klare tekst | `Utrecht` |
| `jrstatcode` | Jaar-specifieke code | `GM0363_2024` |
| `rubriek` | Categorie van het gebied | `gemeente` |
| `geometry` | Polygoongeometrie (EPSG:28992) | — |

**Let op**: `statcode` kan in sommige exportjaren spaties bevatten. Altijd strippen:

```python
gdf['statcode'] = gdf['statcode'].str.strip()
```

---

## 4. Cartomap/nl — fallback bij PDOK-timeout

Bij PDOK-timeouts (veelvoorkomend bij buurt-niveau met 13.000+ polygonen) of als je geen live verbinding wilt:

```python
# Basis-URL patroon:
# https://cartomap.github.io/nl/wgs84/{niveau}_{jaar}.geojson

def haal_cartomap_grenzen(niveau: str, jaar: int = 2024) -> gpd.GeoDataFrame:
    """
    Haal vereenvoudigde grenzen op van cartomap/nl (GitHub).
    Bestanden zijn al in WGS84 (EPSG:4326) — geen CRS-conversie nodig voor webkaarten.
    """
    url = f"https://cartomap.github.io/nl/wgs84/{niveau}_{jaar}.geojson"
    print(f"Laden van cartomap/nl: {url}")
    gdf = gpd.read_file(url)
    print(f"Geladen: {len(gdf)} gebieden | CRS: {gdf.crs}")
    return gdf

# Gebruik:
gemeenten_wgs84 = haal_cartomap_grenzen('gemeente', jaar=2024)
# Beschikbare niveaus: gemeente, wijk, buurt
# Beschikbare jaren: varieert; controleer https://github.com/cartomap/nl
```

**Veldnamen in cartomap/nl** kunnen afwijken van PDOK. Controleer met `gdf.columns` welk veld de regiocode bevat (vaak `statcode` of `GM_CODE`).

---

## 5. Beschikbare jaren

| Bron | Beschikbare jaren | Opmerking |
|------|-------------------|-----------|
| PDOK nieuw formaat | 2015–2025 | Sommige jaren in ontwikkeling |
| PDOK oud formaat | 2012–2022 | Niet aanbevolen voor nieuw gebruik |
| cartomap/nl | 2012–2024 | Controleer GitHub voor nieuwste beschikbaarheid |

### Welk jaar kiezen?

Gebruik altijd het **gebiedsindelingsjaar dat overeenkomt met het jaar van je statistische data**. Als je CBS-data uit 2022 hebt, gebruik dan `jaar=2022` voor de geodata.

**Reden**: Nederland fuseer gemeenten regelmatig (herindelingen). GM-codes van gefuseerde gemeenten bestaan niet meer in latere geodata-jaren.

```python
# Richtlijn:
stat_jaar = 2022  # Het jaar van je CBS-data
geo_jaar = stat_jaar  # Zelfde jaar!
```

---

## 6. Rate limits en foutafhandeling

PDOK WFS heeft geen authenticatie nodig, maar heeft beperkingen:

- **Buurt-niveau**: kan 10–30 seconden duren (13.000+ polygonen); gebruik de fallback als het te lang duurt
- **WFS paginering**: standaard worden 1.000 objecten teruggegeven. Alle lagen zijn normaal gesproken kleiner, maar gebruik bij twijfel `&count=100000` om paginering te voorkomen
- **Timeout**: stel een timeout in bij geopandas:

```python
import geopandas as gpd
import requests

# Verhoog timeout voor grote buurt-datasets
from fiona.env import Env
with Env(GDAL_HTTP_TIMEOUT=60):
    geo = gpd.read_file(wfs_url)
```

### Robuust laden met fallback

```python
def haal_grenzen_met_fallback(niveau: str, jaar: int = 2024) -> gpd.GeoDataFrame:
    """Probeer PDOK; val terug op cartomap/nl bij fout of timeout."""
    import time

    pdok_url = (
        f"https://service.pdok.nl/cbs/gebiedsindelingen/{jaar}/wfs/v1_0"
        f"?request=GetFeature&service=WFS&version=2.0.0"
        f"&typeName={niveau}_gegeneraliseerd&outputFormat=json"
    )

    try:
        print("Proberen: PDOK WFS...")
        start = time.time()
        gdf = gpd.read_file(pdok_url)
        print(f"PDOK geslaagd in {time.time()-start:.1f}s ({len(gdf)} gebieden)")
        return gdf
    except Exception as e:
        print(f"PDOK mislukt: {e}")
        print("Terugvallen op cartomap/nl...")
        fallback_url = f"https://cartomap.github.io/nl/wgs84/{niveau}_{jaar}.geojson"
        gdf = gpd.read_file(fallback_url)
        print(f"cartomap/nl geslaagd ({len(gdf)} gebieden, CRS: {gdf.crs})")
        return gdf
```
