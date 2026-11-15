# Municipal Building-Permit Pipeline (Israel)

An end-to-end ETL pipeline that collects building-permit requests from Israeli municipal engineering
websites, isolates the **commercial / industrial** ones, enriches them with detail pages and official
cadastral parcels, and produces a **building-rights utilization report per parcel** (מימוש זכויות לפי היתרים).

> 🔒 This is a project overview. The source code is private and available on request.

## Results so far (Ramat Gan)

| Stage | Output |
|---|---|
| Collect requests | **33,811** permit requests across **38** cadastral blocks (gush) |
| Classify | **1,536** employment/industry requests, each with a confidence level (784 high · 597 medium · 155 low) and the reason for the decision |
| Detail pages | **1,449** request pages scraped: description, main use, areas, committee decisions, zoning |
| Parcels & rights | Utilization workbook for block 6109: **236** parcels, 591 requests, 59 zoning plans referenced |

Petah Tikva (Segula industrial zone) is configured and has completed the first two stages.

## How it works

```mermaid
flowchart LR
    A[1. Collect requests<br/>Playwright] --> B[2. Classify<br/>rules + regex]
    B --> C[3. Detail pages<br/>Playwright]
    C --> D[4. Change of use<br/>& additions]
    D --> E[5. Cadastral parcels<br/>national SHP layer]
    E --> F[6. Rights per parcel<br/>Excel report]
```

1. **Collect** – pulls the full list of permit requests per block from municipal sites running the Complot system, with polite rate limiting.
2. **Classify** – offline classification into employment/industry, public/institutional, residential and a manual-review queue, using address keywords, applicant names and employment-zone blocks.
3. **Enrich** – visits each employment request's detail page. Resumable from a checkpoint; stops and waits for a person if the site shows a verification step rather than trying to bypass it.
4. **Detect changes** – flags change of use (שימוש חורג / שינוי ייעוד) and building additions in the scraped text.
5. **Parcels** – reads the national parcel layer (Israel Mapping Center) straight from the zipped shapefile with a pure-Python SHP/DBF reader and converts ITM → WGS84.
6. **Report** – joins requests, committee decisions and parcels into a formatted workbook per block: permitted rights, existing area, utilization, plans mentioned and source quotes.

## Design choices

- **Config-driven, multi-city:** each municipality is one JSON file (site, blocks, address format, plan prefixes, employment zones). A new city needs a config, not new code.
- **Explainable classification:** every row carries a confidence level and the rule that produced it, so the manual-review queue is small and auditable.
- **Privacy:** the collected records include applicants' names and are never published.

## Tech

Python · Playwright · pandas · regex · openpyxl · shapefile parsing (no GeoPandas) · ITM/WGS84 projection
