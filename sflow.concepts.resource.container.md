---
id: clhryl4cf2hin7ak17ndfhz
title: Container
desc: 'basically, just folders'
updated: 1750878786396
created: 1730226356459
---

In Semantic Flow, a container:

- is derived directly from filesystem hierarchy; it is the path from the
  namespace root to the parent folder
- if there is no parent folder, the namespace is equal to the root
- the path to the folder plus the folder name composes the namespace identifier
  - a [[sflow.concepts.sf-repo]] allows you to publish namespaces on a static
    site hosting service
  - when published the identifier becomes a FQDN.
- having an identifier makes something an [[RDF
  resource|sflow.concepts.resource]] that can be referred to in RDF data
  - semantically, it doesn't have a referent other than arguably itself, so not
    much reason to refer to it.
- namespace resources can situate/contain and, at their best, contextualize
  other resources
