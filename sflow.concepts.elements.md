---
id: 9c27yly4ed3ju7msf8luhge
title: elements
desc: ''
updated: 1750717469453
created: 1750706813437
---

## Elements Types

Every [[folder|sflow.concepts.mesh.folder]] in the mesh contains one or more of these resource-appropriate elements. They help define the [[sflow.concepts.resource.type]]:

### Dataset Elements

- **`_id/`**
  Holds an RDF file `<name>_id.trig` describing the **identifier itself** (including backlinks, what type of resource it represents, and perhaps identifier provenance).

- **`_ref/`**
  Holds an RDF/JSON-LD file `<name>_ref.jsonld` describing the **referent** (names, labels, comments, for datasets: dataset metadata, for datasetseries: series‐level metadata).

- **`_v/`**
  Present only under **system‐versioned** datasets. Contains numbered version folders (`name-v1/`, `name-v2/`, …) each carrying its own `<name>_vN.trig` metadata distribution.

### Distribution Elements

- **_name_.trig/jsonld/** 
  These are the content of datasets, i.e., "resources that are". There should be only one distribution in the mesh, but alternate serializations might be generated on publish.