---
id: m24m2p2s7iom3rwzvp28kvy
title: Dataset Versioning
desc: ''
updated: 1751296199310
created: 1730373700120
---

As a dataset gets updated, it might make sense keep previous versions of it, i.e., a [[sflow.concept.mesh.resource.element.version-dataset]].

This is especially useful with published data on the Semantic Web that other people and services might be referring to.

## Restrictions 

To avoid infinite loops with versioned datasets of versioned dataset, only 

As always with datasets, metadata dataset versioning is configurable. By default, it should be off.

**Dataset versions** are created "upon [[sflow.concept.weave]]" and correspond to a [[sflow.concept.mesh.resource.element.version-dataset]]. Versioning is sequential, (i.e., no major.minor.patch) and correspond to "_vN" [[sflow.concept.mesh.resource.folder._vN]]


## issues

- ~~when there's only been one version of a dataset, do you create the v-series  system elements?~~
  - if versioning is turned on
- where do you specify that a dataset shouldn't be versioned? 
  - ~~maybe the presence of absence of a _v-series folder is the marker? So you can turn off versioning at any time, but you lose all accumulated versions. Hmm... another argument for keeping the versions under the versioned dataset.~~
  - needs to be somewhere common to element and node datasets

## monotonic version numbers

- they're easy
- you don't have to worry about what semantic versioning means in the context of
  data

## example versioned dataset

```
dataset-name/
├── index.html                      ← Dataset reference page  
├── dataset-name.trig               ← Current distribution (working copy)  
├── dataset-name.jsonld               ← Current distribution (working copy)  
├── _v-series/                      ← DatasetSeries IRI  
│   ├── index.html                  ← Series reference page  
│   ├── _catalog/                   ← Catalog of this series  
│   │   ├── index.html              ← Catalog reference page  
│   │   └── dataset-name_catalog.trig ← Current series catalog  
│   ├── v1/                         ← Version 1 IRI  
│   │   ├── index.html              ← Version 1 reference page  
│   │   └── dataset-name_v1.trig    ← Version 1 data snapshot  
│   └── v2/ …                       ← Further versions…  
└── _assets/                        ← User‐facing assets  
    ├── index.html                  ← Assets reference page  
    └── logo.svg
```
