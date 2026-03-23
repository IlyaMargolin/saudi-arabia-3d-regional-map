# Google Colab Usage Guide

This guide explains how to reproduce the 3D regional map of Saudi Arabia in Google Colab using the files included in this repository.

## Required input files

Before starting, make sure you have the following file available on your computer:

- `saudi_regions_13.geojson`

The CSV file can either be created directly in Colab using the provided code or created inside Colab using the instructions below.

## Step 1. Open Google Colab

Open Google Colab and create a new notebook.

## Step 2. Install the required libraries

Create a new code cell and paste:

~~~python
!pip install pandas numpy geopandas shapely pydeck mapclassify fiona --quiet
~~~

Run the cell.

## Step 3. Upload the regional geometry file

Create a new code cell and paste:

~~~python
from google.colab import files
uploaded = files.upload()
~~~

Run the cell and upload:

- `saudi_regions_13.geojson`

## Step 4. Create the CSV dataset directly in Colab

Create a new code cell and paste:

~~~python
import pandas as pd

data = [
    ["Riyadh", 92, "leading", 4.1, 68.3, 81.2, "Composite development score (test data)", "2026-03-20"],
    ["Makkah", 88, "advanced", 5.0, 66.1, 78.4, "Composite development score (test data)", "2026-03-20"],
    ["Al Madinah", 79, "advanced", 5.6, 63.8, 72.5, "Composite development score (test data)", "2026-03-20"],
    ["Al Qassim", 74, "moderate", 6.1, 61.5, 69.8, "Composite development score (test data)", "2026-03-20"],
    ["Eastern Region", 90, "leading", 4.3, 67.9, 80.5, "Composite development score (test data)", "2026-03-20"],
    ["Asir", 70, "moderate", 6.8, 59.7, 66.2, "Composite development score (test data)", "2026-03-20"],
    ["Tabuk", 68, "moderate", 7.2, 58.9, 64.7, "Composite development score (test data)", "2026-03-20"],
    ["Hail", 61, "lagging", 8.0, 55.3, 60.2, "Composite development score (test data)", "2026-03-20"],
    ["Northern Borders", 58, "lagging", 8.6, 54.1, 58.8, "Composite development score (test data)", "2026-03-20"],
    ["Jazan", 63, "lagging", 7.7, 56.4, 61.0, "Composite development score (test data)", "2026-03-20"],
    ["Najran", 60, "lagging", 8.1, 55.0, 59.4, "Composite development score (test data)", "2026-03-20"],
    ["Al Bahah", 57, "lagging", 8.9, 53.6, 57.9, "Composite development score (test data)", "2026-03-20"],
    ["Al Jouf", 64, "moderate", 7.5, 57.1, 62.4, "Composite development score (test data)", "2026-03-20"],
]

columns = [
    "region_en",
    "score",
    "tier",
    "unemployment_rate",
    "labor_force_participation",
    "economic_activity_index",
    "source_note",
    "as_of_date"
]

df = pd.DataFrame(data, columns=columns)
df.to_csv("saudi_regions_data.csv", index=False)

print("File created: saudi_regions_data.csv")
df
~~~

Run the cell.

## Step 5. Build the 3D map

Create a new code cell and paste:

~~~python
import json
import re

import geopandas as gpd
import pandas as pd
import pydeck as pdk

GEOJSON_PATH = "saudi_regions_13.geojson"
CSV_PATH = "saudi_regions_data.csv"
OUTPUT_HTML = "saudi_3d_development_map.html"

REGION_SYNONYMS = {
    "riyadh": "Riyadh",
    "ar riyad": "Riyadh",
    "ar-riyad": "Riyadh",

    "makkah": "Makkah",
    "mecca": "Makkah",
    "makkah al mukarramah": "Makkah",
    "makkah region": "Makkah",

    "al madinah": "Al Madinah",
    "madinah": "Al Madinah",
    "medina": "Al Madinah",
    "al madinah al munawwarah": "Al Madinah",

    "al qassim": "Al Qassim",
    "qassim": "Al Qassim",
    "qaseem": "Al Qassim",
    "al qaseem": "Al Qassim",
    "al-qassim": "Al Qassim",
    "al-qaseem": "Al Qassim",

    "eastern region": "Eastern Region",
    "eastern province": "Eastern Region",
    "eastern": "Eastern Region",
    "ash sharqiyah": "Eastern Region",
    "sharqiyah": "Eastern Region",

    "asir": "Asir",
    "aseer": "Asir",
    "asir region": "Asir",
    "aseer region": "Asir",

    "tabuk": "Tabuk",

    "hail": "Hail",
    "ha'il": "Hail",
    "hail region": "Hail",

    "northern borders": "Northern Borders",
    "northern border": "Northern Borders",
    "al hudud ash shamaliyah": "Northern Borders",
    "northern borders region": "Northern Borders",

    "jazan": "Jazan",
    "jizan": "Jazan",
    "jazan region": "Jazan",
    "jizan region": "Jazan",

    "najran": "Najran",

    "al bahah": "Al Bahah",
    "al baha": "Al Bahah",
    "bahah": "Al Bahah",
    "baha": "Al Bahah",

    "al jouf": "Al Jouf",
    "al jawf": "Al Jouf",
    "jouf": "Al Jouf",
    "jawf": "Al Jouf",
}

TIER_COLORS = {
    "leading": [0, 191, 140, 235],
    "advanced": [59, 130, 246, 235],
    "moderate": [245, 158, 11, 235],
    "lagging": [239, 68, 68, 235],
}

def normalize_name(value):
    if value is None:
        return ""
    value = str(value).strip().lower()
    value = re.sub(r"[_\\-]+", " ", value)
    value = re.sub(r"\\s+", " ", value)
    return REGION_SYNONYMS.get(value, value.title())

def detect_name_column(gdf):
    candidates = [
        "region", "Region", "name", "Name", "NAME_1", "NAME",
        "admin1Name_en", "shapeName", "province", "Province",
        "ADM1_EN", "reg_name"
    ]
    for c in candidates:
        if c in gdf.columns:
            return c

    for c in gdf.columns:
        if c != "geometry" and str(gdf[c].dtype) == "object":
            return c

    raise ValueError("Could not detect a region-name column in the GeoJSON file.")

def score_to_tier(score):
    if pd.isna(score):
        return "moderate"
    if score >= 85:
        return "leading"
    if score >= 72:
        return "advanced"
    if score >= 60:
        return "moderate"
    return "lagging"

gdf = gpd.read_file(GEOJSON_PATH)
name_col = detect_name_column(gdf)
gdf["region_en"] = gdf[name_col].apply(normalize_name)

df = pd.read_csv(CSV_PATH)
df["region_en"] = df["region_en"].apply(normalize_name)

numeric_cols = [
    "score",
    "unemployment_rate",
    "labor_force_participation",
    "economic_activity_index"
]

for c in numeric_cols:
    if c in df.columns:
        df[c] = pd.to_numeric(df[c], errors="coerce")

if "tier" not in df.columns:
    df["tier"] = df["score"].apply(score_to_tier)
else:
    df["tier"] = df["tier"].fillna(df["score"].apply(score_to_tier))

merged = gdf.merge(df, on="region_en", how="left")

fallback_score = df["score"].median()
fallback_unemployment = df["unemployment_rate"].median()
fallback_lfp = df["labor_force_participation"].median()
fallback_econ = df["economic_activity_index"].median()

merged["score"] = merged["score"].fillna(fallback_score)
merged["unemployment_rate"] = merged["unemployment_rate"].fillna(fallback_unemployment)
merged["labor_force_participation"] = merged["labor_force_participation"].fillna(fallback_lfp)
merged["economic_activity_index"] = merged["economic_activity_index"].fillna(fallback_econ)
merged["source_note"] = merged["source_note"].fillna("Filled automatically to avoid empty map regions")
merged["as_of_date"] = merged["as_of_date"].fillna("2026-03-20")

merged["tier"] = merged["tier"].fillna(merged["score"].apply(score_to_tier))
merged["fill_color"] = merged["tier"].map(TIER_COLORS)

min_s = merged["score"].min()
max_s = merged["score"].max()

if min_s == max_s:
    merged["elevation"] = 90000
else:
    merged["elevation"] = merged["score"].apply(
        lambda x: float(40000 + (x - min_s) / (max_s - min_s) * 180000)
    )

merged_wgs84 = merged.to_crs(epsg=4326)

for col in merged_wgs84.columns:
    if col != "geometry":
        if "datetime" in str(merged_wgs84[col].dtype):
            merged_wgs84[col] = merged_wgs84[col].astype(str)

for col in merged_wgs84.columns:
    if col != "geometry":
        merged_wgs84[col] = merged_wgs84[col].where(merged_wgs84[col].notna(), None)

center = merged_wgs84.union_all().centroid
geojson_data = json.loads(merged_wgs84.to_json())

layer = pdk.Layer(
    "GeoJsonLayer",
    data=geojson_data,
    pickable=True,
    stroked=True,
    filled=True,
    extruded=True,
    wireframe=False,
    get_fill_color="properties.fill_color",
    get_line_color=[255, 255, 255, 120],
    line_width_min_pixels=2,
    get_elevation="properties.elevation",
    elevation_scale=1,
    auto_highlight=True,
    opacity=0.92,
)

view_state = pdk.ViewState(
    latitude=center.y,
    longitude=center.x,
    zoom=4.8,
    pitch=42,
    bearing=0,
)

tooltip = {
    "html": \"""
    <div style="padding:8px 10px;">
        <div style="font-size:16px; font-weight:700; margin-bottom:6px;">{region_en}</div>
        <div><b>Development score:</b> {score}</div>
        <div><b>Tier:</b> {tier}</div>
        <div><b>Unemployment rate:</b> {unemployment_rate}%</div>
        <div><b>Labor force participation:</b> {labor_force_participation}%</div>
        <div><b>Economic activity index:</b> {economic_activity_index}</div>
        <div><b>As of:</b> {as_of_date}</div>
    </div>
    \""",
    "style": {
        "backgroundColor": "rgba(12,18,28,0.95)",
        "color": "white",
        "fontSize": "13px",
        "borderRadius": "8px"
    }
}

deck = pdk.Deck(
    layers=[layer],
    initial_view_state=view_state,
    tooltip=tooltip,
    map_style="mapbox://styles/mapbox/dark-v11",
)

deck.show()
deck.to_html(OUTPUT_HTML, notebook_display=False)

print(f"Map saved to: {OUTPUT_HTML}")
~~~

Run the cell.

## Step 6. Download the generated HTML map

Create a new code cell and paste:

~~~python
from google.colab import files
files.download("saudi_3d_development_map.html")
~~~

Run the cell.

## Expected output

At the end of the workflow, you should have:

- an interactive 3D regional map displayed in Colab;
- a generated file named `saudi_3d_development_map.html` available for download.

## Notes

The current CSV creation block uses the same prototype values as the current repository version. Users may replace these values with another dataset if needed.
