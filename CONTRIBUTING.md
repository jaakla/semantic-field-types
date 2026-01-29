# Contributing to Semantic Field Types

Thank you for your interest in contributing to the Semantic Field Types specification!

## Ways to Contribute

### 1. Propose a New Semantic Type

If you've identified a field pattern that isn't covered by the current taxonomy:

1. Check existing types to ensure it's not already covered
2. Open an issue using the **"New Type Proposal"** template
3. Include:
   - Purpose and use case
   - Example field names
   - Suggested quality rules
   - Valid/invalid aggregations

### 2. Improve an Existing Type

If you have suggestions to improve an existing type definition:

1. Open an issue describing the improvement
2. Or submit a PR with the changes

### 3. Report Issues

- Unclear documentation
- Inconsistencies between types
- Missing quality rules or examples

### 4. Add Domain-Specific Extensions

If you work in a specific domain (healthcare, finance, logistics, etc.) and have domain-specific type patterns, we welcome extensions.

## Pull Request Process

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-duration-type`)
3. Make your changes to `semantic-field-types.md`
4. Ensure your changes follow the existing format
5. Update the CHANGELOG.md if applicable
6. Submit a PR with a clear description

## Type Definition Format

When proposing or editing types, follow this structure:

```markdown
#### X.Y Type Name 🔣
- **Purpose**: What this type represents
- **PII Classification**: (if applicable) PII / Sensitive PII / Non-PII
- **Examples**: `field_name_1`, `field_name_2`
- **Specifiers**: Additional qualifiers (e.g., unit, format)
- **Profiling**: How to analyze this field type
- **Quality Rules**:
  - Rule 1
  - Rule 2
- **Valid Aggregations**: COUNT, SUM, AVG, etc.
- **Invalid Aggregations**: (if applicable)
- **Visualization**: Recommended chart types
- **Analysis**: Common analysis patterns
```

## Code of Conduct

- Be respectful and constructive
- Focus on the technical merits of proposals
- Welcome newcomers and help them contribute

## Questions?

Open a GitHub Discussion for general questions or ideas that aren't yet ready for a formal proposal.

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
