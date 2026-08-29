# Wisconsin Groundwater Model

An interactive 3D model of Wisconsin's water table, built from static water level
records for 1,690 municipal wells held by the Wisconsin DNR, 2013 to 2025.

**Live viewer: https://cash06.github.io/wi-groundwater/**

`index.html` is fully self contained. Both datasets are inlined; the only external
request is three.js from cdnjs. It can be opened straight from disk.

## What it shows

Three aquifer systems, interpolated separately and stacked beneath a terrain
surface built from the 10 m National Elevation Dataset:

| Band | Wells | Median depth to water |
|---|---|---|
| Shallow, under 150 ft | 505 | 22 ft |
| Intermediate, 150 to 600 ft | 750 | 44 ft |
| Deep, 600 ft and over | 413 | 138 ft |

Every well also carries a letter grade from the cleaning pipeline, and the wells
can be filtered or recoloured by that grade.

## Method

The interpolated quantity is **head**, the water table elevation above sea level,
not depth below ground. Head is a smooth physical field; depth is not, because it
jumps with topography. Depth is recovered in the viewer as the gap between the
terrain and the water surface.

Interpolating head directly puts water above ground in 25% of shallow cells, which
is impossible. The water table is a subdued replica of topography, so head is first
fitted against land surface elevation within each band, and only the residual is
interpolated by inverse distance and added back. That drops the impossible cells to
0.0%.

The fitted slope *b* is itself a result. It measures how closely each system tracks
the land above it:

| Band | slope *b* | R² | residual sd |
|---|---|---|---|
| Shallow | 1.004 | 0.995 | 18 ft |
| Intermediate | 0.982 | 0.926 | 47 ft |
| Deep | 0.674 | 0.362 | 130 ft |

A slope near 1.0 means the water table drapes the landscape, an unconfined aquifer.
The deep system's 0.674 means it is decoupled from local topography, which is what
confinement is. The depth bands turn out to be a real confinement gradient rather
than an arbitrary cut.

## Known limitation

**Well positions are provisional.** 1,682 of the 1,690 wells are located only to
their PLSS section, a one square mile block whose internal relief averages 94 ft.
Coordinates come from the DNR field `CALC_LL_LAT_DD_AMT`, a section centroid, so
948 wells share a coordinate with at least one other well and one point carries 14.

Read this as a regional surface. It is not valid for site specific work, and not
valid for anything distance weighted such as well interference, where the true
separation between two wells at the same plotted point is unknown.

The residual sd shown for each layer in the viewer is the honest error bar and
should fall once surveyed coordinates replace the centroids.

## Sources

- Static water levels, well construction, casing and geology: Wisconsin DNR
- Land surface elevation: NED 10 m, resampled to 50 m
- County boundaries: US Census TIGER
- City positions: US Census 2023 Gazetteer, ranked by 2020 population

## Rebuilding

Generated from the analysis repo by three scripts, run in order:

```
python3 src/build_water_table_model.py   # wells, surfaces, grades
python3 src/build_reference_geo.py       # county boundaries, cities
python3 src/build_viewer.py              # writes site/index.html
```

Well coordinates are resolved in exactly one function, `load_coords()`. Dropping a
`data/well_coords_official.csv` with `well,lat,lon` columns overrides the centroids
per well and the whole model rebuilds against real positions with no other edits.
