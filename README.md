# Semantic Field Types

A pragmatic classification system for data fields that enables automated data profiling, quality monitoring, lineage validation, and analysis support.

## Purpose

This semantic type system helps:

- **Data Profiling**: Generate meaningful summaries based on field semantics
- **Data Quality**: Apply appropriate validation rules per semantic type
- **Lineage Validation**: Ensure semantically compatible fields are joined/related
- **BI Analysis**: Validate correct aggregations and visualizations
- **Query Generation**: Guide LLMs and analysts toward semantically valid SQL
- **Privacy Compliance**: Identify and protect PII fields systematically

## Quick Start

See the full specification: **[semantic-field-types.md](./semantic-field-types.md)**

### Type Categories

| Category | Description | Examples |
|----------|-------------|----------|
| **1. Identifiers 🆔** | Uniquely identify entities | `transport_id`, `order_number` |
| **2. Categorical 📝** | Classification and grouping | `status`, `vehicle_type`, `decline_reason` |
| **3. Boolean ✅** | True/false flags | `is_active`, `is_contracted`, `gets_email` |
| **4. Temporal 📅** | Dates, times, durations | `created_at`, `delivery_deadline`, `loading_start_ts` |
| **5. Geographic 📍** | Location data | `latitude`, `longitude`, `h3_index`, `country_code` |
| **6. Numeric Measurements 📊** | Quantifiable values | `distance_km`, `weight_kg`, `temperature_c` |
| **7. Score/Index/Ordinal 📈** | Ratings, ranks, sequences | `performance_score`, `priority_level`, `stop_sequence` |
| **8. Financial 💰** | Monetary values | `price_eur`, `invoice_amount`, `budget` |
| **9. Text 📄** | Textual content | `notes`, `description`, `url` |
| **10. Entity & Business Data 🏢** | Legal entities (non-PII) | `company_name`, `vat_number`, `company_address` |
| **11. Personal Data (PII) 👤🔒** | Individual person data | `driver_name`, `personal_email`, `phone` |
| **12. Metadata 🗂️** | Technical/system data | `source_file`, `partition_key`, `revision` |

### Example Usage

In AI prompts:
* Generate semantic field descriptions for my Snowflake table XYZ using https://github.com/jaakla/semantic-field-types/blob/master/semantic-field-types.md classificaton, so I can manually review them, and then save as to column comments.
* Give me deep data exploration analysis of my BigQuery table XYZ, taking into consideration aggregations, ranges, visualization suggestions etc from semantic field types as defined in table field comments.
* Validate lineage and consistency of my data transformations from sources to data marts, based on semantic field annotations.


In your data catalog or dbt schema.yml:
```yaml

columns:
  - name: transport_id
    meta:
      semantic_type: identifier.primary

  - name: delivery_date
    meta:
      semantic_type: temporal.deadline

  - name: driver_name
    meta:
      semantic_type: personal.name
      pii_classification: pii
      data_protection: masked

  - name: total_cost_eur
    meta:
      semantic_type: financial.price
      properties:
        currency: EUR
```

In analytical database gold/mart table, as field comment:
```json
{"type":"numeric.weight","description":"Cargo weight in kilograms","properties":{"unit":"kg"},"aggregations":["SUM","AVG","MIN","MAX"],"values":{"min":">=0","max":"<=40000","nulls":true}}
```


## Core Principles

1. **Multiple Dimensions**: Fields can have multiple semantic aspects (e.g., a field can be both a temporal event and a deadline)
2. **Inheritance**: Subtypes inherit properties from parent types
3. **Composability**: Semantic markers combine to describe complex field meanings
4. **Pragmatism**: Focus on actionable metadata that drives tooling
5. **Flexibility**: Everything in the semantic definition is optional and can be extended for specific data and project needs

## Contributing

We welcome contributions! See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

- **Propose new types**: Open an issue using the "New Type Proposal" template
- **Improve existing types**: Open an issue or submit a PR
- **Report issues**: Use GitHub Issues for bugs or unclear documentation

## Related Standards

This taxonomy is designed to complement existing standards. Future versions may align more closely with:

**Privacy & Data Protection**
- [W3C Data Privacy Vocabulary (DPV) 2.0](https://w3id.org/dpv/pd) - Machine-readable vocabulary for personal data categories
- GDPR Article 9 Special Categories - Legal basis for sensitive PII classification

**Semantic Web & Metadata**
- [Schema.org](https://schema.org) - Web semantic types (827 entity types, 1528 properties)
- Dublin Core (ISO 15836) - Core metadata elements
- ISO/IEC 11179 - International standard for metadata registries

**Domain-Specific (Logistics/Transport)**
- [UN/CEFACT](https://vocabulary.uncefact.org/) - Semantic vocabulary for international trade and transport
- GS1 Global Data Model - Supply chain identifiers (GTIN, GLN, SSCC)
- UN/LOCODE - Authoritative location codes

**Data Quality Tools**
- Great Expectations - Semantic type-based profiling
- Soda Core - Data profiling with check suggestions

## License

This project is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details.

## Version

Current version: **0.1** - Initial pre-release (January 2026)

See [CHANGELOG.md](./CHANGELOG.md) for version history.
