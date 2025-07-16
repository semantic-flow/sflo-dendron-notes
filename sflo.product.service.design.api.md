---
id: 108ze62n5nw1n3ntaeo2yxf
title: API
desc: ''
updated: 1752703210120
created: 1752678551342
---

## mesh initialization

- create a[[sflo.concept.root-node]] [[sflo.concept.mesh.resource.element.flow.metadata]], [[sflo.concept.mesh.resource.element.handle]] and[[sflo.concept.mesh.resource.element.handle.page]]
- ? copy a [[sflo.concept.mesh.resource.element.documentation-resource.resource-page]] template and default css into a root _assets[[sflo.concept.mesh.resource.element.asset-tree]]
- create initial [[sflo.concept.mesh.resource.element.documentation-resource.resource-page]]s (for the node and all contained elements)
- optionally:
  - if the root node is to be a [[sflo.concept.mesh.resource.node.reference]], maybe some kind of wizard to select some initial reference data, like a zazuko lookup for the thing's class
  - if the root node is to be a [[sflo.concept.mesh.resource.node.data]], maybe a way to specify/download/upload a[[related-topics.dataset.distribution]]

## node initialization

- same as mesh initialization, except:
  - specify an [[sflo.concept.identifier]] and select a parent node (or specify a path which is equivalent),
  - by default, no template/css copy

## weave

- copy any [[sflo.concept.mesh.resource.folder._next]]  to  
