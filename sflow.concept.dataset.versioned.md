---
id: elncscz2versnfcbtvqsg46
title: semantic flow versioned dataset
desc: ''
updated: 1751158295457
created: 1750655550072
---

A dataset whose old versions are kept around. Physically, the historical versions are location in "_vN" folders under the dataset's [[sflow.concept.mesh.resource.folder._v-series]], e.g. "_v1" or "_v9999". 

The collection of versions is a [[sflow.concept.mesh.resource.element.v-series-dataset-series]], so there is also a "_v-series" folder, which itself is a  [[sflow.concept.dataset.unversioned]] 