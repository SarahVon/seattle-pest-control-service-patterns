# Seattle Pest Control Service Patterns: A Two-Mode Network Analysis (2023)

**Published results:** [Open the interactive report on Posit Connect Cloud](https://01a0eab7-b176-9103-0ac6-0aeca19f42c3.share.connect.posit.cloud/)  
**Project source:** [`seattle-pest-service-patterns.Rmd`](seattle-pest-service-patterns.Rmd)

This project explores how pest-control service activity varied across Seattle neighborhoods and seasons in 2023. Using R, spatial joins, and interactive bipartite graphs, I transformed service records into a view of which pest categories were most common and where those services occurred.

A service record may represent a confirmed issue, suspected problem, or preventative treatment, not necessarily a pest sighting.

## Contents

- [Project Overview](#project-overview)
- [Data and Privacy](#data-and-privacy)
- [Data Preparation](#data-preparation)
- [Spatial Analysis](#spatial-analysis)
- [Bipartite Network Analysis](#bipartite-network-analysis)
- [Results](#results)
- [Limitations](#limitations)
- [Data Sources](#data-sources)

## Project Overview

The analysis combines more than 30,000 service records from a local Seattle pest-control company with neighborhood boundary data from the Seattle City GIS Open Data Portal.

The workflow uses R for data preparation, geocoding, spatial analysis, aggregation, and interactive network visualization. Key packages include `tidyverse`, `lubridate`, `tidygeocoder`, `sf`, `bipartiteD3`, and `RColorBrewer`.

The analysis focuses on three questions:

1. Which broad pest categories account for the most service activity?
2. How are those services distributed across Seattle?
3. How do these patterns change by season?

## Data and Privacy

The service data is proprietary and is not included in this repository. To protect customer privacy, individual addresses, coordinate values, and record-level service data are not published.

The public project includes the analysis workflow, aggregated results, and interactive visualizations without identifying the company or its customers.

## Data Preparation

Service records were filtered to Seattle addresses and service dates were standardized and grouped into winter, spring, summer, and fall.

Because technician-entered pest labels were not standardized, related labels were consolidated into broader categories:

- Ants
- Rodents
- Cockroaches
- Biting Insects
- Storage and Structure Pests
- Stinging Insects
- Spiders
- Flying Insects
- Misc Crawling Insects

This grouping improves comparison across the network while reducing detail within individual pest classifications.

## Spatial Analysis

Service addresses were geocoded to latitude and longitude and converted to spatial points using `sf`.

Seattle sub-neighborhood boundaries from the Seattle City GIS Open Data Portal were transformed from their source coordinate reference system (EPSG:2926) to WGS84 (EPSG:4326). A spatial join using `st_within()` then assigned each service location to a sub-neighborhood.

To keep the network visualizations readable, the sub-neighborhoods were grouped into seven project-defined Seattle regions:

- NW Seattle
- NE Seattle
- Magnolia/Queen Anne
- Central Seattle
- Downtown Seattle
- West Seattle
- South/SE Seattle

These groupings were created specifically for this analysis and are not official administrative units.

## Bipartite Network Analysis

The final visualization uses a two-mode, or bipartite, network connecting two sets of nodes:

- **Pest categories**
- **Seattle regions**

Connections between them are weighted by the number of service records associated with each category-region combination.

A reusable R function aggregates the data, constructs the category-by-region matrix, orders categories and regions by service frequency, and generates an interactive `bipartiteD3` visualization. The same function is used for the full-year analysis and each season so the results remain comparable.

## Results

Across the full year, rodents accounted for **46.78%** of service records, followed by ants (**31.23%**) and cockroaches (**15.74%**).

Rodents were the largest category in winter, spring, and fall. Summer was the only season in which ants surpassed rodents, accounting for **40.45%** of records compared with **39.80%** for rodents.

The network also highlights geographic patterns. Ant services had a strong connection with West Seattle, while cockroach services were concentrated in Downtown and South/SE Seattle.

The interactive full-year and seasonal networks can be explored in the [published Posit report](https://01a0eab7-b176-9103-0ac6-0aeca19f42c3.share.connect.posit.cloud/).

## Limitations

The results describe service activity rather than pest prevalence across Seattle. Interpretation is limited by:

- service demand and preventative contracts rather than a random sample of pest presence;
- repeat visits and customer mix;
- inconsistent technician-entered labels and broad category consolidation;
- geocoding quality, spatial-join misses, and project-defined region boundaries;
- use of a single company's Seattle service records from 2023; and
- interactive-package constraints on labels, layout, and customization.

These results are descriptive and should not be interpreted as citywide prevalence estimates or causal relationships.

## Data Sources

**Service records:** Proprietary 2023 data from a local pest-control company. The company is not identified, and the underlying records are not publicly shared.

**Neighborhood boundaries and reference map:** City of Seattle, Office of Planning and Community Development. *Neighborhood Map Atlas—Neighborhoods* (`cityclerk_nma_nhoods`) [GIS dataset]. Derived from the Seattle City Clerk's Office Geographic Indexing Atlas. Available through the [Seattle City GIS Open Data Portal](https://data-seattlecitygis.opendata.arcgis.com/datasets/b4a142f592e94d39a3bf787f3c112c1d/explore). Metadata updated December 1, 2020.
