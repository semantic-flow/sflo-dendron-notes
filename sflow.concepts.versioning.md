---
id: m24m2p2s7iom3rwzvp28kvy
title: Versioning
desc: ''
updated: 1750918256407
created: 1730373700120
---

## issues

- when there's only been one version of a dataset, do you create the v-series
  system elements?

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
