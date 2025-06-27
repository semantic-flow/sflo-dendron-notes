---
id: 8hmkiyjtsey7z8y5oi5xdxm
title: mesh node
desc: ''
updated: 1751002026019
created: 1750999795528
---

The primary constituents of a semantic-mesh are **mesh nodes**. They are physically represented with [[sflow.concepts.semantic-mesh.folder]] and they correspond to [[namespace segments|sflow.concepts.namespace.segment]].

## Types

There are four types of mesh nodes:

- **[[namespace nodes|sflow.concepts.mesh-node.namespace]]**: containers for other mesh nodes
- **[[semantic nodes|sflow.concepts.mesh-node.semantic]]**: nodes that refer to things other than themselves, like people, concepts or relationships. Physically, they are represented with folders that contain a [[sflow.concepts.element.reference-dataset]]