# Methodology: Project 8 — Terrain Analysis and Riparian Area Management in the Nahmint Valley

This document contains the detailed step-by-step methodology followed in Project 8.

---

## Task 1: Prepare the DEM and Map Stream Networks
- Exported the `nahmint_dem` PostGIS raster from QGIS as a GeoTiff
- Applied the Fill tool in ArcGIS Pro to remove sinks from the DEM, producing `Nahmint_fill`
- Ran the Flow Direction tool (D8 method) on the filled DEM to produce `Nahmint_FlowDir`
- Ran the Flow Accumulation tool (integer output, D8) to produce `FlowAcc`
- Experimented with flow accumulation thresholds to identify a stream network consistent with the satellite basemap and shaded relief DEM
- Used the Reclassify tool to assign a value of 1 to cells above the selected threshold and NODATA to all others, producing `Stream_reclass`
- Applied the Stream Order tool (Strahler method) to produce `StreamOrder`, and repeated using the Shreve method for comparison
- Converted the stream order raster to polyline features using Stream to Feature with simplified polylines, producing `StreamNetwork_polyline`
- Generated unique stream segment IDs using the Stream Link tool, producing `StreamLinks`

---

## Task 2: Extract Stream Characteristics
- Multiplied `Stream_reclass` by `Nahmint_DEM` using the Raster Calculator to extract elevation within stream cells, producing `streams_elevation`
- Used the Zonal Statistics tool with `StreamLinks` as zones to calculate maximum elevation change per stream segment
- Converted the elevation change raster to polygons using Raster to Polygon, producing `elevation_change_polygon`
- Performed a Spatial Join (largest overlap, 3 m search radius) between `StreamNetwork_polyline` and `elevation_change_polygon`, producing `StreamNetwork_join`
- Calculated a `PercentGradient` field using elevation change divided by segment length multiplied by 100
- Measured stream widths manually from the ArcGIS satellite basemap for 10–15 samples per stream order and calculated averages
- Assigned average stream widths per order to a new `StreamWidth` field using a Python `def reclass()` function in the Field Calculator

---

## Task 3: Mapping Watersheds and Riparian Management Areas
- Created a `Reserve_BufferDist` field and used a Python conditional function combining gradient and width to assign reserve zone buffer distances based on BC stream class (S2–S6)
- Repeated the process for a management zone buffer distance field
- Applied the Buffer tool to `StreamNetwork_join` using the `Reserve_BufferDist` field, with full sides, round ends, and dissolved output, producing `Reserve_buffer`
- Repeated the Buffer tool for the management zone distances
- Ran the Watershed tool using `Nahmint_FlowDir` and `StreamLinks` to delineate watershed catchments, producing `Nahmint_watersheds`
- Generated 100 m interval contour lines from `Nahmint_DEM` using the Contour tool, producing `Nahmint_contour`
- Produced a final map layout including watersheds, contour lines, stream network symbolized by class, an inset map of example RMAs, and summary text for total reserve/management area and stream network length

---

*Back to [README](README_project8.md)*
