---
id: c4wo7mdjzcit2bmvqr5sx4u
title: sflow-repo
desc: ''
updated: 1750999120513
created: 1729878955236
---

A Semantic Flow repository (or sflow-repo for short) is a git repository that collects of one or more [[semantic meshes|sflow.concepts.semantic-mesh]]. 
  - sflow configuration data
  - [[static site generators|sflow.concepts.site.generation]] and configuration
  - assets, including images, templates and template mappings to apply to src items
  - an optional output directory containing a [[sflow.concepts.site]], (e.g. "docs" for some GitHub pages workflows)

 

## Github Repos

- can either be "username/org pages repository"" (which automatically hosts content at the namesake url, so maybe call it a namesake repository, e.g. djradon.github.io) or 2nd-level (corresponding to an owned repo)
- a 'docs' folder that contains the generated sf site
    - within 'docs' there's a folder for each namespace
  
## Questions

- should data-repos be able to include HTML templates and entity-to-template mappings?
  - yes? but can be overridden at the root/namespace repo
- can you include a mesh in an existing repo?
  - not ideally? for composability (i.e., linking a repo into an existing mesh) you need an unbroken chain.
  - maybe with symbolic links
