---
id: yci31i07s3bwpl9jd2hptod
title: next dataset
desc: ''
updated: 1751300167704
created: 1751238695109
---

The **next dataset** serves as a draft workspace for ongoing changes to a [[sflow.concept.mesh.dataset-type.versioned]]. 

After a version-bumping weave, a next dataset starts identical to the current dataset but can be modified safely without affecting the stable current version. During weaving, _next content becomes the new current dataset and gets snapshotted as the latest version, while _next naturally remains ready for the next round of drafts.

This allows continuous development and version control commits without requiring immediate version bumps or disrupting users of the stable dataset.


## Containment Rules

- must-be-contained-in: [[sflow.concept.mesh.resource.node.dataset]], [[sflow.concept.mesh.resource.element.reference-dataset]]
