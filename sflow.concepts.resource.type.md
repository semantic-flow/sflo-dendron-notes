---
id: jyxdcjrv3npwwki06sfspqg
title: resource type
desc: ''
updated: 1750715409309
created: 1750709118500
---

There are six resources types represented in a mesh or semantic site:

| Mesh Resource Type       | terminal | `_id`       | `_ref`      | `<name>.trig`                   | `_v/`                      |
| ------------------------ | -------- | ----------- | ----------- | ------------------------------- | -------------------------- |
| **Namespace**            | n      |Required    | ❌ Forbidden | ❌ Forbidden                  | ❌ Forbidden                |
| **Thing**                | n      |Required    | Required     | ❌ Forbidden                   | ❌ Forbidden                |
| **Unversioned Dataset**  | y      |Required    | Optional     | Required                       | ❌ Forbidden                |
| **v-Series Dataset**    | y      |Required    | Optional     | Required                       | Required              |
| **Dataset Distribution** | y      |❌ Forbidden| ❌ Forbidden| ❌ Forbidden (it *is* the file)| ❌ Forbidden                |
| **Asset Tree**           | y      |Required    | Optional     | ❌ Forbidden                   | ❌ Forbidden                |

