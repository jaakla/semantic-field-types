# Semantic Field Type System

A comprehensive classification system for data fields designed to support high-fidelity automated data profiling, quality monitoring, lineage validation, and analysis—particularly in AI-driven business intelligence scenarios. This system is tailored for analytical databases, such as organizational data warehouses and data lakes, where a diverse range of users (including both humans and AI agents) interact with well-structured datasets maintained by others.

## Purpose

This semantic type system helps:
- **Data Profiling**: Generate meaningful summaries based on field semantics
- **Data Quality**: Apply appropriate validation rules per semantic type
- **Lineage Validation**: Ensure semantically compatible fields are joined/related
- **BI Analysis**: Validate correct aggregations and visualizations
- **Query Generation**: Guide LLMs and analysts toward semantically valid SQL

## Core Principles

1. **Multiple Dimensions**: Fields can have multiple semantic aspects (e.g., a field can be both a temporal event and a deadline)
2. **Inheritance**: Subtypes inherit properties from parent types
3. **Composability**: Semantic markers combine to describe complex field meanings
4. **Pragmatism**: Focus on actionable metadata that drives tooling
5. **Flexibility**: Everything in the semantic definition is optional


## Versioning
0.1 - initial pre-release (January 2026)

### Known Limitations

* Handling of composite types: data can be stored as arrays, key-value pairs, and structured or semi-structured hierarchies. Modern data systems frequently use nested structures that need first-class support. In current system it can be mentioned as Specifier
* Validation framework: a way to specify well-defined and machine-implementable constraints, patterns (e.g. regexp), and business rules
* Aggregation semantics: Richer aggregation framework that specifies allowed operations and their mathematical definitions per type
* Complex time types for periodic/seasonal patterns
* Sensitive personal details (gender, martial status, religion, health details, biometric data etc) not covered yet, it should be present in very specific databases only.

### Future Considerations: Alignment with External Standards

This taxonomy is currently a pragmatic, internal classification. Future versions should consider alignment with established standards for interoperability and compliance:

#### Privacy & Data Protection
* **W3C Data Privacy Vocabulary (DPV) 2.0** - Machine-readable vocabulary for personal data categories, aligned with GDPR. Namespace: `https://w3id.org/dpv/pd#`. Consider adopting DPV categories for Section 10 (Personal Data Types).
* **GDPR Article 9 Special Categories** - Legal basis for "Sensitive PII" classification; includes health, biometric, genetic data categories not yet covered here.
* **ISO 27701** - Privacy information management extension to ISO 27001; provides framework for data protection controls.

#### Semantic Web & Metadata
* **Schema.org** (v29.4) - 827 entity types, 1528 properties. Consider aligning entity types (Person, Organization, Place) and adopting JSON-LD format for machine-readability.
* **Dublin Core (ISO 15836)** - 15 core metadata elements. Relevant for Metadata Types section standardization.
* **ISO/IEC 11179** - International standard for metadata registries and data element definitions. Provides formal framework for semantic type registration.

#### Domain-Specific (Logistics/Transport)
* **UN/CEFACT Buy-Ship-Pay Reference Data Model** - Semantic vocabulary for international trade and transport (Consignment, Equipment, DangerousGoods, GeographicalCoordinate). Published as JSON-LD at `vocabulary.uncefact.org`.
* **GS1 Global Data Model** - Supply chain identifiers (GTIN, GLN, SSCC) and data exchange standards.
* **UN/LOCODE** - Authoritative location codes for transport/logistics.

#### Healthcare (Reference Model)
* **FHIR Data Types** - Well-structured complex types (`HumanName`, `Address`, `ContactPoint`, `Identifier`) that could inform our personal data type structure.

#### Data Quality & Profiling Tools
* **Great Expectations Semantic Types** - `semantic_types_dict` pattern for automated profiling by semantic type.
* **Soda Core** - Data profiling with check suggestions; potential integration point.

#### Future Actions
1. Add namespace/URI support for machine-readable type definitions
2. Publish as JSON-LD or RDF for tool integration
3. Map types to W3C DPV categories for privacy automation
4. Align transport entities with UN/CEFACT vocabulary
5. Consider formal registration per ISO/IEC 11179

---

## Semantic Type Hierarchy

### 1. IDENTIFIER Types 🆔

Fields that uniquely identify entities in the table, used externally as reference.

#### 1.1 Primary Identifier
- **Purpose**: Uniquely identifies a record within a table. Usually automatically assigned internally
- **Examples**: `transport_id`, `order_id`, `customer_id`
- **Specifiers**: numeric, alphanumeric, uuid, incremental
- **Profiling**: Count distinct, check uniqueness, detect gaps
- **Quality Rules**: 
  - Must be NOT NULL
  - Must be UNIQUE within given scope - No duplicates allowed
  - Should be immutable
- **Valid Aggregations**: COUNT, COUNT DISTINCT
- **Invalid Aggregations**: SUM, AVG, MIN, MAX (except for gap detection)
- **Valid Joins**: Can join to foreign keys of same entity type
- **Visualization**: Bar and line charts of counts, distribution histograms

#### 1.2 Natural Identifier
- **Purpose**: Human-readable identifier from business domain, assigned externally
- **Examples**: `alphanumeric_number`, `order_number`, `tracking_code`
- **Specifiers**: content type: numeric, alphanumeric, uuid, incremental
- **Profiling**: Pattern analysis, format validation, uniqueness check
- **Quality Rules**:
  - Should follow consistent format/pattern
  - Check for format violations
  - Normally unique within given scope - namespace, but duplicates are accepted even within namespace
- **Valid Aggregations**: COUNT, COUNT DISTINCT
- **Invalid Aggregations**: SUM, AVG, MIN, MAX (except for gap detection)
- **Joins**: Generally not useful for joins
- **Visualization**: Text lists, pattern frequency charts, bar and line charts of counts, distribution histograms

#### 1.3 Foreign Key Master Data Reference 🔗
- **Purpose**: References primary identifier in another master data type table
- **Examples**: `shipper_id`, `carrier_id`
- **Profiling**: Orphan detection, cardinality analysis, reference distribution
- **Quality Rules**:
  - Check referential integrity
  - Detect orphan records - referenced entity exists, strict check
- **Valid Aggregations**: COUNT, COUNT DISTINCT
- **Valid Joins**: Must join to compatible primary identifiers in the given master data table
- **Lineage**: Must trace to source entity table
- **Visualization**: Reference distribution, orphan detection reports, Top counts

#### 1.4 Foreign Key Fact Reference 🔗
- **Purpose**: References primary identifier in another fact (transaction) type table
- **Examples**: `assigned_offer_id`, `delivery_id`, `transport_id`
- **Profiling**: Orphan detection, cardinality analysis
- **Quality Rules**:
  - Check referential integrity
  - Detect orphan records, validate referenced entity exists. Forgiving check - referenced entity may be loaded later.
- **Valid Aggregations**: COUNT, COUNT DISTINCT
- **Valid Joins**: Must join to compatible primary identifiers
- **Lineage**: Must trace to source entity tables
- **Visualization**: Orphan detection reports, time distribution of orphans - accept recent ones, problems are if old records are orphans.


#### 1.5 Composite Key Component
- **Purpose**: Part of a multi-column unique identifier, may function as namespace
- **Examples**: `start_h3_10 + end_h3_10` (two equal strength keys), `customer_id (namespace) + alphanumeric_number`
- **Profiling**: Combined uniqueness, partial key cardinality
- **Quality Rules**: Components together must be unique
- **Valid Joins**: Use all components together in joins

#### 1.6 Self-Referencing Foreign Key 🔄
- **Purpose**: References primary identifier in the same table, creating hierarchical/tree structures
- **Examples**: `main_company_id` (parent company), `parent_location_id`, `manager_user_id`
- **Profiling**: Tree depth analysis, orphan detection, cycle detection
- **Quality Rules**:
  - Check referential integrity within same table
  - No circular references (A→B→A)
  - NULL allowed for root/top-level records
  - Validate related classification flags (e.g., `is_subcompany = true` when `main_company_id` is not null)
- **Valid Aggregations**: COUNT, COUNT DISTINCT, tree aggregations (subtree counts)
- **Valid Joins**: Self-join to primary key for parent/child traversal
- **Visualization**: Tree diagrams, hierarchy charts, org charts
- **Analysis**: Hierarchy depth, parent-child ratios, consolidation analysis

---

### 2. CATEGORICAL Types 📝

Fields with discrete, enumerable values.

#### 2.1 Status/State 🎯
- **Purpose**: Current state in a workflow or lifecycle
- **Examples**: `last_observed_status`, `final_status`, `acknowledgement_state`
- **Specifiers**: list of allowed values, open ended or not, allow null
- **Profiling**: Value frequency, state transition analysis, terminal states, missing states
- **Quality Rules**:
  - Must be from known value set
  - Check for unexpected values
  - Validate state transitions (if temporal ordering available)
- **Valid Aggregations**: COUNT, MODE, GROUP BY
- **Visualization**: Pie charts, bar charts, sankey diagrams (transitions), DAG flow charts
- **Analysis**: Status funnels, conversion rates between states

#### 2.2 Category/Classification 🏷️
- **Purpose**: Assigns record to discrete category
- **Examples**: `shipper_tpa_industry`, `vehicle_type`, `creation_mode`
- **Specifiers**: list of allowed values, open ended or not, allow null
- **Profiling**: Category distribution, cardinality, frequent/top and rare categories
- **Quality Rules**:
  - Check against allowed values (if controlled)
  - Detect typos/variants
  - Monitor category emergence and obsolence
- **Valid Aggregations**: COUNT, GROUP BY
- **Visualization**: Bar charts, pie charts, treemaps
- **Analysis**: Category performance comparison, segmentation

#### 2.3 Reason/Qualifier Code
- **Purpose**: Explains why something happened
- **Specifiers**: list of allowed values, open ended or not, allow null
- **Examples**: `decline_reason_qualifier`, `rtv_main_visibility_issue`
- **Profiling**: Reason frequency, patterns by other dimensions
- **Quality Rules**: Should be from known code set, typically open ended qualifier
- **Valid Aggregations**: COUNT, GROUP BY
- **Visualization**: Pareto charts, reason frequency bars

#### 2.4 Method/Type Descriptor
- **Purpose**: Describes how or what method was used
- **Examples**: `rtv_emissions_final_calculation_method`, `fuel_type`, `transport_assingment_method`
- **Specifiers**: list of allowed values, open ended or not, allow null
- **Profiling**: Method distribution, method effectiveness
- **Quality Rules**: Should be from known code set
- **Valid Aggregations**: COUNT, GROUP BY
- **Analysis**: Method comparison, impact analysis

#### 2.5 Entity Type Discriminator 🏷️
- **Purpose**: Distinguishes different entity types within a shared table (polymorphic table pattern)
- **Examples**: `company_type` (S=Shipper, C=Carrier), `user_type`, `document_type`, `record_type`
- **Specifiers**: Closed enumeration, typically single character or short code
- **Profiling**: Type distribution, type-specific field population
- **Quality Rules**:
  - Must be from known closed set
  - NOT NULL (every record must have a type)
  - Immutable after creation
  - Certain fields may only be valid for specific types
- **Valid Aggregations**: COUNT, GROUP BY
- **Visualization**: Pie charts, stacked bars by type
- **Analysis**: Type distribution, type-specific metrics, data model partitioning
- **Usage Pattern**: Often used as filter condition or for conditional joins

#### 2.6 Support Phase/Block Level 🚦
- **Purpose**: Describes the support or operational status level of an entity
- **Examples**: `tp_support_phase`, `tc_support_phase`, `block_level`, `subscription_tier`
- **Specifiers**: Ordered or unordered levels, typically small closed set
- **Profiling**: Level distribution, level transitions over time
- **Quality Rules**:
  - Should be from known level set
  - May have ordering/hierarchy (e.g., blocked > warning > active)
- **Valid Aggregations**: COUNT, GROUP BY
- **Visualization**: Funnel charts, status distribution
- **Analysis**: Support escalation patterns, block rate analysis

---

### 3. BOOLEAN Types ✅

True/false flags indicating presence/absence of a property.

#### 3.1 Feature Flag
- **Purpose**: Indicates if feature/service is enabled
- **Examples**: `is_rtv_enabled` 
- **Profiling**: TRUE/FALSE counts, NULL handling, adoption rates
- **Quality Rules**:
  - Should be NOT NULL (if business rule requires it)
  - TRUE/FALSE/NULL distribution
- **Valid Aggregations**: COUNT, SUM (treating TRUE=1), PROPORTION
- **Visualization**: Stacked bars, pie charts, adoption trends over time
- **Analysis**: Feature adoption, flag correlation analysis

#### 3.2 Classification Flag
- **Purpose**: Classifies record into binary category, 2-value case for Category/Classification 🏷️
- **Examples**: `is_contracted`, `is_freight_forwarding`
- **Anti-pattern**: Some fields like `is_domestic_not_export` should be modeled Category field with two values
- **Profiling**: Category split ratios, NULL handling
- **Valid Aggregations**: COUNT, SUM, PROPORTION, GROUP BY
- **Visualization**: Stacked bars, pie charts
- **Analysis**: Category comparison, impact of classification on metrics

#### 3.3 Derived Boolean Status
- **Purpose**: Boolean derived from other logic/thresholds
- **Examples**: `offer_deadline_passed`, `late_delivery`
- **Profiling**: Derivation logic validation, consistency checks
- **Quality Rules**: Validate derivation logic matches source fields
- **Lineage**: Must trace to source fields used in derivation

#### 3.4 Inheritance Flag 🔗
- **Purpose**: Indicates if a property/setting is inherited from a parent record (typically in hierarchical structures)
- **Examples**: `from_main_bankdata`, `from_main_address`, `from_main_provisioning`, `inherits_parent_settings`
- **Profiling**: Inheritance rate, correlation with hierarchy structure
- **Quality Rules**:
  - Should only be TRUE when parent reference exists (e.g., `main_company_id IS NOT NULL`)
  - If TRUE, the inherited field value should match parent's value
  - FALSE or NULL for root/standalone records
- **Valid Aggregations**: COUNT, SUM, PROPORTION
- **Related Fields**: Typically paired with a self-referencing foreign key and the actual data field being inherited
- **Analysis**: Inheritance adoption rates, data consistency between parent and child

#### 3.5 Communication Preference Flag 📧
- **Purpose**: Indicates if entity should receive specific types of communications
- **Examples**: `gets_email`, `receives_reports`, `receives_messages`, `opt_in_marketing`
- **Profiling**: Opt-in/opt-out rates, preference distribution
- **Quality Rules**:
  - Should be NOT NULL for active entities
  - May have legal/compliance implications (GDPR)
- **Valid Aggregations**: COUNT, SUM, PROPORTION
- **Visualization**: Pie charts, adoption trends
- **Analysis**: Communication reach, preference patterns, compliance monitoring

#### 3.6 Soft Delete Flag 🗑️
- **Purpose**: Indicates record is logically deleted but retained for audit/history
- **Examples**: `is_deleted`, `is_archived`, `is_inactive`
- **Profiling**: Deletion rate over time, retention analysis
- **Quality Rules**:
  - Should be NOT NULL (default FALSE)
  - Deleted records should not be referenced by active records
  - May require related timestamp field (deleted_at)
- **Valid Aggregations**: COUNT, PROPORTION
- **Analysis**: Data retention patterns, deletion trends, filter condition for active-only queries
- **Anti-pattern**: Using NULL to indicate not-deleted; prefer explicit FALSE

---

### 4. TEMPORAL Types 📅

Date and time related fields.

#### 4.1 Event Timestamp ⚡
- **Purpose**: When a specific event/action/transition occurred, basis logical time series, externally given
- **Examples**: `published`, `completed`, `created_at`, `offer_placed_at`
- **Specifiers**: Standard event class: primary (timeline/calendar spine), created, modified, activated, deactivated, deleted, resolution (year, quarter, month,  .. ns), timezone, no-gregorian calendar (eg Hijri)
- **Profiling**: Time range, gaps, temporal distribution, recency
- **Quality Rules**:
  - No future dates
  - Reasonable date ranges for past, within system lifetime
  - Temporal ordering (e.g., created < published < completed) according to expected state transitions
- **Valid Aggregations**: MIN, MAX, COUNT of unique events by time bucket
- **Visualization**: Time series, event timelines, Gantt charts
- **Analysis**: Event frequency, time between events, seasonality
- **Lineage**: Event sequences must be consistent across tables

#### 4.2 Deadline/Target Timestamp 🕐
- **Purpose**: When something is expected/due to happen
- **Examples**: `offer_deadline_datetime`, `acknowledgement_required_until`
- **Specifiers**: Resolution (year, quarter, month,  .. ns), timezone
- **Profiling**: Past vs future deadlines, deadline miss rate
- **Quality Rules**:
  - Should be >= creation timestamp
  - Flag overdue/missed deadlines
  - Reasonable range in the future, compared to creation timestamp
- **Valid Aggregations**: MIN, MAX, COUNT
- **Visualization**: Timeline with actual vs deadline comparison
- **Analysis**: On-time performance, deadline pressure patterns

#### 4.3 Time Window/Range
- **Purpose**: Start or end of a time periods, as pair of timestamps
- **Examples**: `loading_start_ts`, `loading_end_ts`
- **Specifiers**: Resolution (year, quarter, month,  .. ns), timezone
- **Profiling**: Window duration, overlap detection
- **Quality Rules**:
  - Start < End (when paired)
  - Non-negative duration
  - Reasonable duration ranges
- **Valid Aggregations**: Duration calculations (end - start)
- **Visualization**: Gantt charts, duration histograms
- **Analysis**: Window utilization, buffer analysis

#### 4.4 Technical/System Timestamp
- **Purpose**: When record was processed/ingested by system, internally generated, usually in UTC
- **Examples**: `processed_ts`, `ingestion_partition_ts`, `ingested`
- **Profiling**: Data freshness, processing lag, ingestion patterns
- **Specifiers**: time resolution, e.g. ms, ns
- **Quality Rules**:
  - Should be recent (freshness check)
  - Should be >= source event timestamps
- **Valid Aggregations**: MAX (for freshness), date buckets
- **Visualization**: Data freshness dashboards, ingestion monitoring

#### 4.5 Timezone Indicator ⏰
- **Purpose**: Specifies timezone for associated timestamp, e.g. Europe/Berlin
- **Anti-pattern**: Not to be confused with timezone offsets to GMT (e.g. GMT +02:00), or tz abbrevations (e.g. CET), which vary depending on DST
- **Examples**: `loading_time_zone`, `unloading_time_zone`
- **Profiling**: Timezone distribution, consistency with location
- **Quality Rules**:
  - Valid IANA timezone identifiers
  - Consistent with geographic location (country)
- **Valid Aggregations**: COUNT, GROUP BY
- **Visualization**: Geographic timezone maps

---

### 5. GEOGRAPHIC Types 📍

Location and spatial data.

#### 5.1 Coordinate (Latitude) 🌐
- **Purpose**: Latitude component of geographic point
- **Examples**: `loading_lat_dlv`, `unloading_lat_dlv`
- **Profiling**: Coordinate distribution, validity, precision
- **Quality Rules**:
  - Not missing
  - Must be between -90 and 90, suspicious if outside -70 to 75
  - Reasonable precision: not too little or too many decimal places, usually 4-5
  - Not (0, 0) in combination with Longitude
- **Valid Aggregations**: AVG (for centroid), MIN, MAX (for bounding box)
- **Visualization**: Maps (requires paired longitude)
- **Analysis**: Geographic clustering, distance calculations
- **Lineage**: Must pair with compatible longitude field
- **Unit**: Decimal degrees (WGS84)

#### 5.2 Coordinate (Longitude) 🌐
- **Purpose**: Longitude component of geographic point
- **Examples**: `loading_lon_dlv`, `unloading_lon_dlv`
- **Profiling**: Coordinate distribution, validity, precision
- **Quality Rules**:
  - Not missing
  - Must be between -180 and 180
  - Reasonable precision: not too little or too many decimal places, usually 4-5
  - Not (0, 0) in combination with Latitude
- **Valid Aggregations**: AVG (for centroid), MIN, MAX (for bounding box)
- **Visualization**: Maps (requires paired latitude)
- **Lineage**: Must pair with compatible latitude field
- **Unit**: Decimal degrees (WGS84)

#### 5.3 Address Text
- **Purpose**: Street address component, usually with house number
- **Examples**: `loading_street`, `unloading_street_dlv`
- **Profiling**: Completeness, format patterns, geocoding coverage
- **Quality Rules**:
  - Non-empty when address is required
  - Format consistency checks
  - Geocoding success: street, house level or none 
- **Valid Aggregations**: COUNT DISTINCT (location counting)
- **Visualization**: Address frequency lists, street/house level maps
- **Analysis**: Location frequency, address quality scoring

#### 5.3.1 Address Addendum
- **Purpose**: Supplementary address information (building, floor, unit, c/o)
- **Examples**: `address_addendum`, `address_line_2`, `building_name`, `floor`
- **Profiling**: Completeness relative to main address, common patterns
- **Quality Rules**:
  - Should only be populated when main address exists
  - No duplication of main address content
- **Related Fields**: Must be used with primary Address Text field

#### 5.4 City Name
- **Purpose**: City/municipality name
- **Examples**: `loading_city`, `unloading_city`
- **Profiling**: City distribution, spelling variants
- **Quality Rules**:
  - Geocoding success
  - Consistent spelling
  - Consistent with country/region
- **Valid Aggregations**: COUNT, GROUP BY
- **Visualization**: City-level maps, city frequency charts
- **Analysis**: City-to-city flows, urban vs rural patterns

#### 5.5 Postal Code
- **Purpose**: Postal/ZIP code
- **Specifiers**: country code, namespace
- **Examples**: `loading_zip`, `unloading_zip_dlv`
- **Derivates**: most countries havehierarchical post code system, enabling to use 1-2-3 first chars as general region identifier 
- **Profiling**: Format by country, coverage, invalid codes
- **Quality Rules**:
  - Format matches given country standards
  - Valid code for specified country
  - Geocoding success: region, point level or none
- **Valid Aggregations**: COUNT, GROUP BY, GROUP BY larger regions
- **Visualization**: Postal code heat maps
- **Analysis**: Postal zone analysis, delivery density, lane (source-destination postal code regions) distribution

#### 5.6 Spatial Index/Hash
- **Purpose**: Spatial identifier (e.g., H3, S2, Geohash) for e.g. indexing
- **Specifiers**: indexing method, index resolution level
- **Index Level**: most systems require resolution level, e.g. level 10 for H3 means about 75 meters hexagons
- **Examples**: `start_h3_10`, `end_h3_10`, `location_h3_11`
- **Profiling**: Resolution consistency, coverage, cell distribution
- **Quality Rules**:
  - Valid hash format for system (H3/S2/Geohash)
  - Consistent resolution level
- **Valid Aggregations**: COUNT, GROUP BY (for spatial aggregation)
- **Visualization**: Hexbin maps, spatial heatmaps
- **Analysis**: Spatial clustering, proximity analysis
- **Lineage**: Must be derived from lat/lon pairs

#### 5.7 Country/Region Code 🌍
- **Purpose**: ISO country or region codes
- **Specifier**: Subtype : Country (2-char), Country (3-char), Continent, Region, Country+Region
- **Examples**: `loading_country_tour`, `loading_country_dlv`, `l2_loading_region`
- **Profiling**: Geographic distribution, code validity
- **Quality Rules**:
  - Validate against respective reference data like ISO-3166-1 (2-char, 3-char or numeric) for countries, ISO 3166-2 for regions within countries
  - Check for historical codes (ISO 3166-3)
  - Check for invalid codes (not given, not current nor historical)
  - Consistent format (suggested convention: all uppercase)
- **Valid Aggregations**: COUNT, GROUP BY, GROUP BY higher level (e.g. continents from countries, countries from regions)
- **Visualization**: Thematic maps, geo bar charts
- **Analysis**: Geographic concentration, cross-border patterns
- **Lineage**: Region/country codes must be compatible across tables - same coding and resolution

#### 5.8 Location Code (e.g. UN/LOCODE, Plus Code / OLC etc)
- **Purpose**: Standardized location identification using codified schemes for ports, logistics hubs, addresses, or arbitrary precise points.
- **Specifiers**: Code system (e.g., UN/LOCODE, Plus Code/OLC, proprietary), GLN, region/country
- **Examples**: `loading_unlocode`, `delivery_pluscode`
- **Derivates**: Regional or facility lookup, mapping to lat/lon, hierarchical breakdown (e.g., port vs city vs facility)
- **Profiling**: Coverage, code system mix, invalid or obsolete codes, mapping success to physical locations
- **Quality Rules**:
  - Correct format per code system (e.g., UN/LOCODE: 5 chars; Plus Code: 10/11 chars)
  - Valid/recognized code in the system’s reference list
  - Geocoding accuracy: code corresponds to expected countr/region/coordinates
  - Consistent code system usage for the field or dataset
- **Valid Aggregations**: COUNT, GROUP BY, code prefix grouping (e.g., country or region part of code)
- **Visualization**: Logistics/port maps, frequency by location code, hub analysis
- **Analysis**: Movement through key nodes, port-level aggregation, code system harmonization, location precision checks
- **Lineage**: Traceability back to original source (e.g., carrier input, address geocoding, or facility registry lookup)



---

### 6. NUMERIC MEASUREMENT Types 📊

Quantitative measurements with units.

#### 6.1 Count/Quantity 🔢
- **Purpose**: Discrete count of items/events
- **Examples**: `deliveries_n`, `offers_n`, `hazardous_deliveries_n`
- **Profiling**: Distribution, zero counts, outliers
- **Quality Rules**:
  - Must be >= 0 (non-negative)
  - Integer values only
  - Check for unrealistic outliers
- **Valid Aggregations**: SUM, AVG, MIN, MAX, COUNT
- **Visualization**: Histograms, bar charts
- **Analysis**: Totals, averages, distribution analysis
- **Unit**: Count (dimensionless)

#### 6.2 Distance Metric 🛣️
- **Purpose**: Linear distance measurement
- **Examples**: `distance_aerial_km`, `rtv_distance_km`
- **Specifier**: distance type: SFD (shortest feasible route distance), aerial
- **Derivates**: translations to other distance units
- **Profiling**: Range, outliers, zero values, distribution
- **Quality Rules**:
  - Must be >= 0
  - Realistic maximum (e.g., <50,000 km for road transport)
  - Aerial distance <= actual route distance
- **Valid Aggregations**: SUM, AVG, MIN, MAX, PERCENTILE
- **Visualization**: Histograms, distance bands
- **Analysis**: Total distance, average haul, distance-based segmentation, Pareto analysis
- **Unit**: Kilometers (or specified: m, nm, mi)
- **Lineage**: Calculated distances should match source coordinates

#### 6.3 Duration Metric ⏱️
- **Purpose**: Time duration/elapsed time
- **Examples**: `transport_duration_minutes`, `route_time_s`
- **Profiling**: Duration distribution, outliers, zero durations
- **Derivates**: translations to other period units, conversion to human-friendly forms like 'Day:Hour:Min'
- **Quality Rules**:
  - Must be >= 0
  - Reasonable maximum (e.g., <10,000 minutes)
  - Consistent with distance (speed checks)
- **Valid Aggregations**: SUM, AVG, MIN, MAX, PERCENTILE
- **Visualization**: Duration histograms, box plots
- **Analysis**: Total time, average duration, efficiency metrics
- **Unit**: Days, minutes, seconds, hours (specify)

#### 6.4 Volume Metric 📦
- **Purpose**: 3D space (e.g. cargo) volume
- **Examples**: `transport_volume_cbm`
- **Profiling**: Distribution, capacity utilization, zero values
- **Quality Rules**:
  - Must be >= 0
  - Realistic maximum (e.g., <150 CBM for trucks)
- **Valid Aggregations**: SUM, AVG, MIN, MAX
- **Visualization**: Volume distribution, capacity charts
- **Analysis**: Total volume, utilization rates
- **Unit**: Cubic meters (CBM)

#### 6.5 Weight Metric ⚖️
- **Purpose**: Mass/weight measurement
- **Examples**: `transport_weight_kg`, `order_weight_kg`
- **Profiling**: Distribution, overload detection, zero values
- **Quality Rules**:
  - Must be >= 0
  - Realistic maximum (e.g., <40,000 kg for single truckload)
  - Consistent with volume (density checks)
- **Valid Aggregations**: SUM, AVG, MIN, MAX
- **Visualization**: Weight distribution, load charts
- **Analysis**: Total tonnage, weight-based pricing
- **Unit**: Kilograms (KG), Metric Tonnes, LBS - must be given as Specifier
- **Lineage**: RTV weights should align with order weights

#### 6.6 Dimension/Length Metric 📏
- **Purpose**: Linear dimension measurement
- **Examples**: `transport_loading_meter_m`
- **Profiling**: Distribution, capacity checks
- **Quality Rules**:
  - Must be >= 0
  - Realistic maximum (<100)
- **Valid Aggregations**: SUM, AVG, MIN, MAX
- **Unit**: Meters (M)

#### 6.7 Environmental/Emissions Metric 🌿
- **Purpose**: Environmental impact measurements
- **Examples**: `emissions_default_total_wtw_co2_t`, `emissions_fuel_based_accumulated_consumption_l`
- **Specifiers**: measure content, e.g. total emissions, specific emissions, noise emissions etc
- **Profiling**: Distribution, zero emissions, calculation method correlation
- **Quality Rules**:
  - Must be >= 0
  - Reasonable ranges for transport type
  - Validate calculation method consistency
- **Valid Aggregations**: SUM, AVG, PERCENTILE
- **Visualization**: Emissions trends, method comparison
- **Analysis**: Total carbon footprint, efficiency scoring
- **Unit**: Depending on metric: Tonnes (CO2e), Liters (fuel), grammes of CO2e per tonne-km

#### 6.8 Score/Index 📈
- **Purpose**: Calculated quality/performance score
- **Examples**: `rtv_visibility_index`
- **Profiling**: Score distribution, correlation with outcomes
- **Quality Rules**:
  - Within expected range (e.g., 0-100 or 0-1)
  - Validate score calculation
- **Valid Aggregations**: AVG, MIN, MAX, PERCENTILE
- **Visualization**: Score distribution, trends over time
- **Analysis**: Performance benchmarking, data or business service/product quality monitoring

#### 6.9 Price/Amount 💰
- **Purpose**: Monetary values and pricing
- **Examples**: `basic_price_eur`, `selected_offer_price_eur`, `min_offer_price_eur`
- **Specifiers**: Currency (EUR, USD etc), tax type (eg VAT)
- **Profiling**: Price distribution, outliers, zero prices, pricing patterns
- **Quality Rules**:
  - Usually >= 0 (unless credits/refunds allowed)
  - Check for unrealistic values
  - Validate min <= selected <= max (when applicable)
  - Currency consistency
- **Valid Aggregations**: SUM, AVG, MIN, MAX, PERCENTILE
- **Visualization**: Price histograms, box plots, pricing trends
- **Analysis**: Total revenue, average price, price optimization
- **Lineage**: Calculated prices should trace to component prices

---

### 8. TEXT Types 📄

General textual data with various purposes.

#### 8.1 Free Text/Notes
- **Purpose**: Unstructured human-readable text
- **Specifiers**: Language: e.g. EN, unspecified, auto-detectable
- **Examples**: Comments, notes, descriptions (if present)
- **Anti-patterns**: Usage of free text when higher order type (eg Category) can be used
- **Profiling**: Length distribution, completeness, language detection
- **Quality Rules**:
  - Character encoding validation
  - Length limits
- **Valid Aggregations**: COUNT (non-null), LENGTH statistics
- **Visualization**: Word clouds, length distribution
- **Analysis**: Sentiment analysis, keyword extraction

#### 8.2 Structured Text/Chain Data 🔄
- **Purpose**: Serialized structured data (comma-separated, JSON, etc.)
- **Examples**: `freight_forwarding_company_assignment_chain`, `freight_forwarding_transport_id_chain`
- **Specifiers**: expected structure (e.g. json-schema)
- **Profiling**: Chain length, parsing success, value distribution
- **Quality Rules**:
  - Valid format/delimiter
  - Parse without errors
  - Individual components valid
- **Valid Aggregations**: Array/chain length, explode and aggregate components
- **Visualization**: Chain length distribution, flow diagrams
- **Analysis**: Chain complexity, multi-hop patterns

#### 8.3 URL/Link 🔗
- **Purpose**: Web address or resource locator
- **Examples**: `company_url`, `website`, `logo_url`, `api_endpoint`, `file_location`
- **Specifiers**: Protocol (http, https, ftp, binary data), resource type (website, API, image, pdf file)
- **Profiling**: URL validity, domain distribution, protocol usage
- **Quality Rules**:
  - Valid URL format (RFC 3986)
  - Protocol should be https for security-sensitive links
  - Domain should be reachable (optional liveness check)
  - No credentials embedded in URL
- **Valid Aggregations**: COUNT (non-null), COUNT DISTINCT domains
- **Visualization**: Domain frequency charts
- **Analysis**: Domain concentration, protocol security audit

---

### 9. ENTITY & BUSINESS DATA Types 🏢

Data about legal entities, organizations, and business operations. Generally **non-PII** as this is publicly available business information.

#### 9.1 Legal Entity Name
- **Purpose**: Name of a legal entity (company, organization, location, product)
- **PII Classification**: Non-PII (publicly available business information)
- **Examples**: `company_name`, `shipper_name`, `carrier_name`, `location_name`, `organization_name`, `trading_name`
- **Specifiers**: Language, character set (Latin, Cyrillic, etc.), name type (legal, trading, display)
- **Profiling**: Length distribution, character set analysis, duplicate/similar name detection
- **Quality Rules**:
  - Non-empty for required entities
  - Character encoding validation (UTF-8)
  - Watch for patterns indicating test/demo data (e.g., "Test", "Demo", "Example")
  - Consistent capitalization
  - Legal entity suffix validation (GmbH, Ltd, Inc, etc.)
- **Valid Aggregations**: COUNT DISTINCT
- **Visualization**: Text lists, search/filter fields, word clouds
- **Analysis**: Name standardization, duplicate detection, test data identification, company registry matching
- **Related Subtypes**: May have latinized/transliterated variant field (e.g., `latin_name`)

#### 9.2 External Registration Number 🔢
- **Purpose**: Official registration/identification numbers from external authorities for legal entities
- **PII Classification**: Non-PII (public business registration)
- **Examples**: `ogrn` (Russian company ID), `vat_number`, `duns_number`, `tax_id`, `eori_number`
- **Specifiers**: Country/jurisdiction, issuing authority, format pattern
- **Profiling**: Format validation, country consistency, uniqueness
- **Quality Rules**:
  - Format matches expected pattern for type (e.g., OGRN is 13 or 15 digits)
  - Check digit validation where applicable
  - Consistent with entity's country/jurisdiction
  - Unique within namespace
- **Valid Aggregations**: COUNT, COUNT DISTINCT
- **Analysis**: Registration coverage, format compliance, country distribution

#### 9.3 Business Address 🏠
- **Purpose**: Physical address of a business entity or operational location
- **PII Classification**: Non-PII (publicly available business information)
- **Examples**: `company_address`, `warehouse_address`, `office_address`, `registered_address`, `loading_address`
- **Specifiers**: Address type (registered, operational, warehouse), address components
- **Profiling**: Geocoding validation, address standardization, completeness
- **Quality Rules**:
  - Required components present (street, city, postal code, country)
  - Postal code format valid for country
  - Country code standardized (ISO 3166)
  - Geocodable to valid coordinates
  - Consistent with business registration (for registered address)
- **Valid Aggregations**: COUNT, COUNT by country/region, geographic clustering
- **Visualization**: Maps, location clusters, geographic distribution
- **Analysis**: Location optimization, geographic coverage, address standardization
- **Related Types**: Links to Geospatial coordinates (latitude/longitude)

#### 9.4 Business Email 📧
- **Purpose**: Generic business email addresses (not associated with specific individuals)
- **PII Classification**: Non-PII
- **Examples**: `info_email`, `support_email`, `sales_email`, `company_email`
- **Specifiers**: Purpose (info, support, sales, billing)
- **Profiling**: Domain distribution, format validity
- **Quality Rules**:
  - Valid email format (RFC 5322)
  - Domain matches company domain
  - Role-based address pattern (info@, support@, sales@)
- **Valid Aggregations**: COUNT, COUNT DISTINCT, COUNT by domain
- **Visualization**: Domain frequency charts
- **Analysis**: Email domain concentration, role coverage

#### 9.5 Business Phone 📞
- **Purpose**: Business switchboard and department phone numbers
- **PII Classification**: Non-PII
- **Examples**: `company_phone`, `office_phone`, `support_hotline`, `fax`
- **Specifiers**: Phone type (switchboard, department, fax), country code
- **Profiling**: Country code distribution, format consistency
- **Quality Rules**:
  - Valid phone format (E.164 recommended)
  - Country code present for international use
  - Length appropriate for country
- **Valid Aggregations**: COUNT, COUNT DISTINCT, COUNT by country code
- **Visualization**: Country distribution
- **Analysis**: Geographic distribution, contact coverage

---

### 10. PERSONAL DATA Types (PII) 👤🔒

Data directly or indirectly identifying natural persons. **All types in this category are PII** and require appropriate data protection measures including masking, encryption, access control, and compliance with privacy regulations (GDPR, CCPA, etc.).

> 📋 **Standards Note**: Consider future alignment with [W3C Data Privacy Vocabulary (DPV) 2.0](https://w3id.org/dpv/pd) for machine-readable personal data categorization.

> ⚠️ **Data Protection Notice**: Fields in this category require:
> - Access control and authorization
> - Encryption at rest and in transit
> - Audit logging for access
> - Retention policies per regulations
> - Right to erasure compliance

#### 10.1 Natural Person Name 👤
- **Purpose**: Name of an individual person (first name, last name, full name)
- **PII Classification**: PII - requires masking, encryption, or access control
- **Examples**: `driver_name`, `contact_person`, `first_name`, `last_name`, `full_name`, `dispatcher_name`
- **Specifiers**: Language, character set (Latin, Cyrillic, etc.), name component (first, last, full, title)
- **Profiling**: Length distribution, character set analysis (limited to authorized personnel)
- **Quality Rules**:
  - Non-empty for required contacts
  - Character encoding validation (UTF-8)
  - Watch for placeholder values (e.g., "N/A", "Unknown", "Test User")
  - No email/phone patterns embedded in name field
- **Valid Aggregations**: COUNT DISTINCT (on masked/hashed values)
- **Data Protection**:
  - Apply masking for non-privileged access
  - Encrypt at rest
  - Log access for audit trail
  - Retention policies per GDPR/privacy regulations
- **Visualization**: Masked display (e.g., "J*** D***"), access-controlled full display
- **Analysis**: Limited to aggregated statistics; full name analysis requires privacy approval
- **Related Subtypes**: May have pseudonymized variant field (e.g., `driver_id` instead of `driver_name`)

#### 10.2 Personal Email Address 📧
- **Purpose**: Electronic mail address associated with a natural person
- **PII Classification**: PII - requires protection
- **Examples**: `driver_email`, `dispatcher_email`, `contact_email`, `user_email`
- **Specifiers**: Purpose (contact, login, notification)
- **Profiling**: Domain distribution, format validity (limited access)
- **Quality Rules**:
  - Valid email format (RFC 5322)
  - Domain exists (MX record check optional)
  - No disposable/temporary email domains for critical contacts
- **Valid Aggregations**: COUNT, COUNT DISTINCT (on hashed values)
- **Data Protection**:
  - Apply masking (e.g., "j***@***.com")
  - Encrypt at rest
  - Consent required for marketing use
  - Unsubscribe/preference management
- **Visualization**: Masked display, domain distribution (aggregated)
- **Analysis**: Limited to aggregated statistics

#### 10.3 Personal Phone Number 📱
- **Purpose**: Telephone contact number associated with a natural person
- **PII Classification**: PII - requires protection
- **Examples**: `driver_phone`, `mobile`, `contact_phone`, `emergency_contact`, `personal_mobile`
- **Specifiers**: Phone type (mobile, landline), country code, purpose (contact, SMS, emergency)
- **Profiling**: Country code distribution (aggregated only)
- **Quality Rules**:
  - Valid phone format (E.164 recommended)
  - Country code present for international use
  - Length appropriate for country
  - No obvious test patterns (e.g., 000-000-0000, 123-456-7890)
- **Valid Aggregations**: COUNT, COUNT DISTINCT (on hashed values), COUNT by country code
- **Data Protection**:
  - Apply masking (e.g., "+49 *** *** **89")
  - Encrypt at rest
  - SMS consent tracking for marketing
- **Visualization**: Country distribution (aggregated), masked display
- **Analysis**: Geographic distribution (aggregated only)

#### 10.4 Personal Address 🏠
- **Purpose**: Physical address associated with a natural person (home, delivery address)
- **PII Classification**: PII - requires protection and access control
- **Examples**: `driver_address`, `home_address`, `billing_address`, `delivery_address` (when personal)
- **Specifiers**: Address type (home, billing, delivery), address components (street, city, postal, country)
- **Profiling**: Limited to authorized personnel; country/region distribution only for analytics
- **Quality Rules**:
  - Required components present (street, city, postal code, country)
  - Postal code format valid for country
  - Country code standardized (ISO 3166)
  - No placeholder values ("TBD", "Unknown")
- **Valid Aggregations**: COUNT by country/region (aggregated), COUNT DISTINCT (on hashed values)
- **Data Protection**:
  - Apply masking for non-privileged access
  - Encrypt at rest
  - Minimize storage (delete when no longer needed)
  - Separate from business location data
- **Visualization**: Country/region heatmaps only (no precise locations)
- **Analysis**: Aggregated geographic distribution; precise analysis requires privacy approval

#### 10.5 Personal Identification Document 🪪
- **Purpose**: Government-issued identification numbers for natural persons
- **PII Classification**: **Sensitive PII** - highest protection level required
- **Examples**: `passport_number`, `national_id`, `driver_license_number`, `social_security_number`
- **Specifiers**: Document type, issuing country, expiry tracking
- **Profiling**: Strictly limited; format validation only with audit logging
- **Quality Rules**:
  - Format valid for document type and country
  - Check digit validation where applicable
  - Expiry date tracking (if stored)
  - Never log or display in plain text
- **Valid Aggregations**: COUNT (existence only), never aggregate actual values
- **Data Protection**:
  - Encrypt at rest AND in transit
  - Access requires explicit authorization and audit logging
  - Tokenize where possible (store reference, not value)
  - Retention strictly limited by legal requirements
  - Right to erasure compliance (GDPR Article 17)
- **Visualization**: Never display; show only validation status or masked indicator
- **Analysis**: Not permitted on actual values; compliance audit only
- **Legal**: Subject to specific regulations (GDPR, CCPA, sector-specific rules)

---

### 11. METADATA Types 🗂️

Technical and system metadata.

#### 11.1 Source/System Metadata 📁
- **Purpose**: Technical information about data source
- **Examples**: `source_filename`, `ingested`
- **Profiling**: Source distribution, ingestion completeness
- **Quality Rules**: Track all expected sources
- **Valid Aggregations**: COUNT, GROUP BY source
- **Visualization**: Source contribution charts
- **Analysis**: Source quality comparison, ingestion monitoring

#### 11.2 Partition Key 🗄️
- **Purpose**: Physical table partitioning key
- **Examples**: `p_is_contracted`, `p_completed_month`
- **Profiling**: Partition distribution, skew detection
- **Quality Rules**:
  - Should align with query patterns
  - Check partition skew
- **Valid Aggregations**: COUNT per partition
- **Visualization**: Partition size distribution
- **Analysis**: Query performance optimization, partition pruning

#### 11.3 Version/Revision Number 🔢
- **Purpose**: Record version for optimistic locking or change tracking
- **Examples**: `revision`, `version`, `row_version`, `etag`
- **Profiling**: Version distribution, update frequency
- **Quality Rules**:
  - Must be >= 0 (or >= 1 depending on convention)
  - Should increment on each update
  - Integer values only
- **Valid Aggregations**: MAX (for latest), AVG (for update frequency proxy)
- **Analysis**: Update frequency, data churn analysis
- **Usage**: Optimistic concurrency control, CDC (Change Data Capture)

#### 11.4 Data Origin Flag 📥
- **Purpose**: Indicates the source system or method of data creation
- **Examples**: `from_crm`, `is_api_created`, `source_system`, `creation_method`
- **Profiling**: Origin distribution, quality by origin
- **Quality Rules**:
  - Should be NOT NULL
  - Valid origin values from known systems
- **Valid Aggregations**: COUNT, GROUP BY
- **Visualization**: Pie charts, origin distribution bars
- **Analysis**: Data provenance, quality comparison by source, integration coverage

---

## Field Semantic Annotation Format

Field metadata can be saved as table column comments, dbt model column comments etc using json format.

*Semantic type has emoji prefix for easier loading.*

Values are primarily human-readable, specific sample values are preferred but there could be smart AI to read other future values them. 

```json
{
  "type": numeric.weight, // minimal
  "description": "Cargo weight in kilograms",
  "properties": //specific details depending on type
  { "unit": "kg" // for numeric data
  },
  "aggregations": ["SUM", "AVG", "MIN", "MAX"],
  "values":{ // value expectations (as soft rules)
    "min": ">0",
    "max": "<=100000",
    "nulls": true //ok to have null value
  }
}
```

### Example Annotations

```yaml
transports: # table name
  transport_id:
    db_type: INT64
    semantic_type: identifier.primary
    specifiers: 
      - numeric
      - incremental
    description: "Unique identifier for each transport"
    nullable: false
    cardinality: unique
    quality_rules:
      - NOT NULL
      - UNIQUE
      - No gaps in sequence (if auto-increment)
    valid_aggregations: ["COUNT", "COUNT DISTINCT"]
    invalid_aggregations: ["SUM", "AVG"]
    visualization: 
      - "bar chart of counts"

  first_loading_lat_dlv:
    db_type: FLOAT
    semantic_type: geographic.latitude
    description: "First loading latitude in decimal degrees"
    unit: "decimal_degrees"
    nullable: true
    cardinality: high
    related_fields: 
      - "first_loading_lon_dlv"
    quality_rules:
      - "BETWEEN -90 AND 90"
      - "NOT (0,0) unless valid"
      - "Precision <= 6 decimal places"
      - "Consistent with first_loading_country_dlv"
    valid_aggregations: 
      - "AVG for centroid"
      - "MIN/MAX for bbox"
    visualization: 
      - "map (with longitude)"
      - "scatter plot"

  route_distance_km:
    db_type: INT64
    semantic_type: numeric.distance
    description: "Total routed transport distance"
    unit: "kilometers"
    nullable: true
    cardinality: high
    related_fields: 
      - "distance_aerial_first_to_last_km"
      - "rtv_distance_km"
    quality_rules:
      - ">= 0"
      - "<= 10000 (realistic max)"
      - ">= distance_aerial_first_to_last_km (route >= aerial)"
    valid_aggregations: 
      - "SUM"
      - "AVG"
      - "MIN"
      - "MAX"
      - "PERCENTILE"
    visualization: 
      - "histogram"
      - "box plot"
      - "distance bands"
    lineage_rules:
      - "Must be derived from coordinates or routing engine"
      - "Should be >= aerial distance between endpoints"

  selected_offer_price_eur:
    db_type: FLOAT
    semantic_type: numeric.currency_amount
    description: "Selected offer price in EUR"
    properties:
      currency: EUR
    nullable: true
    cardinality: high
    related_fields: 
      - "min_offer_price_eur"
      - "max_offer_price_eur"
      - "basic_price_eur"
    quality_rules:
      - ">= 0"
      - "BETWEEN min_offer_price_eur AND max_offer_price_eur"
      - "NOT NULL when status = 'assigned'"
    valid_aggregations: 
      - "SUM for revenue"
      - "AVG"
      - "MIN"
      - "MAX"
      - "PERCENTILE"
    visualization:
      - "histogram"
      - "trend line"
      - "pricing distribution"
    analysis_patterns:
      - "Total revenue: SUM(selected_offer_price_eur)"
      - "Average price per km: AVG(selected_offer_price_eur / route_distance_km)"

  is_contracted:
    db_type: BOOL
    semantic_type: boolean.classification_flag
    description: "Whether the load is contracted"
    nullable: false
    cardinality: "low (2-3 values)"
    quality_rules:
      - "NOT NULL required"
      - "TRUE/FALSE only (no NULL unless explicitly allowed)"
    valid_aggregations: 
      - "COUNT"
      - "SUM (TRUE=1)"
      - "PROPORTION"
    visualization:
      - "pie chart"
      - "stacked bar"
      - "trend over time"
    analysis_patterns:
      - "Contract rate: SUM(is_contracted) / COUNT(*)"
      - "Compare contracted vs spot: GROUP BY is_contracted"

  first_loading_start_ts_dlv:
    db_type: TIMESTAMP
    semantic_type: temporal.time_window
    description: "Start timestamp for first loading"
    nullable: true
    cardinality: high
    related_fields:
      - "first_loading_end_ts_dlv"
      - "loading_time_zone_dlv"
    quality_rules:
      - ">= created_at (loading can't be before order creation)"
      - "< first_loading_end_ts_dlv (start before end)"
      - "Reasonable future date (within planning horizon)"
    valid_aggregations: 
      - "MIN"
      - "MAX"
      - "COUNT by date bucket"
    visualization:
      - "timeline"
      - "gantt chart"
      - "date histogram"
    analysis_patterns:
      - "Loading window duration: first_loading_end_ts_dlv - first_loading_start_ts_dlv"
      - "Time until loading: first_loading_start_ts_dlv - created_at"
```

---

## Semantic Compatibility Rules

### Joins & Relationships

**Compatible joins require:**
- Same semantic base type (IDENTIFIER-to-IDENTIFIER, GEOGRAPHIC-to-GEOGRAPHIC)
- Same entity domain (customer_id joins to customer_id, not to transport_id)
- Same units (km joins to km, not to miles)

**Examples:**
```sql
-- ✅ VALID: Same identifier domain
SELECT * FROM transports t
JOIN carriers c ON t.carrier_id = c.carrier_id

-- ❌ INVALID: Different identifier domains
SELECT * FROM transports t  
JOIN carriers c ON t.transport_id = c.carrier_id

-- ✅ VALID: Same unit system
SELECT * FROM transports t
JOIN routes r ON t.route_distance_km = r.distance_km

-- ❌ INVALID: Unit mismatch
SELECT * FROM transports t
JOIN routes r ON t.route_distance_km = r.distance_miles
```

### Calculations & Derived Fields

**Compatible calculations require:**
- Same units (can't add kg + km)
- Same measurement type (can't average prices and distances together)
- Appropriate temporal ordering (event A before event B)

**Examples:**
```sql
-- ✅ VALID: Same units
SELECT transport_weight_kg + rtv_transport_weight_kg AS total_weight

-- ❌ INVALID: Different units
SELECT transport_weight_kg + transport_volume_cbm AS invalid_sum

-- ✅ VALID: Duration calculation between compatible timestamps
SELECT first_loading_end_ts_dlv - first_loading_start_ts_dlv AS loading_duration

-- ❌ INVALID: Meaningless operation on identifiers
SELECT transport_id + carrier_id AS invalid_calculation
```

### Aggregations

**Valid aggregations by semantic type:**

| Semantic Type | Valid | Invalid |
|--------------|-------|---------|
| IDENTIFIER (primary) | COUNT, COUNT DISTINCT | SUM, AVG, MIN, MAX |
| IDENTIFIER (foreign key) | COUNT, COUNT DISTINCT | SUM, AVG |
| BOOLEAN | COUNT, SUM (as 0/1), AVG (proportion) | MIN, MAX |
| TEMPORAL | MIN, MAX, COUNT by bucket | SUM, AVG (unless converting to epoch) |
| NUMERIC (count) | ALL | - |
| NUMERIC (measurement) | ALL | - |
| FINANCIAL | SUM, AVG, MIN, MAX, PERCENTILE | - (watch for NULL handling) |
| CATEGORICAL | COUNT, MODE, GROUP BY | SUM, AVG, MIN, MAX |
| GEOGRAPHIC (coordinates) | AVG (centroid), MIN/MAX (bbox) | SUM |

---

## Data Quality Monitoring Templates

### By Semantic Type

#### IDENTIFIER Quality Checks
```sql
-- Uniqueness
SELECT field_name, COUNT(*) as duplicates
FROM table
GROUP BY field_name
HAVING COUNT(*) > 1;

-- Null check
SELECT COUNT(*) as null_count
FROM table  
WHERE field_name IS NULL;

-- Referential integrity
SELECT fk.field_name
FROM table fk
LEFT JOIN referenced_table pk ON fk.field_name = pk.field_name
WHERE pk.field_name IS NULL;
```

#### GEOGRAPHIC Quality Checks
```sql
-- Coordinate bounds
SELECT COUNT(*) as invalid_coords
FROM table
WHERE latitude NOT BETWEEN -90 AND 90
   OR longitude NOT BETWEEN -180 AND 180;

-- (0, 0) detection
SELECT COUNT(*) as zero_coords
FROM table
WHERE latitude = 0 AND longitude = 0;

-- Country consistency
SELECT COUNT(*) as mismatched_country
FROM table
WHERE country_code = 'DE'
  AND (latitude NOT BETWEEN 47 AND 55 OR longitude NOT BETWEEN 6 AND 15);
```

#### TEMPORAL Quality Checks
```sql
-- Future dates (where not expected)
SELECT COUNT(*) as future_dates
FROM table
WHERE event_timestamp > CURRENT_TIMESTAMP();

-- Temporal ordering
SELECT COUNT(*) as ordering_violations
FROM table
WHERE created_at > published
   OR published > completed;

-- Data freshness
SELECT 
  MAX(processed_ts) as last_processed,
  DATEDIFF(hour, MAX(processed_ts), CURRENT_TIMESTAMP()) as hours_stale
FROM table;
```

#### NUMERIC Quality Checks
```sql
-- Negative values (where invalid)
SELECT COUNT(*) as negative_values
FROM table
WHERE distance_km < 0;

-- Outlier detection (IQR method)
WITH stats AS (
  SELECT
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY metric_field) as q1,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY metric_field) as q3
  FROM table
)
SELECT COUNT(*) as outliers
FROM table, stats
WHERE metric_field < (q1 - 1.5 * (q3 - q1))
   OR metric_field > (q3 + 1.5 * (q3 - q1));

-- Cross-field validation
SELECT COUNT(*) as violations
FROM table
WHERE route_distance_km < distance_aerial_first_to_last_km;
```

#### FINANCIAL Quality Checks
```sql
-- Price range validation
SELECT COUNT(*) as invalid_prices
FROM table
WHERE selected_offer_price_eur < 0
   OR selected_offer_price_eur > 100000;

-- Min/Max consistency
SELECT COUNT(*) as violations
FROM table
WHERE selected_offer_price_eur NOT BETWEEN min_offer_price_eur AND max_offer_price_eur;
```

---

## Data Profiling Templates

### By Semantic Type

#### IDENTIFIER Profiling
```sql
SELECT
  'transport_id' as field_name,
  COUNT(*) as total_count,
  COUNT(DISTINCT transport_id) as distinct_count,
  COUNT(*) - COUNT(DISTINCT transport_id) as duplicate_count,
  COUNT(*) - COUNT(transport_id) as null_count,
  MIN(transport_id) as min_value,
  MAX(transport_id) as max_value
FROM table;
```

#### CATEGORICAL Profiling
```sql
SELECT
  status_field as value,
  COUNT(*) as frequency,
  COUNT(*) * 100.0 / SUM(COUNT(*)) OVER () as percentage
FROM table
GROUP BY status_field
ORDER BY frequency DESC
LIMIT 20;
```

#### GEOGRAPHIC Profiling
```sql
SELECT
  country_code,
  COUNT(*) as count,
  AVG(latitude) as avg_lat,
  AVG(longitude) as avg_lon,
  STDDEV(latitude) as stddev_lat,
  STDDEV(longitude) as stddev_lon
FROM table
WHERE latitude IS NOT NULL AND longitude IS NOT NULL
GROUP BY country_code
ORDER BY count DESC;
```

#### NUMERIC Profiling
```sql
SELECT
  'distance_km' as metric,
  COUNT(*) as total_count,
  COUNT(distance_km) as non_null_count,
  AVG(distance_km) as mean,
  STDDEV(distance_km) as std_dev,
  MIN(distance_km) as min,
  PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY distance_km) as p25,
  PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY distance_km) as median,
  PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY distance_km) as p75,
  MAX(distance_km) as max
FROM table;
```

#### TEMPORAL Profiling
```sql
SELECT
  DATE_TRUNC('month', event_timestamp) as month,
  COUNT(*) as event_count,
  MIN(event_timestamp) as first_event,
  MAX(event_timestamp) as last_event
FROM table
GROUP BY DATE_TRUNC('month', event_timestamp)
ORDER BY month;
```

---

## Visualization Recommendations

### By Semantic Type

| Semantic Type | Primary Viz | Secondary Viz | Avoid |
|--------------|-------------|---------------|-------|
| IDENTIFIER | Bar chart (counts) | Table | Pie chart, Line chart |
| CATEGORICAL (status) | Bar chart, Pie chart | Sankey (flow) | Line chart (unless over time) |
| CATEGORICAL (category) | Bar chart, Treemap | Pie chart | Line chart |
| BOOLEAN | Stacked bar, Pie | Trend line (adoption) | Scatter plot |
| TEMPORAL (event) | Time series line | Histogram (distribution) | Pie chart |
| GEOGRAPHIC (coords) | Map (choropleth/points) | Heatmap | Bar chart |
| GEOGRAPHIC (country) | Choropleth map | Bar chart | Line chart |
| NUMERIC (count) | Histogram, Bar | Box plot | Pie chart |
| NUMERIC (measurement) | Histogram, Box plot | Scatter (vs another metric) | Pie chart |
| FINANCIAL | Histogram, Trend line | Box plot, Waterfall | Pie chart for amounts |

---

## BI Analysis Validation Rules

### Aggregation Validation

**Validate that:**
1. Identifiers are not summed/averaged
2. Counts use COUNT or COUNT DISTINCT, not SUM
3. Prices use SUM for revenue, AVG for average price
4. Measurements use appropriate aggregations (SUM for totals, AVG for averages)
5. Booleans are summed only when calculating TRUE counts or proportions

### Grouping Validation

**Validate that:**
1. GROUP BY uses categorical, identifier, or temporal bucket fields
2. GROUP BY doesn't use high-cardinality text without purpose
3. Date grouping uses appropriate granularity (don't group by minute for yearly reports)

### Join Validation

**Validate that:**
1. Joins are on semantically compatible fields
2. Foreign keys join to primary keys of the same entity
3. Units match across joined tables
4. Geographic fields join at the same resolution level

### Filter Validation

**Validate that:**
1. Temporal filters use appropriate date ranges
2. Numeric filters don't exclude valid values
3. Boolean filters handle NULL appropriately
4. Categorical filters use valid values from the domain

---

## Usage Examples

### Example 1: Validate a query for semantic correctness

```sql
-- Query to validate
SELECT
  transport_id,
  AVG(carrier_id) as avg_carrier,  -- ❌ INVALID: Can't average identifiers
  SUM(is_contracted) as contract_count,  -- ✅ VALID: Boolean sum for TRUE count
  MIN(published) as first_publish,  -- ✅ VALID: MIN on temporal event
  SUM(route_distance_km) / COUNT(*) as avg_distance  -- ✅ VALID: Proper average calculation
FROM transports
GROUP BY transport_id;
```

**Semantic validation findings:**
- ❌ `AVG(carrier_id)`: IDENTIFIER fields should not be averaged (use COUNT DISTINCT instead)
- ✅ `SUM(is_contracted)`: Valid boolean aggregation for counting TRUE values
- ✅ `MIN(published)`: Valid temporal aggregation
- ✅ Manual average calculation: Valid, though AVG(route_distance_km) is simpler

### Example 2: Suggest semantic improvements

```sql
-- Original query
SELECT
  COUNT(*) as total,
  SUM(CASE WHEN is_rtv_enabled = TRUE THEN 1 ELSE 0 END) as rtv_count
FROM transports;

-- Semantically optimized
SELECT
  COUNT(*) as total_transports,
  SUM(is_rtv_enabled::INT) as rtv_enabled_count,
  AVG(is_rtv_enabled::INT) as rtv_enabled_proportion
FROM transports;
```

### Example 3: Generate data quality report

```sql
-- Based on semantic types, generate comprehensive quality report
WITH quality_checks AS (
  SELECT
    -- IDENTIFIER checks
    COUNT(DISTINCT transport_id) = COUNT(*) as transport_id_unique,
    COUNT(transport_id) = COUNT(*) as transport_id_not_null,
    
    -- GEOGRAPHIC checks
    SUM(CASE WHEN first_loading_lat_dlv NOT BETWEEN -90 AND 90 THEN 1 ELSE 0 END) as invalid_latitude_count,
    SUM(CASE WHEN first_loading_lon_dlv NOT BETWEEN -180 AND 180 THEN 1 ELSE 0 END) as invalid_longitude_count,
    
    -- NUMERIC checks
    SUM(CASE WHEN route_distance_km < 0 THEN 1 ELSE 0 END) as negative_distance_count,
    SUM(CASE WHEN route_distance_km < distance_aerial_first_to_last_km THEN 1 ELSE 0 END) as distance_violation_count,
    
    -- TEMPORAL checks
    SUM(CASE WHEN published < created_at THEN 1 ELSE 0 END) as temporal_ordering_violations,
    SUM(CASE WHEN published > CURRENT_TIMESTAMP() THEN 1 ELSE 0 END) as future_publish_dates,
    
    -- FINANCIAL checks
    SUM(CASE WHEN selected_offer_price_eur < 0 THEN 1 ELSE 0 END) as negative_price_count,
    SUM(CASE WHEN selected_offer_price_eur NOT BETWEEN min_offer_price_eur AND max_offer_price_eur THEN 1 ELSE 0 END) as price_range_violations
    
  FROM transports
)
SELECT * FROM quality_checks;
```

---

## Semantic Type Application Guide

### Step 1: Identify Base Type
Determine the primary semantic category from the 9 main types.

### Step 2: Add Subtypes
Refine with specific subtype (e.g., IDENTIFIER → foreign_key).

### Step 3: Add Specifiers, Descriptors
Include additional semantic properties for fields:
- Entity domain: customer, transport, carrier, etc.
- Data source: calculated, measured, user_input, system_generated
- Mutability: immutable, append_only, mutable
- Sensitivity: pii, financial, public

### Step 4: Define Relationships
List related fields that must be semantically compatible.

### Step 5: Specify Quality Rules
Define validation rules based on semantic type.

### Step 6: Document Usage
Specify valid aggregations, visualizations, and analysis patterns.

---

## Implementation Checklist

- [ ] Annotate all table fields with semantic types
- [ ] Define quality rules per semantic type
- [ ] Create data quality monitoring queries
- [ ] Build data profiling templates
- [ ] Document field relationships and lineage requirements
- [ ] Configure BI tool to validate semantic correctness
- [ ] Create semantic type validation tests in dbt
- [ ] Document visualization recommendations per field
- [ ] Train analysts on semantic type system
- [ ] Integrate semantic metadata into data catalog

---

## Maintenance

**Review semantic types when:**
- New fields are added to tables
- Field meanings or business rules change
- Data quality issues are discovered
- New analysis patterns emerge
- Lineage requirements change

**Update this document:**
- Add new semantic subtypes as needed
- Refine quality rules based on real data issues
- Expand analysis patterns based on common queries
- Document edge cases and exceptions
