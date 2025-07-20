---
id: 108ze62n5nw1n3ntaeo2yxf
title: API
desc: ''
updated: 1753015597934
created: 1752678551342
---

**Mesh Management:**
- `POST /api/mesh/init` - Initialize new mesh
- `POST /api/mesh/node` - Create new node
- `GET /api/mesh/node/{path}` - Retrieve node
- `PUT /api/mesh/node/{path}` - Update node
- `DELETE /api/mesh/node/{path}` - Delete node

**Component Management:**
- `POST /api/mesh/node/{path}/component` - Add component
- `GET /api/mesh/node/{path}/component/{type}` - Get component
- `PUT /api/mesh/node/{path}/component/{type}` - Update component

**Dataset Operations:**
- `POST /api/mesh/dataset/upload` - Upload RDF dataset distribution
- `POST /api/mesh/dataset/extract` - Extract entities from RDF dataset
- `GET /api/mesh/dataset/{path}/versions` - List dataset versions
- `POST /api/mesh/dataset/{path}/weave` - Trigger weave operation
- `POST /api/mesh/assets/upload` - Upload binary file resources (separate from dataset extraction)


## mesh initialization

- create a[[sflo.concept.root-node]] [[sflo.concept.mesh.resource.element.flow.metadata]], [[sflo.concept.mesh.resource.element.handle]] and[[sflo.concept.mesh.resource.element.handle.page]]
- ? copy a [[sflo.concept.mesh.resource.element.documentation-resource.resource-page]] template and default css into a root _assets[[sflo.concept.mesh.resource.element.asset-tree]]
- create initial [[sflo.concept.mesh.resource.element.documentation-resource.resource-page]]s (for the node and all contained elements)
- optionally:
  - if the root node is to be a [[sflo.concept.mesh.resource.node.reference]], maybe some kind of wizard to select some initial reference data, like a zazuko lookup for the thing's class
  - if the root node is to be a [[sflo.concept.mesh.resource.node.data]], maybe a way to specify/download/upload a[[related-topics.dataset.distribution]]

## node initialization

- same as mesh initialization, except:
  - specify an [[sflo.concept.relative-identifier]] and select a parent node (or specify a path which is equivalent),
  - by default, no template/css copy

## weave

- copy any [[sflo.concept.mesh.resource.folder._next]]  to  
