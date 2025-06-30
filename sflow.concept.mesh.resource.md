---
id: xmdevh3s6gvp93m3nyc6683
title: mesh resource
desc: ''
updated: 1751208168293
created: 1750709094321
---

There are 

## Types of Mesh Resources

### Folder-Based Mesh Resources

- [[sflow.concept.mesh.resource.node]]
- [[sflow.concept.mesh.resource.element]]
- [[sflow.concept.mesh.resource.asset-tree]] (terminal)

### File-based Mesh Resources (terminal)

- [[sflow.concept.mesh.resource.file.resource-page]]
- [[sflow.concept.mesh.resource.file.changelog]]
- [[sflow.concept.mesh.resource.file.readme]]
- [[sflow.concept.mesh.resource-type.distribution]]


- the structure of the mesh is built on [[sflow.concept.mesh.resource.node]] and [[sflow.concept.mesh.resource.element]]
  - but note that [[sflow.concept.mesh.resource.asset-tree]] are "mesh terminal": other than their [[sflow.concept.mesh.resource.element.flow-dataset]], their other files and folders are attached to but not contained in a mesh 
- with its path and (locally unique) name, every resource has an [[sflow.concept.identifier]]

In RDF-land, a resource is any node in an RDF graph that can be represented with
an IRI. (The other kinds of RDF graph nodes are literals and blank nodes)

Since folders and files in an [[sflow.concept.mesh.resource.asset-tree]] are addressable, you could consider them RDF resources. But other than the asset's [[sflow.concept.mesh.resource.element.flow-dataset]] (and it's contained files and folder), asset-tree contents are not sf-resources.