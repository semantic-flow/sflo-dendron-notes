---
id: 17l23vl7sqg997hr773dh23
title: Identifier
desc: ''
updated: 1751749960590
created: 1750654763700
---

Mesh identifiers are like URLs (or URIs), but without the scheme (i.e., https:// or file://).

if the identifier starts with a slash, its path is relative to the [[sflo.concept.root-node]].

if it starts with a `../` it refers to the parent, `../../` refers to the grandparent, etc.

otherwise it is a relative identifier, pathed from the current path.

## Identifier Semantics

Identifiers have the same semantics as [[sflo.concept.url]]

## Identifier Name Limitations

- initial underscores prefix all reserved dataset identifiers and should be avoided in general


![[sflow#^pnoqpr3ff4za]] 