---
id: 8hmkiyjtsey7z8y5oi5xdxm
title: mesh node
desc: ''
updated: 1751088025073
created: 1750999795528
---

The primary constituents of a semantic-mesh are **mesh nodes**. They are physically represented with [[sflow.concept.mesh.resource.folder]] and they correspond to [[namespace segments|sflow.concept.namespace.segment]].

## Types

There are four types of mesh nodes:

- **[[namespace nodes|sflow.concept.mesh.resource.node.namespace]]**: containers for other mesh nodes
- **[[semantic nodes|sflow.concept.mesh.resource.node.reference]]**: nodes that refer to things other than themselves, like people, concepts or relationships. Physically, they are represented with folders that contain a [[sflow.concept.mesh.resource.element.reference-dataset]]
- **[[dataset nodes|sflow.concept.mesh.resource.node.dataset]]**: nodes that have distributions and, optionally, 