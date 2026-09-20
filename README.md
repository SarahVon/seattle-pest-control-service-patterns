# Seattle Pest Control Service Patterns (2023)

This portfolio project examines how a local pest-control company's Seattle service records varied by broad pest category, Seattle region, and season during 2023. It is a **two-mode network**: one node set is consolidated pest categories and the other is seven analysis regions. Edge weights are counts of service records connecting a category to a region.

## Data handling and privacy

The underlying service records are proprietary. They may contain service addresses, geocoded coordinates, customer-related details, and record-level dates/labels, so no raw or prepared data is included here. The portfolio HTML was intentionally omitted because its self-contained interactive payload contains embedded generated chart data. The public source is the portfolio-specific R Markdown file; it documents the workflow without publishing records, addresses, coordinates, or customer details.

The omitted inputs include the private source workbook, prepared analysis workbook, and Seattle neighborhood-boundary shapefile (including its sidecar files). Do not add `.xlsx`, shapefile, address, coordinate, `.RData`, `.Rhistory`, DOCX, rsconnect, or deployment files to this repository.

## Workflow

1. Read the private service workbook and retain Seattle records.
2. Parse service dates and derive winter, spring, summer, and fall.
3. Consolidate inconsistent technician-entered pest labels into broader categories (for example, ants, rodents, biting insects, cockroaches, spiders, and stinging insects).
4. Read Seattle City GIS neighborhood boundaries, transform them to WGS84, and spatially join geocoded service points to neighborhoods.
5. Map many neighborhood names into seven legible analysis regions: NW Seattle, NE Seattle, Magnolia/Queen Anne, Central Seattle, Downtown Seattle, West Seattle, and South/SE Seattle. These are project groupings, not official administrative units.
6. Use the prepared private table to aggregate records by `Target` and `Neighborhood` (and by `Season` for seasonal views).
7. Render interactive bipartite charts with `bipartiteD3`, using counts as edge weights and consistent category colors.

The source also documents the geocoding step. Geocoding is not rerun automatically: the source assumes coordinates have already been prepared in the private inputs. The public code uses `SEATTLE_PEST_DATA_DIR` (default `data-private`) rather than an absolute local path.

## Reproduction

Install R and the packages listed in the setup chunk of `seattle-pest-service-patterns.Rmd`, including `bipartiteD3`, `bipartite`, `readxl`, `r2d3`, `tidyverse`, `RColorBrewer`, `sf`, `rnaturalearth`, `rnaturalearthdata`, `patchwork`, `writexl`, `tidygeocoder`, `lubridate`, and `printr` (plus their system dependencies). Obtain authorized copies of the private workbooks and neighborhood shapefile separately, place them in a private input directory, and set `SEATTLE_PEST_DATA_DIR` to that directory. Render the Rmd with `rmarkdown::render()`; never commit that input directory.

## Outputs and interpretation

The report produces one full-year chart and four seasonal charts. In the source report, rodents led three seasons and ants led summer; the percentages describe this company's service activity, not citywide pest prevalence. Repeat visits, preventive treatments, customer mix, technician labeling, missing spatial matches, and the seven-region aggregation all affect the patterns.

## Validation and limitations

The source checks row counts, columns, missing values, category frequencies, and the spatial join workflow. Results are descriptive and should not be interpreted as confirmed sightings, population estimates, or causal effects. The private data cannot be independently reproduced from this public repository.

## Repository contents

- `seattle-pest-service-patterns.Rmd` — sanitized portfolio source and workflow.
- `README.md` — data, methods, privacy, and reproduction documentation.
- `.gitignore` — prevents private/intermediate files from being staged.

## Attribution

Neighborhood assignment uses Seattle City GIS neighborhood boundaries: [Seattle City GIS Open Data](https://data-seattlecitygis.opendata.arcgis.com/datasets/b4a142f592e94d39a3bf787f3c112c1d/explore). R package and data-source attribution is documented in the R Markdown source.
