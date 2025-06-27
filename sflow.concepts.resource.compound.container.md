---
id: clhryl4cf2hin7ak17ndfhz
title: Container
desc: 'basically, just folders'
updated: 1750971836913
created: 1730226356459
---

In Semantic Flow, a container:

- is physically situated as a folder in a filesystem hierarchy
- the path to the folder plus the folder name composes the namespace identifier
  - a [[sflow.concepts.sflow-repo]] allows you to publish namespaces on a static
    site hosting service
  - when sitting on local storage,
  - when published, the identifier becomes a FQDN.
- if it contains other [[resources|sflow.concepts.resource]], its path  determines their [[sflow.concepts.namespace]]
  - if there is no parent container, i.e., it is situated at the top of a
    [[sflow.concepts.semantic-mesh]], it is a [[sflow.concepts.namespace.root]]
- having an identifier makes something an [[RDF resource|sflow.concepts.resource]] that can be referred to in RDF data
  - semantically, it doesn't have a referent other than arguably itself, so not
    much reason to refer to it.
- namespace resources can situate/contain and, at their best, contextualize
  other resources
