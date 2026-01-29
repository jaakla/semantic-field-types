# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Initial public release preparation

## [0.1.0] - 2026-01-29

### Added
- Initial semantic field types specification
- **Identifier Types**: Primary, Natural, Foreign Key references
- **Categorical Types**: Enum, Status, Hierarchy levels, Country/Language codes
- **Boolean Types**: State flags, Existence flags, Validation flags
- **Temporal Types**: Event timestamps, Deadlines, Durations, Date components
- **Geographic Types**: Coordinates, H3 indexes, GeoJSON, Address components
- **Numeric Measurement Types**: Currency, Distance, Weight, Duration, Count, Percentage
- **Text Types**: Free text, Structured text, URLs
- **Entity & Business Data Types**: Legal entity names, Registration numbers, Business addresses
- **Personal Data Types (PII)**: Person names, Email, Phone, Personal address, ID documents
- **Metadata Types**: Source metadata, Partition keys, Version numbers

### Documentation
- Core principles and versioning
- Known limitations section
- Future considerations for standards alignment (W3C DPV, Schema.org, UN/CEFACT, etc.)
- SQL examples for data quality checks
- Profiling recommendations per type

---

## Version Numbering

- **0.x.x**: Pre-release versions, API may change
- **1.0.0**: First stable release (planned)
