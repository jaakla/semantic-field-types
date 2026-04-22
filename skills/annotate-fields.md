# Annotate Fields with Semantic Types

Analyze the provided database table schema, field list, or CREATE TABLE statement and annotate each field with the appropriate semantic type from the [Semantic Field Types](https://github.com/jaakla/semantic-field-types) taxonomy.

## Semantic Type Reference

### 1. `identifier.*` — Unique entity references
| Subtype | When to use | Examples |
|---------|-------------|---------|
| `identifier.primary` | Auto-assigned unique record ID | `transport_id`, `order_id`, `customer_id` |
| `identifier.natural` | Human-readable business domain ID | `order_number`, `tracking_code`, `invoice_ref` |
| `identifier.foreign_key_master` | Reference to master data table | `carrier_id`, `shipper_id`, `product_id` |
| `identifier.foreign_key_fact` | Reference to fact/transaction table | `assigned_offer_id`, `delivery_id` |
| `identifier.composite_key` | Part of multi-column unique key | `customer_id + alphanumeric_number` |
| `identifier.self_referencing` | Hierarchical/tree structure key | `main_company_id`, `parent_location_id` |

### 2. `categorical.*` — Discrete enumerable values
| Subtype | When to use | Examples |
|---------|-------------|---------|
| `categorical.status` | Workflow or lifecycle state | `order_status`, `delivery_state`, `final_status` |
| `categorical.category` | Classification or grouping | `vehicle_type`, `product_category`, `creation_mode` |
| `categorical.reason` | Why something happened | `decline_reason`, `cancellation_reason` |
| `categorical.method` | How something was done | `fuel_type`, `payment_method`, `assignment_method` |
| `categorical.discriminator` | Polymorphic entity type indicator | `company_type`, `user_type`, `record_type` |
| `categorical.support_phase` | Operational/support tier or level | `support_phase`, `block_level`, `subscription_tier` |

### 3. `boolean.*` — True/false flags
| Subtype | When to use | Examples |
|---------|-------------|---------|
| `boolean.feature_flag` | Feature/service enablement | `is_rtv_enabled`, `has_tracking` |
| `boolean.classification` | Binary category membership | `is_contracted`, `is_domestic` |
| `boolean.derived` | Computed from logic/thresholds | `offer_deadline_passed`, `late_delivery` |
| `boolean.inheritance` | Inherited from parent in hierarchy | `from_main_bankdata`, `from_main_address` |
| `boolean.communication_preference` | Opt-in/opt-out consent flags | `gets_email`, `opt_in_marketing` |
| `boolean.soft_delete` | Logical deletion marker | `is_deleted`, `is_archived`, `is_inactive` |

### 4. `temporal.*` — Dates and times
| Subtype | When to use | Examples |
|---------|-------------|---------|
| `temporal.event` | When a real-world event occurred | `created_at`, `delivered_at`, `offer_placed_at` |
| `temporal.deadline` | Expected or required completion time | `delivery_eta`, `offer_deadline`, `payment_due_date` |
| `temporal.time_window` | Start/end of a time interval (pair) | `loading_start_ts`, `loading_end_ts` |
| `temporal.system` | Data processing/ingestion timestamp | `ingested_at`, `updated_at`, `etl_loaded_at` |
| `temporal.timezone` | Timezone for associated timestamp | `loading_time_zone`, `delivery_timezone` |

### 5. `geographic.*` — Location and spatial data
| Subtype | When to use | Examples |
|---------|-------------|---------|
| `geographic.coordinate` | Decimal degree lat/lon (WGS84) | `latitude`, `longitude`, `pickup_lat` |
| `geographic.address` | Street address with house number | `street_address`, `delivery_address_line` |
| `geographic.address_addendum` | Supplementary address detail | `address_line_2`, `building_name`, `floor` |
| `geographic.city` | Municipality name | `city`, `pickup_city`, `delivery_city` |
| `geographic.postal_code` | ZIP/postal code | `postal_code`, `zip_code`, `postcode` |
| `geographic.spatial_index` | H3 / Geohash / S2 cell ID | `h3_index`, `h3_10`, `geohash` |
| `geographic.country_code` | ISO 3166 country or region code | `country_code`, `origin_country`, `iso2` |
| `geographic.location_code` | UN/LOCODE, IATA, or similar codes | `unlocode`, `airport_code`, `port_code` |

### 6. `numeric.*` — Physical measurements with units
| Subtype | When to use | Examples |
|---------|-------------|---------|
| `numeric.count` | Discrete item/event counts | `deliveries_n`, `stop_count`, `pallet_count` |
| `numeric.distance` | Linear distance | `distance_km`, `aerial_distance_km`, `route_length_m` |
| `numeric.duration` | Elapsed time | `transport_duration_minutes`, `dwell_time_hours` |
| `numeric.volume` | Three-dimensional space | `transport_volume_cbm`, `cargo_volume_m3` |
| `numeric.weight` | Mass | `transport_weight_kg`, `cargo_weight_tons` |
| `numeric.dimension` | Object length/width/height | `pallet_height_m`, `cargo_length_cm` |
| `numeric.emissions` | CO2 / environmental impact | `emissions_total_co2_t`, `co2_kg` |
| `numeric.temperature` | Thermal measurements | `cargo_temperature_c`, `ambient_temp_f` |

### 7. `score.*` — Ratings, ranks, and ordinal values
| Subtype | When to use | Examples |
|---------|-------------|---------|
| `score.performance` | Operational quality metric (ratio/%) | `on_time_rate`, `fill_rate`, `performance_index` |
| `score.rating` | Subjective quality rating | `customer_rating`, `driver_stars`, `nps_score` |
| `score.composite_index` | Multi-factor aggregated score | `risk_score`, `health_score`, `reliability_index` |
| `score.rank` | Relative position within a group | `carrier_rank`, `supplier_rank` |
| `score.sequence` | Ordinal position in an ordered set | `stop_sequence`, `step_number`, `line_order` |
| `score.priority` | Processing importance or urgency | `priority_level`, `urgency_code` |

### 8. `financial.*` — Monetary values
| Subtype | When to use | Examples |
|---------|-------------|---------|
| `financial.price` | Per-unit price or cost rate | `basic_price_eur`, `rate_per_km`, `unit_cost` |
| `financial.amount` | Total transaction monetary value | `invoice_amount_eur`, `revenue`, `order_value` |
| `financial.budget` | Financial cap or planned amount | `transport_budget_eur`, `credit_limit` |

### 9. `text.*` — Textual content
| Subtype | When to use | Examples |
|---------|-------------|---------|
| `text.free_text` | Unstructured human-written notes | `notes`, `comments`, `description`, `remarks` |
| `text.structured_text` | Serialized structured data as text | `route_chain`, `status_history_json`, `config_blob` |
| `text.url` | Web address or resource locator | `document_url`, `image_link`, `report_url` |

### 10. `entity.*` — Business / legal entity data (Non-PII)
| Subtype | When to use | Examples |
|---------|-------------|---------|
| `entity.legal_name` | Official company or org name | `company_name`, `carrier_legal_name` |
| `entity.registration_number` | Government-issued business ID | `vat_number`, `duns_number`, `tax_id`, `ogrn` |
| `entity.business_address` | Physical business location | `company_address`, `registered_address` |
| `entity.business_email` | Role-based / departmental email | `info@company.com`, `support@`, `logistics@` |
| `entity.business_phone` | Business switchboard / dept phone | `company_phone`, `support_line` |

> **Note:** Role-based emails (`info@`, `support@`) are NOT PII. Named personal emails (`john.doe@`) ARE PII — use `personal.email` instead.

### 11. `personal.*` — Personal data / PII ⚠️ Privacy protection required
| Subtype | PII level | When to use | Examples |
|---------|-----------|-------------|---------|
| `personal.name` | PII | Individual person's name | `driver_name`, `contact_person`, `full_name` |
| `personal.email` | PII | Person-identifying email address | `driver_email`, `user_email`, `personal_email` |
| `personal.phone` | PII | Personal phone/mobile number | `driver_phone`, `personal_mobile` |
| `personal.address` | PII | Home / residential address | `home_address`, `billing_address` (person) |
| `personal.id_document` | **Sensitive PII** | Government-issued ID numbers | `passport_number`, `national_id`, `ssn`, `driver_license` |

### 12. `metadata.*` — Technical system metadata
| Subtype | When to use | Examples |
|---------|-------------|---------|
| `metadata.source` | Data source or originating system | `source_system`, `source_file`, `etl_job_name` |
| `metadata.partition_key` | Table partitioning field | `partition_date`, `event_date`, `load_date` |
| `metadata.version` | Record version for CDC/SCD | `record_version`, `row_version`, `scd_version` |
| `metadata.origin_flag` | Source system or migration indicator | `migrated_from`, `legacy_indicator` |

---

## Aggregation Rules

| Type group | Valid aggregations | Invalid aggregations |
|------------|-------------------|---------------------|
| `identifier.*` | COUNT, COUNT_DISTINCT | SUM, AVG, MIN, MAX |
| `categorical.*`, `boolean.*` | COUNT, GROUP_BY, MODE | SUM, AVG |
| `temporal.event` | MIN, MAX, COUNT, DATE_TRUNC | SUM, AVG |
| `numeric.*` | SUM, AVG, MIN, MAX, MEDIAN | — |
| `financial.*` | SUM (amount), AVG/MIN/MAX (price) | SUM on price alone |
| `score.rank`, `score.sequence` | MIN, MAX, COUNT | SUM, AVG |
| `text.*` | COUNT, LENGTH stats | SUM, AVG, MIN, MAX |
| `personal.*`, `entity.*` | COUNT, COUNT_DISTINCT | SUM, AVG, MIN, MAX |

---

## Quality Rules by Type

| Type | Key rules |
|------|-----------|
| `identifier.primary` | NOT NULL, UNIQUE, immutable |
| `identifier.foreign_key_master` | Referential integrity (strict — referenced row must exist) |
| `identifier.foreign_key_fact` | Referential integrity (forgiving — row may arrive later) |
| `categorical.*` | Values from known set; check for unexpected values |
| `boolean.*` | Prefer NOT NULL with explicit FALSE default |
| `temporal.event` | Valid date range; not future-dated unless expected |
| `temporal.deadline` | Must be after creation timestamp |
| `temporal.time_window` | Start must be before end |
| `geographic.coordinate` | Lat: −90 to 90; Lon: −180 to 180; reject (0,0) |
| `geographic.country_code` | Must be valid ISO 3166-1 alpha-2 or alpha-3 |
| `numeric.distance`, `numeric.weight`, `numeric.volume` | Must be > 0 |
| `numeric.count` | Must be ≥ 0; integer expected |
| `financial.*` | ≥ 0; valid ISO 4217 currency |
| `personal.*` | Masking/encryption required; access control; audit log |
| `personal.id_document` | Strong encryption; strict access; minimum retention |

---

## Task

For each field in `$ARGUMENTS`:

1. **Select** the most specific matching semantic type from the taxonomy above.
2. **Add `pii_classification`** (`pii` or `sensitive_pii`) for any `personal.*` type.
3. **Add `properties.unit`** for `numeric.*` and `financial.*` fields.
4. **Write a concise description** (one sentence, business-meaningful).
5. **Output** in the requested format. If none specified, produce **both** dbt YAML and JSON column-comment formats.

### dbt YAML output
```yaml
columns:
  - name: transport_weight_kg
    meta:
      semantic_type: numeric.weight
      properties:
        unit: kg
    description: "Gross cargo weight in kilograms."

  - name: driver_name
    meta:
      semantic_type: personal.name
      pii_classification: pii
      data_protection: masked
    description: "Full name of the assigned driver — PII, masked in reporting."
```

### JSON column-comment output
```json
{"type":"numeric.weight","description":"Gross cargo weight in kilograms.","properties":{"unit":"kg"},"aggregations":["SUM","AVG","MIN","MAX"],"values":{"min":">0","max":"<=40000","nulls":true}}
```

If the input is ambiguous or a field could match multiple types, pick the primary semantic intent and note the alternative in a comment.
