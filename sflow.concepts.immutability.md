---
id: q4jqx5qr6hbp6fo3pfvh042
title: Immutability
desc: ''
updated: 1751157458362
created: 1723213911939
---

- if you're updating a dataset, the principle of immutability is preserved in that the old data can still exist and be discoverable from the catalog 
  - sometimes, e.g., for compliance reasons, you have to hard delete a resource (as opposed to just tombstoning it, a soft delete), or even modifying it. 
  - [[hashes|sflow.features.changing-historical-datasets]] can be used to detect 

## References

https://s11.no/2013/prov/resources-that-change-state/