---
id: 8hmkiyjtsey7z8y5oi5xdxm
title: mesh node
desc: ''
updated: 1751088025073
created: 1750999795528
---

The primary constituents of a semantic-mesh are **mesh nodes**. They are physically represented with [[sflow.concepts.flow.folder]] and they correspond to [[namespace segments|sflow.concepts.namespace.segment]].

## Types

There are four types of mesh nodes:

- **[[namespace nodes|sflow.concepts.mesh.node.namespace]]**: containers for other mesh nodes
- **[[semantic nodes|sflow.concepts.mesh.node.reference]]**: nodes that refer to things other than themselves, like people, concepts or relationships. Physically, they are represented with folders that contain a [[sflow.concepts.flow.element.reference-dataset]]
- **[[dataset nodes|sflow.concepts.mesh.node.dataset]]**: nodes that have distributions and, optionally, 