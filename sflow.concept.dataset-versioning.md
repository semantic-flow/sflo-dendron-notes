---
id: m24m2p2s7iom3rwzvp28kvy
title: Dataset Versioning
desc: ''
updated: 1751138597765
created: 1730373700120
---

As a dataset gets updated, it might make sense keep previous versions of it, i.e., with a versioned dataset.


To avoid an infinite loop with versioned datasets of versioned dataset, [[sflow.mesh.resource.element.version-dataset]] (_v1, _v2, etc) and their [[sflow.mesh.resource.element.v-series-dataset-series]] (_v-series) 

As always with datasets, metadata dataset versioning is configurable. By default, it should be off.


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
