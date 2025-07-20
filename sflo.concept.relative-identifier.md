---
id: 17l23vl7sqg997hr773dh23
title: Relative Identifier
desc: ''
updated: 1753037773688
created: 1750654763700
---

Relative identifiers are like URLs (or URIs), but without the scheme (i.e., https:// or file://). They correspond to the filesystem location of their corresponding resources. Since identifiers are maintained per-node in the node's _config-flow, their paths are relative to the distributions where they are specified. 

if it starts with a `../` it refers to the parent, `../../` refers to the grandparent, etc.

## Identifier Semantics

Identifiers have the same semantics as [[sflo.concept.url]]

## Identifier Name Limitations

- initial underscores prefix all reserved dataset identifiers and should be avoided in general
![[sflo#^pnoqpr3ff4za]] 

## Examples

Directory structure: node/flow/snapshot/distribution.trig

Example: my-dataset/_config-flow/_current/config.trig
Meta-flow example: my-dataset/_meta-flow/_current/meta.trig

Relative identifiers from meta-flow distributions:

Node self-reference: "../../my-dataset"
Other flows: "../../_config-flow/_current/config.trig", "../../_data-flow/_current/data.jsonld"
Elements in other flows: Same pattern, just different flow names