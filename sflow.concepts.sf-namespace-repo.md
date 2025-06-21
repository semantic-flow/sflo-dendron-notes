---
id: c4wo7mdjzcit2bmvqr5sx4u
title: namespace-repo
desc: ''
updated: 1750487831694
created: 1729878955236
---


- can either be "username/org pages repository"" (which automatically hosts content at the namesake url, so maybe call it a namesake repository, e.g. djradon.github.io) or 2nd-level (corresponding to an owned repo)
- an namespace-repo contains
  - a 'docs' folder that contains the generated sf site
    - within 'docs' there's a folder for each namespace
    - an additional "root" level folder below "docs" for site assets named "_assets"
    - index.html and some index.* distribution for discovering what's contained in the site
  - a '_data_repos' folder for repos (especially [[sflow.concepts.sf-data-repo]]), folders, and files
  - templates and template mappings to apply to src items
  - assets/*reponame* folder for keeping images, stylesheets, and other resources
  
## Questions

- should data-repos be able to include HTML templates and entity-to-template mappings?
  - yes? but can be overridden at the root/namespace repo
- should 
