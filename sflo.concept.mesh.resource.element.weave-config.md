---
id: 9el7ivz22xvt8exiz7tj23f
title: Weave Config
desc: ''
updated: 1751748254256
created: 1751646537672
---

Every [[sflo.concept.mesh.resource.node]] can have a weave-config.jsonld file including the [[root node|sflo.concept.root-node]]

All configuration is optional. If part of the config data is not present, the [[sflo.concept.weave-process]] recurses up the tree until it has checked for inheritable config and composed it.

If anything is missing after the recurse, it uses sensible defaults

## Config 

- Versioning policy - Default policy + specific dataset overrides
- Distribution syntaxes - Default + supported formats
- Cleanup policy - Handle extraneous/misplaced files
- Publishing base IRI - For IRI generation
- Template mappings - resource-specific template choices
- Convenience distributions - Whether to generate node-level copies
- Resource page generation - README.md handling + AI fallback options
- distribution naming - whether to include node parent name or just node name

```file
{
  "resource_pages": {
    "description_source": "README.md",
    "ai_fallback": {
      "enabled": true,
      "provider": "openai",  // or "anthropic", "local", etc.
      "model": "gpt-4"
    }
  },
  "convenience_distributions": true,
  "versioning": {
    "default_policy": "versioned",
    "overrides": {
      "temp-dataset": "unversioned",
      "scratch-notes": "unversioned" 
    }
  }
}
```