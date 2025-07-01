---
id: 074yxm151l47n1aak58ggni
title: current dataset
desc: ''
updated: 1751300034791
created: 1751237726111
---

The current dataset represents the stable, published version of a dataset's content. It serves as the authoritative source that users and external systems reference, containing the most recent released data while remaining unchanged during active development.

The current dataset always matches the content of the latest versioned dataset (e.g., `_v3/`) and remains identical to the [[sflow.concept.mesh.resource.element.next-dataset]] until new changes begin. During weaving, the `_next` content becomes the new current dataset and gets snapshotted as the next version, ensuring the current dataset stays synchronized with the latest stable release.

This provides a stable reference point for citations and external links while allowing ongoing development work to proceed safely in the `_next` dataset without disrupting users of the published data.