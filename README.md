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
| **1. Identifiers** | Uniquely identify entities | `transport_id`, `order_number` |
| **2. Categorical** | Classification and grouping | `status`, `country_code`, `transport_type` |
| **3. Boolean** | True/false flags | `is_active`, `has_trailer` |
| **4. Temporal** | Dates, times, durations | `created_at`, `delivery_date`, `transit_time` |
| **5. Geographic** | Location data | `latitude`, `longitude`, `h3_index` |
| **6. Numeric Measurements** | Quantifiable values | `distance_km`, `weight_kg`, `price_eur` |
| **8. Text** | Textual content | `notes`, `description`, `url` |
| **9. Entity & Business Data** | Legal entities (non-PII) | `company_name`, `vat_number` |
| **10. Personal Data (PII)** | Individual person data | `driver_name`, `email`, `phone` |
| **11. Metadata** | Technical/system data | `source_file`, `partition_key` |

### Example Usage

```yaml
# In your data catalog or dbt schema.yml
columns:
  - name: transport_id
    meta:
      semantic_type: identifier.primary

  - name: delivery_date
    meta:
      semantic_type: temporal.deadline

  - name: driver_name
    meta:
      semantic_type: personal.natural_person_name
      pii_classification: pii

  - name: total_cost_eur
    meta:
    semantic_type: numeric.currency_amount
      properties:
        currency: EUR
```

## Core Principles

1. **Multiple Dimensions**: Fields can have multiple semantic aspects
2. **Inheritance**: Subtypes inherit properties from parent types
3. **Composability**: Semantic markers combine to describe complex field meanings
4. **Pragmatism**: Focus on actionable metadata that drives tooling
5. **Flexibility**: Everything in the semantic definition is optional

## Contributing

We welcome contributions! See [CONTRIBUTING.md](./CONTRIBUTING.md) for guidelines.

- **Propose new types**: Open an issue using the "New Type Proposal" template
- **Improve existing types**: Open an issue or submit a PR
- **Report issues**: Use GitHub Issues for bugs or unclear documentation

## Related Standards

This taxonomy is designed to complement existing standards:

- [Open Semantic Interchange (OSI)](https://github.com/open-semantic-interchange/OSI) - Semantic layer interoperability
- [W3C Data Privacy Vocabulary (DPV)](https://w3id.org/dpv/pd) - Personal data categories
- [Schema.org](https://schema.org) - Web semantic types
- [UN/CEFACT](https://vocabulary.uncefact.org/) - Trade and transport vocabulary

## License

This project is licensed under the MIT License - see the [LICENSE](./LICENSE) file for details.

## Version

Current version: **0.1** (Initial pre-release)

See [CHANGELOG.md](./CHANGELOG.md) for version history.
