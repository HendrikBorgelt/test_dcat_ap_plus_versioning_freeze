# Projects Using DCAT-AP+

## Active domain profiles

### ChemDCAT-AP

[:octicons-repo-16: nfdi-de/chem-dcat-ap](https://github.com/nfdi-de/chem-dcat-ap) · [:octicons-book-16: Documentation](https://nfdi-de.github.io/chem-dcat-ap)

ChemDCAT-AP is the profile from which DCAT-AP+ was originally extracted. Developed jointly by [NFDI4Chem](https://nfdi4chem.de) and [NFDI4Cat](https://nfdi4cat.org/), it specializes the generic DCAT-AP+ classes for chemistry and catalysis research data. Key specializations include:

- `SubstanceSample` a subclass of `EvaluatedEntity` mapped to SIO
- Domain-specific sub-slots like `used_catalyst` (`is_a: carried_out_by`, mapped to `RXNO:0000425`) and `generated_product` (`is_a: had_output_entity`, mapped to `RO:0004008`)
- Dedicated properties for chemical identifiers (`inchikey`, `smiles`) that replace the generic `has_qualitative_attribute` pattern

ChemDCAT-AP is deployed in production within the [NFDI4Chem Search Service](https://search.nfdi4chem.de/), where it powers semantically rich metadata export and the NFDI Chemistry Knowledge Graph.

### NMR-DCAT-AP

[:octicons-repo-16: NFDI4Chem/nmr-dcat-ap](https://github.com/NFDI4Chem/nmr-dcat-ap) · [:octicons-book-16: Documentation](https://nfdi4chem.github.io/nmr-dcat-ap/)

NMR-DCAT-AP extends ChemDCAT-AP to formalize the [MARGARITAS](https://doi.org/10.25504/FAIRsharing.c29400) minimum information standard for NMR spectroscopy. It demonstrates how the DCAT-AP+ → ChemDCAT-AP inheritance chain can be extended further to encode specific MIChI (Minimum Information for a Chemical Investigation) standards. The workflow serves as a validated template for encoding other MIChI standards.

## Cross-domain validation

### NOMAD (materials science)

We are collecting dataset examples from [NOMAD](https://nomad-lab.eu/), a materials science data repository, to test DCAT-AP+ against metadata from a domain adjacent to but distinct from chemistry. These examples are being used to validate that the provenance core pattern generalizes beyond its original chemistry and catalysis context. Further examples from other domains are expected as we present this work more broadly within the NFDI Section (Meta)data, Terminology, Provenance.

## Emerging interest

### NFDIcore alignment

We are collaborating with the developers of the [NFDIcore ontology](https://nfdi.fiz-karlsruhe.de/ontology/) to explore mapping DCAT-AP+ schema elements to NFDIcore classes and predicates in addition to PROV-O, DCTerms and QUDT. NFDIcore is a BFO-aligned ontology that provides narrower, more precisely scoped terms that match the intent of several DCAT-AP+ node shapes. Because DCAT-AP+ uses LinkML's `class_uri` and `slot_uri` for ontology alignment (see [Design Patterns: foundational principle](design-patterns.md#foundational-principle-linkml-elements-as-shacl-shapes)), swapping or supplementing the underlying ontology mappings does not require restructuring the schema.

### Life sciences (RO-Crate / Schema.org)

In conversations with NFDI colleagues from the life sciences, who use [RO-Crate](https://www.researchobject.org/ro-crate/) and Schema.org for attaching domain-specific metadata to datasets, there is good alignment potential. Their approach shares the activity/process focus that DCAT-AP+ introduces. A formal comparison and possible bridging is a future direction.

## Using DCAT-AP+ in your project?

If you are building a domain profile on DCAT-AP+, see the [extension rules](how-to-extend.md) for the contract between DCAT-AP+ and downstream schemas. We welcome additions to this page. Feel free to open an issue or pull request.
