# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

_No unreleased changes_

## [0.1.0] - 2026-01-30

### Added
- Initial semantic field types specification with 12 type categories

**Type Categories**
- **1. Identifier Types 🆔**: Primary, Natural, Foreign Key (Master/Fact), Composite Key, Self-Referencing FK
- **2. Categorical Types 📝**: Status/State, Category/Classification, Reason/Qualifier, Method/Type, Entity Discriminator, Support Phase
- **3. Boolean Types ✅**: Feature Flag, Classification Flag, Derived Status, Inheritance Flag, Communication Preference, Soft Delete
- **4. Temporal Types 📅**: Event Timestamp, Deadline/Target, Time Window/Range, Technical/System Timestamp, Timezone Indicator
- **5. Geographic Types 📍**: Coordinates (Lat/Lon), Address components, City, Postal Code, Spatial Index (H3/S2/Geohash), Country/Region Code, Location Code (UN/LOCODE)
- **6. Numeric Measurement Types 📊**: Count/Quantity, Distance, Duration, Volume, Weight, Dimension, Environmental/Emissions, Temperature
- **7. Score/Index/Ordinal Types 📈**: Performance Score, Quality Rating, Composite Index, Rank/Position, Sequence/Order, Priority/Urgency
- **8. Financial Types 💰**: Price/Unit Cost, Transaction Amount, Budget/Limit
- **9. Text Types 📄**: Free Text/Notes, Structured Text/Chain Data, URL/Link
- **10. Entity & Business Data Types 🏢**: Legal Entity Name, External Registration Number, Business Address, Business Email, Business Phone
- **11. Personal Data Types (PII) 👤🔒**: Person Name, Personal Email, Personal Phone, Personal Address, ID Documents
- **12. Metadata Types 🗂️**: Source/System Metadata, Partition Key, Version/Revision, Data Origin Flag

### Documentation
- Core principles: Multiple Dimensions, Inheritance, Composability, Pragmatism, Flexibility
- Field Semantic Annotation Format (JSON and YAML examples)
- Semantic Compatibility Rules for joins, calculations, and aggregations
- Data Quality Monitoring Templates (SQL examples per type)
- Data Profiling Templates per semantic type
- Visualization Recommendations matrix
- BI Analysis Validation Rules
- Implementation Checklist
- Known limitations (composite types, validation framework, aggregation semantics)
- Future considerations for standards alignment (W3C DPV, Schema.org, UN/CEFACT, GS1, ISO/IEC 11179)

---

## Version Numbering

- **0.x.x**: Pre-release versions, API may change
- **1.0.0**: First stable release (planned)
