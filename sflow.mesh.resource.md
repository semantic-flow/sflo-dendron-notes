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

- [[sflow.mesh.resource.node]]
- [[sflow.mesh.resource.element]]
- [[sflow.mesh.resource.asset-tree]] (terminal)

### File-based Mesh Resources (terminal)

- [[sflow.mesh.resource.file.resource-page]]
- [[sflow.mesh.resource.file.changelog]]
- [[sflow.mesh.resource.file.readme]]
- [[sflow.mesh.resource-type.distribution]]


- the structure of the mesh is built on [[sflow.mesh.resource.node]] and [[sflow.mesh.resource.element]]
  - but note that [[sflow.mesh.resource.asset-tree]] are "mesh terminal": other than their [[sflow.mesh.resource.element.flow-dataset]], their other files and folders are attached to but not contained in a mesh 
- with its path and (locally unique) name, every resource has an [[sflow.concept.identifier]]

In RDF-land, a resource is any node in an RDF graph that can be represented with
an IRI. (The other kinds of RDF graph nodes are literals and blank nodes)

Since folders and files in an [[sflow.mesh.resource.asset-tree]] are addressable, you could consider them RDF resources. But other than the asset's [[sflow.mesh.resource.element.flow-dataset]] (and it's contained files and folder), asset-tree contents are not sf-resources.