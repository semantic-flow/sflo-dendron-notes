---
id: clhryl4cf2hin7ak17ndfhz
title: Namespace
desc: 'basically, just folders'
updated: 1750726293147
created: 1730226356459
---

In Semantic Flow, a namespace:

- is represented as a [[mesh folder|sflow.concepts.mesh.folder]]
- has a parent folder, unless it is a namespace root
- the path to the folder plus the folder name composes the namespace identifier
  - a [[sflow.concepts.sf-repo]] allows you to publish namespaces on a static site hosting service 
  - when published the identifier becomes a FQDN.
- having an identifier makes something an [[RDF resource|sflow.concepts.resource]] that can be referred to in RDF data
  - semantically, it doesn't have a referent other than arguably itself, so not much reason to refer to it.
- namespace resources can situate/contain and, at their best, contextualize other resources

