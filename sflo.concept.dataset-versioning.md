---
id: m24m2p2s7iom3rwzvp28kvy
title: Dataset Versioning
desc: ''
updated: 1751692364688
created: 1730373700120
---

As a dataset gets updated, it might make sense keep previous versions of it, i.e., a [[sflo.concept.mesh.resource.element.version-dataset]].

This is especially useful with published data on the Semantic Web that other people and services might be referring to.

## Restrictions 

To avoid infinite loops with versioned datasets of versioned dataset, only 

As always with datasets, metadata dataset versioning is configurable. By default, it should be off.

**Dataset versions** are created "upon [[sflo.concept.weave-process]]" and correspond to a [[sflo.concept.mesh.resource.element.version-dataset]]. Versioning is sequential, (i.e., no major.minor.patch) and correspond to "_vN" [[sflo.concept.mesh.resource.folder._vN]]


## issues

- ~~when there's only been one version of a dataset, do you create the v-series  system elements?~~
  - if versioning is turned on
- where do you specify that a dataset shouldn't be versioned? 

  - needs to be somewhere common to element and node datasets

## monotonic version numbers

- they're easy
- you don't have to worry about what semantic versioning means in the context of
  data
