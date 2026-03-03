# Automatic Generation of DCAT-AP+

In DCAT-AP+ we do not manually recreate DCAT-AP in LinkML but **auto-generate** it as the base layer from the authoritative SHACL shapes published by SEMIC. This ensures that the LinkML schema stays fully aligned with the official specification and can be updated systematically when DCAT-AP evolves.

## Why auto-generate?

Manual porting of a complex specification invites drift. The DCAT-AP SHACL shapes define ~25 node shapes with ~150 property shapes, each with cardinality constraints, range definitions, and IRI mappings. Reproducing this by hand would be error-prone and hard to maintain across DCAT-AP releases.

By scripting the translation, we get two guarantees:

1. **Semantic identity**: Every `class_uri` and `slot_uri` in the generated LinkML schema is copied verbatim from the SHACL `sh:targetClass` and `sh:path` attributes. The resulting model is structurally equivalent to the official shapes.
2. **Reproducibility**: When SEMIC publishes a new DCAT-AP release, re-running the script against the updated SHACL shapes produces an updated base layer, making the delta to our extension layer explicit.

## The pipeline

![conversion_pipeline_dark.svg](images/conversion_pipeline_dark.svg#only-dark)
![conversion_pipeline_light.svg](images/conversion_pipeline_light.svg#only-light)
The script produces **two** LinkML schemas from the same input:

| Output                                                  | Purpose                                                                                                                                                                          |
|---------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [`dcat_ap_linkml.yaml`](schema/dcat_ap_linkml.yaml)     | A near-1:1 translation of the DCAT-AP SHACL shapes into LinkML. Useful as a standalone reusable artifact for anyone wanting DCAT-AP in LinkML without extensions.                |
| [`dcat_ap_plus.yaml`](schema/dcat_ap_plus.yaml)         | The same base layer **plus** the DCAT-AP+ extension: the provenance core, attribute patterns, and classification pattern described in [Design Patterns](design-patterns.md).     |

## Input: Which SHACL shapes?

The script uses the JSON-LD serialization of the DCAT-AP 3.0.0 SHACL shapes, downloaded from the [SEMIC DCAT-AP repository (`master` branch, `releases/3.0.0/shacl/`)](https://github.com/SEMICeu/DCAT-AP/tree/master/releases/3.0.0/shacl).

!!! warning "SEMIC publishes multiple shape files that differ"
    The shapes in the `master` branch's `releases/3.0.0` folder differ from those in the [tagged `3.0.0` release](https://github.com/SEMICeu/DCAT-AP/releases/tag/3.0.0) and the `3.0.0` branch. We use the `master` branch version because it is the one linked from the [official specification website](https://semiceu.github.io/DCAT-AP/releases/3.0.0/) and reflects the most recent editorial corrections. See also [DCAT-AP issue #428](https://github.com/SEMICeu/DCAT-AP/issues/428).

## How the translation works

The [`dcat_ap_shacl_2_linkml.py`](https://github.com/nfdi-de/dcat-ap-plus/blob/main/src/dcat_ap_plus/dcat_ap_shacl_2_linkml.py) script iterates over each SHACL node shape in the [JSON-LD file](https://github.com/nfdi-de/dcat-ap-plus/blob/main/src/dcat_ap_plus/dcat_ap_shacl.jsonld) and maps it to a LinkML construct:

**Node shapes → classes or datatypes.** A node shape whose `sh:targetClass` points to an ontology class (e.g. `dcat:Dataset`) becomes a LinkML class. A node shape targeting an XSD datatype (e.g. `xsd:duration`) becomes a LinkML datatype.

**Property shapes → slots.** Each `sh:property` within a node shape becomes a slot on the derived class. Cardinality (`sh:minCount`, `sh:maxCount`), range (`sh:class`, `sh:datatype`), and the property IRI (`sh:path`) are all preserved.

**Naming convention.** Slot names are converted from the DCAT-AP camelCase convention to LinkML's snake_case (e.g. `accessURL` → `access_URL`, `contactPoint` → `contact_point`).

### Handling of union ranges

The DCAT-AP shapes contain two kinds of unions:

- **Object class unions** (e.g. `dcat:primaryTopic` can range over `Dataset`, `DatasetSeries`, `Catalogue`, or `DataService`): handled via LinkML's [`any_of`](https://linkml.io/linkml/schemas/advanced.html#unions-as-ranges) keyword.
- **Datatype unions** (e.g. the `TemporalLiteral` shape unions `xsd:date`, `xsd:dateTime`, `xsd:gYear`, and `xsd:gYearMonth`): due to a [known LinkML limitation](https://github.com/linkml/linkml/issues/1813), these are conservatively restricted to `xsd:date`. This is a stricter interpretation than the official DCAT-AP shapes and will be relaxed once LinkML supports datatype unions.

!!! note "Shapes that are skipped"
    The script explicitly ignores `rdfs:Literal` (replaced by LinkML's default `string` range), the `CataloguedResource` union shape (replaced by the `Any` class with `any_of` constraints), and a duplicate `mediaType` shape that appears to be an editorial error in the source.

## What is auto-generated vs. manually authored

| Layer | How it's created | Where in the script |
|---|---|---|
| **DCAT-AP base** (classes, slots, datatypes, enums from the official shapes) | Auto-generated by `parse_dcat_ap_shacl_shapes()` | [Lines 1–411](https://github.com/nfdi-de/dcat-ap-plus/blob/main/src/dcat_ap_plus/dcat_ap_shacl_2_linkml.py#L1-L411) |
| **DCAT-AP+ extension** (provenance core, attributes, `ClassifierMixin`, contextual metadata) | Programmatically added by `build_dcatapplus_linkml()` | [Lines 412–991](https://github.com/nfdi-de/dcat-ap-plus/blob/main/src/dcat_ap_plus/dcat_ap_shacl_2_linkml.py#L412-L991) |

The extension layer is authored *in Python code*, not in raw YAML, so that it builds on top of the same `SchemaBuilder` object that holds the auto-generated DCAT-AP base. This ensures that references between base and extension elements (e.g. making `was_generated_by` mandatory on `Dataset`, or adding slots to `Activity`) are validated at build time.

Elements belonging to the DCAT-AP+ extension are tagged with `in_subset: [domain_agnostic_core]` in the schema, making it easy to distinguish them from the auto-generated DCAT-AP base.

## Re-running the generation

To regenerate both schemas after updating the input SHACL shapes or modifying the extension code:

```bash
# 1. If DCAT-AP has released new shapes, replace the input file:
#    Download the updated dcat_ap_shacl.jsonld into src/dcat_ap_plus/

# 2. Run the build script:
uv run python src/dcat_ap_plus/dcat_ap_shacl_2_linkml.py

# 3. Validate the generated schema and test data:
uv run linkml-validate tests/data/valid/AnalysisDataset-001.yaml \
  -s src/dcat_ap_plus/schema/dcat_ap_plus.yaml -C AnalysisDataset

# 4. Regenerate the documentation:
rm -rf docs/elements/*.md && \
  uv run gen-doc -d docs/elements src/dcat_ap_plus/schema/dcat_ap_plus.yaml
```

The CI pipeline (GitHub Actions) runs schema validation and data validation on every pull request, catching regressions automatically.
