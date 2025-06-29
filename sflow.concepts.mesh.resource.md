---
id: xmdevh3s6gvp93m3nyc6683
title: mesh resource
desc: ''
updated: 1751157828754
created: 1750709094321
---

There are 

## Types of Mesh Resources

### Folder-Based Mesh Resources

- [[sflow.concepts.mesh.node]]
- [[sflow.concepts.flow.element]]
- [[sflow.concepts.flow.resource.asset-tree]] (terminal)

### File-based Mesh Resources (terminal)

- [[sflow.concepts.flow.resource.page]]
- [[sflow.concepts.flow.resource.changelog]]
- [[sflow.concepts.flow.resource.readme]]
- [[sflow.concepts.flow.resource.distribution]]


- the structure of the mesh is built on [[sflow.concepts.mesh.node]] and [[sflow.concepts.flow.element]]
  - but note that [[sflow.concepts.flow.resource.asset-tree]] are "mesh terminal": other than their [[sflow.concepts.flow.element.mesh-dataset]], their other files and folders are attached to but not contained in a mesh 
- with its path and (locally unique) name, every resource has an [[sflow.concepts.identifier]]

In RDF-land, a resource is any node in an RDF graph that can be represented with
an IRI. (The other kinds of RDF graph nodes are literals and blank nodes)

Since folders and files in an [[sflow.concepts.flow.resource.asset-tree]] are addressable, you could consider them RDF resources. But other than the asset's [[sflow.concepts.flow.element.mesh-dataset]] (and it's contained files and folder), asset-tree contents are not sf-resources.