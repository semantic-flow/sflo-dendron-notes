---
id: xmdevh3s6gvp93m3nyc6683
title: mesh resource
desc: ''
updated: 1751129595158
created: 1750709094321
---

- any [[sflow.concepts.mesh.node]], [[sflow.concepts.flow.element]], [[sflow.concepts.flow.resource.page]], [[sflow.concepts.flow.resource.changelog]], or [[sflow.concepts.flow.resource.readme]] in a mesh
  - but note that [[sflow.concepts.flow.resource.asset-tree]] are "mesh terminal": other than their [[sflow.concepts.flow.element.catalog-dataset]], their other files and folders are attached to but not contained in a mesh 
- with its path and (locally unique) name, every resource has an [[sflow.concepts.identifier]]

In RDF-land, a resource is any node in an RDF graph that can be represented with
an IRI. (The other kinds of RDF graph nodes are literals and blank nodes)

Since folders and files in an [[sflow.concepts.flow.resource.asset-tree]] are addressable, you could consider them RDF resources. But other than the asset's [[sflow.concepts.flow.element.catalog-dataset]] (and it's contained files and folder), asset-tree contents are not sf-resources.