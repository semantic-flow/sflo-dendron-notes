---
id: 108ze62n5nw1n3ntaeo2yxf
title: API
desc: ''
updated: 1753138562498
created: 1752678551342
---

## Mesh Management ##

- `POST /api/mesh/init` - Initialize new mesh

## Node Management **

- `POST /api/mesh/node` - Create new node
- `GET /api/mesh/node/{path}` - Retrieve node
- `PUT /api/mesh/node/{path}` - Update node
- `DELETE /api/mesh/node/{path}` - Delete node
- `graft` - start a new node from an existing snapshot

**Flow and Snapshot Management:**
- `weave`
- `POST /api/mesh/{path}/weave` - Trigger weave operation

**Distribution Operations:**
TODO: review

- `POST /api/mesh/dataset/upload` - Upload RDF dataset distribution
- `POST /api/mesh/dataset/extract` - Extract entities from RDF dataset
- `GET /api/mesh/dataset/{path}/versions` - List dataset versions
- `POST /api/mesh/assets/upload` - Upload binary file resources (separate from dataset extraction)

## Service Management


## mesh initialization

- create a[[sflo.concept.root-node]] [[sflo.concept.mesh.resource.element.flow.metadata]], [[sflo.concept.mesh.resource.element.handle]] and[[sflo.concept.mesh.resource.element.handle.page]]
- ? copy a [[sflo.concept.mesh.resource.element.documentation-resource.resource-page]] template and default css into a root _assets[[sflo.concept.mesh.resource.element.asset-tree]]
- create initial [[sflo.concept.mesh.resource.element.documentation-resource.resource-page]]s (for the node and all contained elements)
- optionally:
  - if the root node is to be a [[sflo.concept.mesh.resource.node.reference]], maybe some kind of wizard to select some initial reference data, like a zazuko lookup for the thing's class
  - if the root node is to be a [[sflo.concept.mesh.resource.node.data]], maybe a way to specify/download/upload a[[related-topics.dataset.distribution]]
- set defaults, especially for provenance/attribution

## node initialization

- same as mesh initialization, except:
  - specify an [[sflo.concept.relative-identifier]] and select a parent node (or specify a path which is equivalent),
  - by default, no template/css copy

## weave

- copy any [[sflo.concept.mesh.resource.folder._next]]  to  
