---
id: xmdevh3s6gvp93m3nyc6683
title: sf-resource
desc: ''
updated: 1751089966059
created: 1750709094321
---

- any [[sflow.concepts.mesh-node]], [[sflow.concepts.element]], or [[sflow.concepts.mesh-resource-page]] in a mesh
  - but note that [[sflow.concepts.mesh-node.asset-tree]] are "mesh terminal": other than their [[sflow.concepts.element.catalog-dataset]], the other files and folders are not sf-resources 
- with its path and (locally unique) name, every resource has an [[sflow.concepts.identifier]]

In RDF-land, a resource is any node in an RDF graph that can be represented with
an IRI. (The other kinds of RDF graph nodes are literals and blank nodes)

Since folders and files in an [[sflow.concepts.mesh-node.asset-tree]] are addressable, you could consider them RDF resources. But other than the asset's [[sflow.concepts.element.catalog-dataset]] (and it's contained files and folder), asset-tree contents are not sf-resources.