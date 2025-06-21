---
id: fv43m7jgc5rrqv20pe9enb4
title: Supporting Transposable Repos
desc: ''
updated: 1750490334262
created: 1750489919875
---

## Problem

- supporting multiple sflow repos for any site


## Solution

- sf-root-repo contains no data, ids, or probably assets
- maybe it has its "authoritative" template overrides, 



sf-root-repo

    No actual data or identifiers.

    Only contains the site scaffold (templates, CI/CD, docs, "root assets" and "ultimate" template rules).

    Publishes to https://<org>.github.io/.

One or more sf-data-repos

    Each data-repo is effectively a sub-namespace of the root:

    https://<org>.github.io/{repo-name}/

    Holds IRIs under three “buckets” (as folders):

        /data/… → datasets & versions

        /id/… → identifiers (things, series, catalogs)

        /assets/… → images, CSS, etc.

    Owns its own Git history of all versions.

Integration step

    Root-repo’s build pulls in each data-repo (via submodule, CI checkout, or published artifact).

    For each data-repo, you rewrite its local @base to /{repo-name}/.

    You merge their /id, /data, /assets trees into the final site under /{repo-name}/….