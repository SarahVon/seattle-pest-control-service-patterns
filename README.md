# Seattle Pest Control Service Patterns (2023)

**Published results:** [Open the interactive report](https://01a0bc03-40b2-ca8c-6045-21517970ca76.share.connect.posit.cloud/)  
**Public source:** [`seattle-pest-service-patterns.Rmd`](seattle-pest-service-patterns.Rmd)

This analysis uses a weighted bipartite network to compare pest-service activity across broad pest categories, seven Seattle regions, and seasons. A service record indicates a requested or performed service, not a confirmed sighting or prevalence estimate.

## Purpose

I created this analysis to make category–region relationships easier to inspect than a table of records and to compare how those relationships change across winter, spring, summer, and fall.

## Setup

Install R and the packages used by the public source:

```r
install.packages(c(
  "bipartiteD3", "bipartite", "readxl", "r2d3", "tidyverse",
  "RColorBrewer", "sf", "rnaturalearth", "rnaturalearthdata",
  "patchwork", "writexl", "tidygeocoder", "lubridate", "printr"
))
```

## Data and privacy boundary

The source workbook contains proprietary service information, including dates, technician-entered targets, addresses, and geocoding coordinates. Raw and prepared workbooks, record-level rows, addresses, names, coordinates, neighborhood files, and rendered HTML with embedded chart payloads are intentionally excluded. The public Rmd resolves private inputs through `SEATTLE_PEST_DATA_DIR` and does not publish private values.

## Workflow

1. Read the authorized workbook and retain Seattle records.
2. Parse dates and assign winter, spring, summer, or fall.
3. Use prepared coordinates and join service points to Seattle neighborhood boundaries.
4. Consolidate technician-entered labels and neighborhoods into analysis groups.
5. Aggregate category–region counts for the full year and each season.
6. Render interactive bipartite graphs with `bipartiteD3`.

Related pest labels are grouped into Ants, Rodents, Cockroaches, Biting Insects, Flying Insects, Stinging Insects, Storage and Structure Pests, Spiders, and Misc Crawling Insects. Regions are project-defined groupings, not official administrative units.

## Results

The published report contains full-year and seasonal graphs. In the source summary, rodents represented **46.78%** of annual records, ants **31.23%**, and cockroaches **15.74%**. Rodents led in winter (**54.58%**), spring (**46.52%**), and fall (**47.21%**); ants led in summer (**40.45%**). Cockroach activity remained especially visible in Downtown and South/SE Seattle.

These percentages describe service records in the selected period, not citywide pest prevalence. Interpretation is limited by service demand, repeat visits, customer mix, inconsistent labels, geocoding and spatial-join quality, project-defined regions, and the single-company 2023 sample.

## Reproducibility

Obtain authorized copies of the private workbooks and complete neighborhood shapefile separately, place them outside version control, set `SEATTLE_PEST_DATA_DIR`, and render:

```r
rmarkdown::render("seattle-pest-service-patterns.Rmd")
```

Reproduction requires access to proprietary inputs but does not require committing them or generated HTML.

## Repository contents

- `seattle-pest-service-patterns.Rmd` — public workflow and setup
- `README.md` — methods, results, privacy boundary, and reproduction guidance
- `.gitignore` — safeguards for private and generated files
