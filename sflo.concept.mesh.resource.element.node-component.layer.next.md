---
id: yci31i07s3bwpl9jd2hptod
title: next layer
desc: ''
updated: 1751956257316
created: 1751238695109
---

The **next layer** serves as a draft workspace for ongoing changes to a [[sflo.concept.mesh.resource.element.node-component.versioned]]. 

After a version-bumping weave, a next layer starts identical to the current dataset but can be modified safely without affecting the stable current version. During weaving, _next content becomes the new current dataset and gets snapshotted as the latest version, while _next naturally remains ready for the next round of drafts.

This allows continuous development and version control commits without requiring immediate version bumps or disrupting users of the stable dataset.


## Containment Rules

- must-be-contained-in: [[sflo.concept.mesh.resource.node.data]], [[sflo.concept.mesh.resource.element.node-component.reference]]
