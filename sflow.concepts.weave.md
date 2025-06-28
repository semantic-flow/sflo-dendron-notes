---
id: rall4fbxm369okmy5383sf8
title: Weave
desc: ''
updated: 1751138966041
created: 1751128698638
---

- checks for required [[sflow.concepts.flow.resource.system]] and creates them if missing
- optionally removes extraneous files, interactively if requested
- for changed datasets (i.e., a distribution has changed)
  - creates a new [[sflow.concepts.flow.element.version-dataset]] 
  - updates version metadata
  - generates 
- regenerates affected [[sflow.concepts.flow.resource.page]]