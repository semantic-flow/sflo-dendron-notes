---
id: tn4qh04jkscnv9gpr5cq2x6
title: service config
desc: ''
updated: 1752429050607
created: 1752331872321
---

Service config refers to the final configuration used by the [[service]]. It can contain default settings for[[sflo.concept.node-config]], but it is a separate concept.

- `flow-service-config.jsonld` in the flow installation directory (or anywhere else, conceivably even in an _assets)
- whether "contained services" run, and on which ports
- what node-level config "system defaults" should be used in the absence of [[sflo.concept.mesh.resource.element.node-component.config]] and [[sflo.concept.mesh.resource.element.node-config-defaults]]
- logging configuration
