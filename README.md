# Seattle Pest Control Service Patterns (2023)

**Published results:** [open the interactive report on Posit Connect Cloud](https://01a0bc03-40b2-ca8c-6045-21517970ca76.share.connect.posit.cloud/)
**Public source:** [`seattle-pest-service-patterns.Rmd`](seattle-pest-service-patterns.Rmd)

This analysis describes how a Seattle pest-control company's 2023 service activity varied by broad pest category, grouped region, and season. It uses a **two-mode (bipartite) network**: one node set is pest categories, the other is seven Seattle analysis regions, and weighted edges count service records connecting them.

## Contents

- [Purpose and research question](#purpose-and-research-question)
- [Setup and packages](#setup-and-packages)
- [Data and privacy boundary](#data-and-privacy-boundary)
- [Import and preparation workflow](#import-and-preparation-workflow)
- [Cleaning and preparation](#cleaning-and-preparation)
- [Pest-category grouping](#pest-category-grouping)
- [Summary calculations](#summary-calculations)
- [Neighborhood and region grouping](#neighborhood-and-region-grouping)
- [Bipartite graph construction](#bipartite-graph-construction)
- [Full-year and seasonal results](#full-year-and-seasonal-results)
- [Conclusion and limitations](#conclusion-and-limitations)
- [Reproducibility](#reproducibility)

## Purpose and research question

The goal is to make service-pattern relationships easier to inspect than a long list of individual records. The analysis asks:

1. Which broad pest categories account for the most service activity?
2. How do category–region relationships differ across Seattle?
3. How do those relationships change between winter, spring, summer, and fall?
4. What does an interactive bipartite graph reveal that category or region totals alone do not?

A service record indicates that a pest-control service was requested or performed. It is **not necessarily a confirmed sighting**, population estimate, or causal observation.

## Setup and packages

The public R Markdown source uses a private-input directory rather than an absolute machine-specific path. Install R and the packages below (and the system dependencies required by `sf`):

```r
install.packages(c(
  "bipartiteD3", "bipartite", "readxl", "r2d3", "tidyverse",
  "RColorBrewer", "sf", "rnaturalearth", "rnaturalearthdata",
  "patchwork", "writexl", "tidygeocoder", "lubridate", "printr"
))
```

The main roles are `readxl` for workbook import, `lubridate` and `tidyverse` for preparation and summaries, `sf` for spatial joins, `tidygeocoder` for the original geocoding workflow, and `bipartiteD3` for interactive network charts. `bipartite`, `r2d3`, and `RColorBrewer` support network analysis, D3 rendering, and node colors.

## Data and privacy boundary

The source table contains proprietary service information collected by a local pest-control company. The original records include fields such as service dates, manually entered pest targets, addresses, and geocoding coordinates; operational records may also contain customer or business details. Those records are not published here.

This repository intentionally excludes:

- raw or prepared Excel workbooks;
- addresses, names, customer/business details, coordinates, or record-level rows;
- the Seattle neighborhood shapefile and its sidecar files;
- rendered self-contained HTML whose embedded interactive payload could expose generated chart data;
- `.RData`, history, deployment, or `rsconnect` files.

The public Rmd documents the method with private file names resolved through `SEATTLE_PEST_DATA_DIR`; it does not print private values or record-level data. Do not treat the public repository as a data release.

## Import and preparation workflow

The complete workflow is:

1. Read the private service workbook and retain records whose address is in Seattle.
2. Parse the service date and derive a season: winter (December–February), spring (March–May), summer (June–August), or fall (September–November).
3. Use cached latitude and longitude prepared from the service address. Geocoding is documented but is not rerun during ordinary rendering.
4. Transform Seattle neighborhood boundaries from the source CRS (EPSG:2926) to WGS84 (EPSG:4326).
5. Convert service points to an `sf` object and spatially join points within neighborhood polygons.
6. Collapse technician-entered pest labels and neighborhood names into analysis categories.
7. Read the authorized prepared analysis table and aggregate category–region counts for full-year and seasonal graphs.

The public Rmd uses paths of this form, with no private path committed:

```r
data_dir <- Sys.getenv("SEATTLE_PEST_DATA_DIR", "data-private")
raw_file <- file.path(data_dir, "sea_pests.xlsx")
prepared_file <- file.path(data_dir, "seattle_pests_2023.xlsx")
```

## Cleaning and preparation

Dates are converted with `lubridate::mdy()`. Seasons are assigned from the resulting month, and missing-value checks are run before spatial processing. The analysis retains the fields needed for the network: date, season, grouped target, and grouped region. Geometry is dropped before the final count table is built.

The source also records the original scale of the import: more than 30,000 service entries were present before the Seattle filter. Exact row-level outputs are intentionally not reproduced in this README.

## Pest-category grouping

Technician-entered targets were not standardized, so related labels were consolidated with `dplyr::case_when()`. The broad categories are:

| Analysis category | Examples consolidated |
| --- | --- |
| Ants | carpenter, nuisance, pavement, odorous, moisture, and general ants |
| Rodents | mice, rats, vertebrate, and general rodent labels |
| Cockroaches | cockroach labels retained as a category |
| Biting Insects | bed bugs, fleas, and mites |
| Flying Insects | house flies and other non-stinging flying-insect labels |
| Stinging Insects | wasps, hornets, and related labels |
| Storage and Structure Pests | fabric pests, food-storage pests, beetles, and termites |
| Spiders | spider labels retained as a category |
| Misc Crawling Insects | other labels not mapped into the principal groups |

This grouping improves readability but trades away species- and label-level detail.

## Summary calculations

For each time period, the analysis counts records by `Target` and `Neighborhood`:

```r
counts <- data |>
  dplyr::count(Target, Neighborhood, name = "Sightings")

matrix <- counts |>
  tidyr::pivot_wider(
    names_from = Neighborhood,
    values_from = Sightings,
    values_fill = 0
  ) |>
  tibble::column_to_rownames("Target") |>
  as.matrix()
```

Category and region totals are calculated with `count()`/`summarise(n())`, sorted descending, and expressed as percentages of the selected period. The graph's percentages therefore describe the distribution of service records in that period—not the prevalence of pests in Seattle.

## Neighborhood and region grouping

Service points are joined to the [Seattle City GIS neighborhood boundaries](https://data-seattlecitygis.opendata.arcgis.com/datasets/b4a142f592e94d39a3bf787f3c112c1d/explore). To keep the network legible, sub-neighborhoods are mapped into seven analysis regions:

- **NW Seattle**
- **NE Seattle**
- **Magnolia/Queen Anne**
- **Central Seattle**
- **Downtown Seattle**
- **West Seattle**
- **South/SE Seattle**

These are project-defined geographic groupings, not official administrative units. Unmatched points can be lost from the grouped network, and the aggregation hides variation among individual neighborhoods.

## Bipartite graph construction

`bipartiteD3::bipartite_D3()` receives the category-by-region matrix. Pest categories form the primary nodes, regions form the secondary nodes, and each edge width is proportional to the number of service records in that category–region pair. Nodes are sorted by their total counts, pest nodes use a manual `RColorBrewer::Paired` palette, and the graph exposes percentages interactively.

The same function is called once for the full year and once for each season. This makes the charts comparable while allowing the network structure and dominant edges to change with the time filter.

## Full-year and seasonal results

The [published Posit report](https://01a0bc03-40b2-ca8c-6045-21517970ca76.share.connect.posit.cloud/) contains the interactive full-year and seasonal graphs. They are not embedded in this repository because self-contained HTML can carry generated data in its payload.

Reported patterns from the source analysis:

- **Full year:** rodents accounted for **46.78%** of records, ants **31.23%**, and cockroaches **15.74%**. Rodents were especially prominent in South/SE and Central Seattle; ants had a strong West Seattle connection; cockroaches were concentrated in Downtown and South/SE Seattle.
- **Winter:** rodents **54.58%**, ants **21.80%**, cockroaches **18.59%**.
- **Spring:** rodents **46.52%**, ants **35.79%**, cockroaches **13.93%**.
- **Summer:** ants led at **40.45%**, followed by rodents at **39.80%** and cockroaches at **13.23%**.
- **Fall:** rodents **47.21%**, ants **25.83%**, and cockroaches **17.52%**.

Across seasons, the graphs show rodents as the leading category except in summer, when ants lead. Cockroach edges remain especially visible in Downtown Seattle. Smaller categories collectively make up less than 10% of annual activity in the source summary, though their relative share can still matter in a seasonal or regional slice.

## Conclusion and limitations

The network view makes two kinds of structure visible at once: which pest categories dominate overall and which regions account for each category's activity. The clearest recurring pattern is a contrast between rodent-heavy colder periods and a summer increase in ant activity, alongside persistent Downtown connections for cockroaches.

Interpretation is limited by:

- service demand and preventive contracts rather than a random sample of pest presence;
- repeat visits and customer mix;
- inconsistent technician-entered labels and broad category consolidation;
- geocoding quality, spatial-join misses, and region definitions created for this analysis;
- a single company's Seattle service records from 2023;
- interactive-package constraints on labels, layout, and customization.

These results are descriptive. They should not be used as citywide prevalence estimates or causal claims.

## Reproducibility

To rerun the analysis, obtain authorized copies of the private source workbook, prepared workbook, and complete neighborhood shapefile separately. Place them in a local directory outside version control, set `SEATTLE_PEST_DATA_DIR` to that directory, and render:

```r
rmarkdown::render("seattle-pest-service-patterns.Rmd")
```

The repository's `.gitignore` is preserved to help prevent private and intermediate files from being staged. Reproduction requires access to the proprietary inputs, but it does **not** require committing those inputs or any rendered HTML containing generated chart data.

## Repository contents

- `seattle-pest-service-patterns.Rmd` — sanitized, public workflow and setup.
- `README.md` — methods, interpretation, privacy boundary, and reproduction guidance.
- `.gitignore` — safeguards for private and generated files.
