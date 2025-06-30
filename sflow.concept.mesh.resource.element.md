---
id: 9c27yly4ed3ju7msf8luhge
title: sflow element
desc: ''
updated: 1751159498825
created: 1750706813437
---

## Elements Types

### User Datasets

[[sflow.concept.mesh.resource.element.reference-dataset]] and [[sflow.concept.mesh.resource.element.flow-dataset]] are the only user-modifiable [[sflow.concept.mesh.resource.element]] datasets. 
  - Reference datasets contain data about their [[sflow.concept.mesh.resource.node.reference]]'s referent.
  - Catalog datasets contain metadata about their parent "as a [[sflow.concept.mesh.resource]]"

### System Datasets

- **[[sflow.concept.mesh.resource.element.flow-dataset]]** hold metadata about their parent dataset, possibly including backlinks, provenance and anything else that should be versioned.

- **`_ref/`** Holds an RDF/JSON-LD file `<name>_ref.jsonld` describing the
  **referent** (names, labels, comments, for datasets: dataset metadata, for
  datasetseries: series‐level metadata).

- **`_v/`** Present only under **system‐versioned** datasets. Contains numbered
  version folders (`name-v1/`, `name-v2/`, …) each carrying its own
  `<name>_vN.trig` metadata distribution.

