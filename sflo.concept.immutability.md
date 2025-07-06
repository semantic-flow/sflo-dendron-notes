---
id: q4jqx5qr6hbp6fo3pfvh042
title: Immutability
desc: ''
updated: 1751774805823
created: 1723213911939
---

- if you're updating a dataset, the principle of immutability is preserved in that the old data can still exist and be discoverable from the metadata 
- [[sflo.concept.mesh.resource.element.version-dataset]] (e.g. in [[sflo.concept.mesh.resource.folder._vN]]) should be treated as immutable. If you need to refer to the current dataset "as is", you should refer to its corresponding dataset version.
  - note that it is possible for the current version to have mutated since the last version bump, but "weave before commit" addresses this.
- sometimes, e.g., for compliance reasons, you have to hard delete a resource (as opposed to just tombstoning it, a soft delete), or even modifying it. 
  - [[hashes|sflo.feature.changing-historical-datasets]] can be used to detect 

## References

https://s11.no/2013/prov/resources-that-change-state/