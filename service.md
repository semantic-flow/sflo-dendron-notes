---
id: vlx4pyivfp5u0kxl1weczft
title: flow-service
desc: ''
updated: 1752443724459
created: 1750729257339
---

- supports the official clients ([[cli]] and [[sflo.product.flow-web]]) and third-party clients by
  - providing an [[api]]
  - serving meshes via a contained static http service
  - keeping central control of mesh operations.
    - e.g. locking sub-meshes during weave.
    - enables parallelism, which can speed up the weave and ensure processes don't clobber each others' work. It can also provide locking to allow multi-user or multi-client modification.

## Functions

- file watchers
- file locking



## Dependencies

