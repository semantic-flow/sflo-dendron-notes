---
id: 074yxm151l47n1aak58ggni
title: current layer
desc: ''
updated: 1751956164950
created: 1751237726111
---

The current layer represents the stable, published version of a dataset's content. It serves as the authoritative source that users and external systems reference, containing the most recent released data while remaining unchanged during active development.

The current layer always matches the content of the latest versioned layer (e.g., `_v3/`) and remains identical to the [[sflo.concept.mesh.resource.element.node-component.layer.next]] until new changes begin. During weaving, the `_next` content becomes the new current layer and gets snapshotted as the next version, ensuring the current layer stays synchronized with the latest stable release.

This provides a stable reference point for citations and external links while allowing ongoing development work to proceed safely in the `_next` dataset without disrupting users of the published data.