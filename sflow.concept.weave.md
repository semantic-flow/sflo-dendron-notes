---
id: rall4fbxm369okmy5383sf8
title: Weave
desc: ''
updated: 1751237912902
created: 1751128698638
---

- checks for required [[sflow.concept.mesh.resource-facet.system]] and creates them if missing
- optionally removes extraneous files, interactively if requested
- for changed datasets (i.e., a distribution has changed)
  - creates a new [[sflow.concept.mesh.resource.element.version-dataset]] 
  - updates version metadata
  - generates 
- regenerates affected [[sflow.concept.mesh.resource.element.resource-page]]

## Best Practices

### Weave Before Commit, and Definitely Weave Before Push

This ensures that:

- broken references are cleaned up
- [[sflow.concept.mesh.resource-facet.dataset.current]] is identical to the latest version
- if you need to be able to work on a draft changes, 

## Features

### Tombstoning

If you know a mesh is permanently moving to a new location (or even if a branch is being created somewhere else), you should be able to tell weave to insert references to the new location

