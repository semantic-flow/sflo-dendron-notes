---
id: jyxdcjrv3npwwki06sfspqg
title: resource type
desc: ''
updated: 1750778037055
created: 1750709118500
---

There are six resources types represented in a mesh or semantic site:

| Mesh Resource Type       |terminal| `_ref`      | `<name>.trig`                   | `_v/`                      |
| ------------------------ | ------ | ----------- | ------------------------------- | -------------------------- |
| **Namespace**            | n      | n | n                  | n                |
| **Thing**                | n      | Required     | n                   | n                |
| **Unversioned Dataset**  | y      | Optional     | Required                       | n                |
| **Versioned Dataset**    | y      | Optional     | Required                       | Required              |
| **Dataset Distribution** | y      | n| n (it *is* the file)| n                |
| **Asset Tree**           | y      | Optional     | n                   | n                |

