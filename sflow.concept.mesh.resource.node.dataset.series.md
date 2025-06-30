---
id: tn4fb1v558dd7sh1r5ubh7n
title: mesh dataset-series
desc: ''
updated: 1751298556546
created: 1751003183255
---



A **mesh dataset-series node** is not generally distinguished from **[[sflow.concept.mesh.resource.node.dataset]]s** in terms of file structure, but their metadata should identify them as [[related-topics.dcat.dataset-series]] and identify the datasets in their series.

The contained datasets don't need to physically live inside the series' folder, but that can be an intuitive design pattern.

[[sflow.concept.mesh.dataset-type.versioned]] are not mesh dataset-series themselves, but they contain [[sflow.concept.mesh.resource.element.v-series-dataset-series]] which are. (And [[sflow.concept.mesh.resource.element.version-dataset]] that are not.) 