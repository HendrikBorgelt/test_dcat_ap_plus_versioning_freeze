# Versioning

## Versioning scheme: SchemaVer

DCAT-AP+ follows the **SchemaVer** scheme ([introduced by Snowplow](https://snowplow.io/blog/introducing-schemaver-for-semantic-versioning-of-schemas), also used by [Human Cell Atlas](https://github.com/HumanCellAtlas/metadata-schema/blob/master/docs/evolution.md#schema-versioning) and [OpenLineage](https://github.com/OpenLineage/OpenLineage/blob/main/spec/Versioning.md)). SchemaVer is designed for data schemas, where the critical question is *"will existing data still validate?"*, unlike SemVer, which is designed for code APIs.

Given a version number **MODEL.REVISION.ADDITION**:

| Increment | When | Effect on existing data |
|---|---|---|
| **MODEL** | A breaking schema change that prevents interaction with *any* historical data (e.g. removing a mandatory slot, changing a `class_uri`) | All existing data must be migrated |
| **REVISION** | A schema change that may prevent interaction with *some* historical data (e.g. making an optional slot mandatory, narrowing a range) | Some existing data may need updates |
| **ADDITION** | A schema change compatible with *all* historical data (e.g. adding a new optional slot, adding a new class) | No data migration needed |

### Examples of each level

- **ADDITION** (0.1.0 → 0.1.1): Adding an optional `comment` slot to `DataGeneratingActivity`
- **REVISION** (0.1.x → 0.2.0): Making `rdf_type` on `DataGeneratingActivity` mandatory (previously recommended)
- **MODEL** (0.x.y → 1.0.0): Restructuring the `QuantitativeAttribute` to use a different ontology alignment

Versions are tagged in git (e.g. `v1.2.3`) and published as [GitHub releases](https://github.com/HendrikBorgelt/test_dcat_ap_plus_versioning_freeze/releases) with changelogs.

## Relationship to DCAT-AP versions

DCAT-AP+ currently tracks **DCAT-AP 3.0.0**. The DCAT-AP base layer is [auto-generated](automatic-generation.md) from the official SHACL shapes. When SEMIC publishes a new DCAT-AP release (e.g. 3.1.0), the version impact on DCAT-AP+ depends on the nature of upstream changes:

- If DCAT-AP adds new optional properties → DCAT-AP+ ADDITION
- If DCAT-AP changes cardinality or ranges → DCAT-AP+ REVISION or MODEL, depending on scope
- Major DCAT-AP version changes (e.g. 4.0) → likely DCAT-AP+ MODEL bump

The DCAT-AP version that DCAT-AP+ is built on is documented in the schema's `source` field and in the generated schema description.

## Dynamic versioning in development

During development, the version string in the schema YAML header is managed by [`setuptools-scm` / `hatch-vcs`](https://github.com/pypa/hatch-vcs) via a dynamic versioning plugin. This is why the schema header shows a version like:

```yaml
version: "0.1.0rc3.post21.dev0+913fb36"
```

This string is **auto-generated from git tags**. Hence, it should not be edited manually. The components mean:

| Part | Meaning |
|---|---|
| `0.1.0rc3` | The last git tag (release candidate 3 of version 0.1.0) |
| `.post21` | 21 commits since that tag |
| `.dev0` | Development build |
| `+913fb36` | The git commit hash |

On tagged releases, the version simplifies to the clean SchemaVer string (e.g. `0.1.0`).
