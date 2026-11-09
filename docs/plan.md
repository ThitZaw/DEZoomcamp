# SEA Forest Monitor

## Project Plan --- Version 1.0

**Project type:** Geospatial data engineering and analytics\
**Project status:** Planning finalized; data validation is the next
activity\
**Release:** MVP / Version 1\
**Publishing goal:** Personal portfolio project

------------------------------------------------------------------------

## 1. Project Overview

SEA Forest Monitor is a geospatial data analytics project designed to
analyze historical tree-cover loss across selected Southeast Asian
countries.

The project will collect, process, aggregate, and visualize
satellite-derived forest data as country-level statistics through an
interactive dashboard.

### Main research question

**How has tree cover changed across selected Southeast Asian countries,
and which countries have experienced the greatest tree-cover loss?**

## 2. Project Objectives

1.  Build a reproducible geospatial data pipeline.
2.  Analyze historical tree-cover loss from 2001--2025, subject to
    source-data availability and validation.
3.  Compare tree-cover-loss patterns across selected countries.
4.  Calculate annual loss, cumulative loss, and cumulative loss
    percentage.
5.  Visualize geographic and historical patterns at country level.
6.  Demonstrate practical data engineering and geospatial analytics
    skills through a public portfolio project.

## 3. Project Scope

### 3.1 Geographic coverage

Southeast Asia, with a maximum of five countries in Version 1.

Initial candidate countries:

-   Malaysia
-   Indonesia
-   Thailand
-   Myanmar
-   Vietnam

These are candidate countries, not yet final selections. Final inclusion
depends on data validation.

### 3.2 Historical coverage

The intended analysis period is 2001--2025, subject to source dataset
availability, version validation, and consistent methodology.

### 3.3 Spatial granularity

Country-level analysis only.

Sub-country analysis, such as state or province comparisons, and
pixel-level dashboard exploration are excluded from Version 1. Source
raster pixels may still be processed behind the scenes to calculate
country-level statistics.

### 3.4 Tree-canopy threshold

Use a **30% baseline tree-canopy threshold**.

Only pixels with at least 30% tree canopy in 2000 qualify for the
baseline tree-cover area and associated tree-cover-loss calculations.

This is an analytical threshold for this project. It is not intended to
represent a universal or legal definition of forest.

### 3.5 In-scope features

-   Historical tree-cover analysis.
-   Annual tree-cover loss.
-   Cumulative tree-cover loss.
-   Cumulative loss percentage relative to baseline tree cover.
-   Country-level comparisons.
-   Country-level choropleth visualization.
-   Historical trend analysis.
-   Country and year filters.
-   On-demand data pipeline execution.
-   Incremental processing with support for historical corrections.

### 3.6 Out of scope

-   Animal tracking and migration.
-   GPS tracking.
-   State/province-level analysis.
-   Geographic hotspot detection.
-   Real-time monitoring.
-   Machine learning and predictive analytics.
-   Forest regrowth analysis.
-   Complex distributed processing.

## 4. Data Requirements

### 4.1 Primary datasets

  -----------------------------------------------------------------------
  Dataset                 Candidate source        Purpose
  ----------------------- ----------------------- -----------------------
  Country boundaries      Natural Earth           Country boundaries,
                                                  geographic filtering,
                                                  and mapping

  Tree cover and loss     Hansen Global Forest    Historical baseline
                          Change                  tree cover and annual
                                                  tree-cover-loss
                                                  analysis
  -----------------------------------------------------------------------

Google Earth Engine is the proposed access and extraction approach for
Hansen data. The exact extraction method will be decided during
technical planning.

### 4.2 Data selection criteria

A country can be included in Version 1 when:

-   A valid and suitable country boundary is available.
-   The source data covers the required geography.
-   Historical observations are available for the intended analysis
    period.
-   Country-level statistics can be calculated consistently.
-   The data is sufficiently complete and suitable for comparison.

The project will include no more than five countries.

### 4.3 Licensing and attribution

The project aims to use datasets that are accessible without paid
subscriptions.

-   Natural Earth data is public domain.
-   Hansen Global Forest Change is distributed under CC BY 4.0 and
    requires appropriate attribution.
-   Source-specific terms must be reviewed and recorded before
    publishing data or derived outputs.
-   Dataset versions, source references, extraction dates, and
    attribution must be documented.

A free-to-access dataset is not necessarily attribution-free. The
project will comply with the applicable terms.

## 5. Analytical Requirements

### 5.1 Baseline tree cover

Calculate the area of pixels with at least 30% tree canopy in 2000.

**Unit:** hectares.

### 5.2 Annual tree-cover loss

Calculate the area of qualifying baseline pixels that experienced
tree-cover loss during each year.

**Unit:** hectares per year.

### 5.3 Cumulative tree-cover loss

Calculate the sum of annual tree-cover loss from 2001 through the
selected year.

**Unit:** hectares.

### 5.4 Cumulative loss percentage

Calculate cumulative tree-cover loss as a percentage of baseline tree
cover in 2000.

**Formula:**

`Cumulative loss percentage = (Cumulative tree-cover loss / Baseline tree cover) × 100`

The denominator is the qualifying baseline tree-covered area in 2000,
not the country's total land area.

### 5.5 Analytical limitations

-   Tree-cover loss does not necessarily represent permanent
    deforestation.
-   Tree-cover loss does not necessarily indicate natural-forest loss.
-   The 30% canopy threshold is a project-specific analytical choice,
    not a universal forest definition.
-   Country comparisons should consider differences in baseline
    tree-covered area.
-   Source methodology and version changes must be documented.
-   The Hansen baseline tree-cover layer and loss layer have specific
    definitions and limitations; calculated metrics must follow the
    source documentation.

## 6. Dashboard Requirements

The dashboard will contain three pages.

### 6.1 Regional Overview

**Purpose:** Provide an overview of tree-cover loss across the selected
Southeast Asian countries.

Requirements:

-   Display total tree-cover loss across the selected countries.
-   Identify the country with the highest tree-cover loss for the
    selected period.
-   Display a country-level choropleth map.
-   Display country-level statistics.
-   Support year selection.

### 6.2 Country Comparison

**Purpose:** Compare tree-cover loss between countries.

Requirements:

-   Compare annual tree-cover loss by country.
-   Compare cumulative tree-cover loss by country.
-   Rank countries by tree-cover loss.
-   Compare historical country-level trends.

### 6.3 Historical Trends

**Purpose:** Explore how tree-cover loss changes over time.

Requirements:

-   Display annual tree-cover-loss trends.
-   Display cumulative tree-cover-loss trends.
-   Support historical year selection.
-   Filter trends by country.

### 6.4 Dashboard filters

-   Country selection.
-   Year selection.

The 30% canopy threshold remains fixed in Version 1.

### 6.5 Geographic display limitation

The dashboard will display country-level choropleth results only. It
will not display or claim to identify sub-country forest-loss hotspots.

## 7. Data Quality Requirements

The following checks are mandatory.

  -----------------------------------------------------------------------
  ID                      Requirement             Expected behaviour
  ----------------------- ----------------------- -----------------------
  DQ-01                   Country identifiers     Reject missing or
                                                  invalid country
                                                  identifiers

  DQ-02                   Measurements            Reject missing or
                                                  invalid measurements
                                                  where a value is
                                                  required

  DQ-03                   Duplicate records       Detect duplicate
                                                  country-year records

  DQ-04                   Negative values         Reject negative
                                                  tree-cover-loss
                                                  measurements

  DQ-05                   Cumulative consistency  Ensure cumulative loss
                                                  does not decrease over
                                                  time for a consistent
                                                  dataset version and
                                                  methodology

  DQ-06                   Data completeness       Identify missing
                                                  country-year
                                                  combinations

  DQ-07                   Data lineage            Record source, dataset
                                                  version, and extraction
                                                  date

  DQ-08                   Error logging           Record rejected records
                                                  and their rejection
                                                  reasons
  -----------------------------------------------------------------------

**Missing data must not automatically be converted to zero.** Zero means
a valid observation of no detected loss; missing means that a valid
measurement is unavailable.

## 8. Data Pipeline Requirements

### 8.1 Execution

-   Pipeline execution is on demand.
-   The initial run loads the available historical period.
-   Subsequent runs use incremental processing where possible.
-   The pipeline must support reprocessing affected historical periods
    when source data or methodology changes.

### 8.2 Processing workflow

1.  Extract source data.
2.  Validate source and geographic inputs.
3.  Clean and transform extracted values.
4.  Calculate annual and cumulative metrics.
5.  Aggregate statistics by country and year.
6.  Validate transformed results.
7.  Load validated results into the analytical store.
8.  Record execution details and errors.

### 8.3 Reliability and traceability

-   Idempotent execution.
-   Duplicate prevention.
-   Execution status and error logging.
-   Source dataset version tracking.
-   Extraction timestamp tracking.
-   Historical correction support.
-   Rejected-record logging.

Running the pipeline repeatedly against the same source version and
parameters must not duplicate records or inflate metrics.

## 9. Acceptance Criteria

Version 1 is complete when:

1.  Up to five validated Southeast Asian countries are supported.
2.  Annual tree-cover-loss statistics are available for the validated
    portion of 2001--2025.
3.  Annual loss, cumulative loss, and cumulative loss percentage are
    calculated correctly.
4.  All mandatory data-quality checks are implemented and tested.
5.  All three dashboard pages are functional.
6.  Country and year filters work correctly.
7.  Pipeline reruns do not create duplicate records or inflate results.
8.  Documentation explains the data sources, licensing and attribution,
    methodology, data definitions, and limitations.

If source data does not support a complete year or country, the
limitation must be reported rather than filled with invented or assumed
values.

## 10. Future Enhancements

The following features are excluded from Version 1 and may be considered
for later releases:

-   State and province-level analysis.
-   Geographic forest-loss hotspot analysis.
-   Asian elephant habitat distribution.
-   Tree-cover loss within mapped elephant ranges.
-   Protected-area analysis.
-   Additional satellite datasets.
-   Forest gain and regrowth analysis.

Future enhancements must be assessed against data availability,
licensing, maintenance cost, and a clearly defined analytical
requirement.

## 11. Technology Selection

Technology selection is intentionally deferred to a separate technical
planning stage.

That stage will evaluate options for:

-   Data extraction.
-   Geospatial processing.
-   Data transformation and orchestration.
-   Data storage.
-   Backend services, if required.
-   Dashboard development.
-   Testing and deployment.

No additional technology or infrastructure should be introduced without
a clear project requirement.

------------------------------------------------------------------------

**End of Project Plan --- SEA Forest Monitor Version 1.0**
