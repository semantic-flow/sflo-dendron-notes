---
id: faq001
title: Why don't namespace nodes and data nodes have reference components?
desc: ''
updated: 1751994453796
created: 1751351297000
---

## Question

Why don't [[namespace nodes|sflo.concept.mesh.resource.node.namespace]] have [[reference components|sflo.concept.mesh.resource.element.node-component.reference]] like [[reference nodes|sflo.concept.mesh.resource.node.reference]] do?

## Answer

URLs that correspond to namespace nodes don't necessarily have a referent. If they do, give them a [[sflo.concept.mesh.resource.element.node-component.reference]] and make them reference nodes!

e.g.:
- `ns/` : doesn't refer to anything
- `ns/people` : refers to people, should probably be a reference node

Since reference nodes can do everything namespace nodes can do (i.e., contain other nodes), there's no drawback to making namespace nodes into reference nodes.
