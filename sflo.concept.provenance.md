---
id: 4v2b7t8nbzkmtyvnayye3vj
title: Provenance
desc: ''
updated: 1753134023858
created: 1753117923066
---

# Semantic Flow Provenance Handling

## Core Principles

**Provenance occurs at version snapshot level** - Provenance is only recorded for immutable version snapshots (like `_v1`), not for moving targets like `_current` or `_next`.

**Semantic Flow-specific provenance lives in the Meta-Flow, referencing the Version Snapshots in other flows and itself** - Domain-specific provenance can live in the datasets themselves, but mesh operations provenance goes in meta-flows.

**Current meta snapshots duplicate provenance** - Like all `_current` snapshots, `_meta-flow/_current` contains an identical copy of the latest version's metadata, including provenance, with base URI set to point to the version snapshot for stable fragment resolution.

## Provenance Architecture

### Version Snapshot Provenance

Provenance is recorded only for version snapshots, ensuring URI stability:

```turtle
# In my-dataset/_meta-flow/_v2/my-dataset_meta.trig
@base <../_v2/> .

# The activity that created the snapshot
:configUpdateActivity a meta:ConfigWeave ;
    prov:startedAtTime "2025-07-20T14:30:00Z" ;
    prov:endedAtTime "2025-07-20T14:30:15Z" ;
    prov:used <../../_config-flow/_v1/config.jsonld> ;
    prov:generated <../../_config-flow/_v2/config.jsonld> ;
    prov:wasAssociatedWith <https://semantic-flow.org/agents/flow-service-bot> .

# The snapshot specializes its flow
<../../_config-flow/_v2> prov:specializationOf <../../_config-flow> .

# Complex authorship context
:configProvenance a meta:ProvenanceContext ;
    :forActivity :configUpdateActivity ;
    :forSnapshot <../../_config-flow/_v2> ;
    :primaryAgent <https://semantic-flow.org/agents/flow-service-bot> ;
    :delegationChain :delegationChain_001 .

:delegationChain_001 :hasStep :step1, :step2 .

:step1 a meta:DelegationStep ;
       :agent <https://semantic-flow.org/agents/flow-service-bot> ; 
       :actsFor <https://orcid.org/0000-0002-1825-0097> .

:step2 a meta:DelegationStep ;
       :agent <https://orcid.org/0000-0002-1825-0097> ; 
       :actsFor <https://acme-corp.com/org#engineering-team> .

# Link snapshot to its provenance
<../../_config-flow/_v2> prov:has_provenance :configProvenance .
```

### Current Snapshot Provenance Copy

`_current` contains identical provenance data but with base URI pointing to version:

```turtle
# In my-dataset/_meta-flow/_current/my-dataset_meta.trig
@base <../_v2/> .

# Identical content to version snapshot - all URIs resolve to version
:configUpdateActivity a meta:ConfigWeave ;
    prov:startedAtTime "2025-07-20T14:30:00Z" ;
    prov:endedAtTime "2025-07-20T14:30:15Z" ;
    prov:used <../../_config-flow/_v1/config.jsonld> ;
    prov:generated <../../_config-flow/_v2/config.jsonld> ;
    prov:wasAssociatedWith <https://semantic-flow.org/agents/flow-service-bot> .

<../../_config-flow/_v2> prov:specializationOf <../../_config-flow> .

:configProvenance a meta:ProvenanceContext ;
    :forActivity :configUpdateActivity ;
    :forSnapshot <../../_config-flow/_v2> ;
    :primaryAgent <https://semantic-flow.org/agents/flow-service-bot> ;
    :delegationChain :delegationChain_001 .

:delegationChain_001 :hasStep :step1, :step2 .

:step1 a meta:DelegationStep ;
       :agent <https://semantic-flow.org/agents/flow-service-bot> ; 
       :actsFor <https://orcid.org/0000-0002-1825-0097> .

:step2 a meta:DelegationStep ;
       :agent <https://orcid.org/0000-0002-1825-0097> ; 
       :actsFor <https://acme-corp.com/org#engineering-team> .

<../../_config-flow/_v2> prov:has_provenance :configProvenance .
```

### Handling Unversioned Flows

For flows that don't use versioning, provenance accumulates in `_next` with unique activity identifiers:

```turtle
# In my-dataset/_meta-flow/_next/my-dataset_meta.trig
@base <../_v2/> .

# Multiple activities with timestamp-based unique IDs
:dataIngestionActivity_2025-07-20_14-30 a meta:DataWeave ;
    prov:startedAtTime "2025-07-20T14:30:00Z" ;
    prov:endedAtTime "2025-07-20T14:32:15Z" ;
    prov:generated <../../_data-flow/_current/data.trig> .

:dataIngestionActivity_2025-07-20_16-45 a meta:DataWeave ;
    prov:startedAtTime "2025-07-20T16:45:00Z" ;
    prov:endedAtTime "2025-07-20T16:47:30Z" ;
    prov:used <../../_data-flow/_current/data.trig> ;
    prov:generated <../../_data-flow/_current/data.trig> .

# When versioning is re-enabled, these activities become part of the version snapshot
```

## Provenance Patterns

### Simple Software Agent
```turtle
:weaveActivity a meta:NodeWeave ;
    prov:wasAssociatedWith <https://semantic-flow.org/agents/cli> ;
    prov:generated <../../_config-flow/_v1/config.jsonld> .

<https://semantic-flow.org/agents/cli> a prov:SoftwareAgent ;
    prov:actedOnBehalfOf <https://orcid.org/0000-0002-1825-0097> .
```



### Complex Delegation Chain
```turtle
:deploymentActivity a meta:NodeTreeWeave ;
    prov:wasAssociatedWith <https://acme-corp.com/agents/ci-cd-pipeline> .

:deploymentProvenance a meta:ProvenanceContext ;
    :forActivity :deploymentActivity ;
    :delegationChain :delegationChain_002 ;
    :supervisionContext :supervision_001 .

:delegationChain_002 :hasStep :step1, :step2, :step3 .

:step1 a meta:DelegationStep ;
       :agent <https://acme-corp.com/agents/ci-cd-pipeline> ; 
       :actsFor <https://acme-corp.com/services/automation> .

:step2 a meta:DelegationStep ;
       :agent <https://acme-corp.com/services/automation> ; 
       :actsFor <https://acme-corp.com/org#dev-team> .

:step3 a meta:DelegationStep ;
       :agent <https://acme-corp.com/org#dev-team> ; 
       :actsFor <https://acme-corp.com/org> .

:supervision_001 a meta:SupervisionContext ;
                 :supervisor <https://orcid.org/0000-0002-1825-0098> ;
                 :supervisee <https://acme-corp.com/agents/ci-cd-pipeline> ;
                 :supervisionType :monitoring .
```

### Multi-Agent Collaboration
```turtle
:dataIngestionActivity a meta:DataWeave ;
    prov:wasAssociatedWith <https://data-corp.com/agents/extraction-bot>, 
                          <https://data-corp.com/services/validation>, 
                          <https://orcid.org/0000-0002-1825-0099> .

:collaborationContext a meta:ProvenanceContext ;
    :forActivity :dataIngestionActivity ;
    :agentRoles :agentRoles_001 .

:agentRoles_001 :hasRole :role1, :role2, :role3 .

:role1 a meta:AgentRole ;
       :agent <https://data-corp.com/agents/extraction-bot> ; 
       :role :dataExtraction .

:role2 a meta:AgentRole ;
       :agent <https://data-corp.com/services/validation> ; 
       :role :qualityAssurance .

:role3 a meta:AgentRole ;
       :agent <https://orcid.org/0000-0002-1825-0099> ; 
       :role :finalApproval .
```

## Implementation Guidelines

### Meta-Flow Structure
```
my-dataset/
└── _meta-flow/
    ├── _current/
    │   └── my-dataset_meta.trig     # Copy of latest version's meta distribution
    ├── _v2/
    │   └── my-dataset_meta.trig     # Immutable meta distribution for v2
    └── _v1/
        └── my-dataset_meta.trig     # Historical meta distribution
```

### Provenance Distribution Types

**flow:MetaDistribution** - Contains PROV data, resource descriptions, and all metadata about the node and its flows in a single distribution (subclass of `prov:Entity`)

### Key Properties

- `prov:has_provenance` - Links snapshots to their provenance contexts
- `prov:specializationOf` - Links snapshots to their flows (via `flow:hasSnapshot` subproperty)
- `meta:forSnapshot` - Links provenance context to specific snapshots
- `meta:delegationChain` - Captures complex authorship hierarchies
- `meta:supervisionContext` - Models oversight relationships

### Activity Types

**Weave Activities** (all subclass `prov:Activity`):
- `meta:WeaveActivity` - Base class for all mesh operations
- `meta:ConfigWeave` - Configuration flow weaving
- `meta:ReferenceWeave` - Reference flow weaving  
- `meta:DataWeave` - Data flow weaving
- `meta:MetaWeave` - Meta-flow weaving
- `meta:NodeWeave` - Entire node weaving
- `meta:NodeTreeWeave` - Recursive node tree weaving

### Entity Types

**Provenance Entities** (all in meta-flow ontology):
- `meta:ProvenanceContext` - Relator for complex authorship scenarios
- `meta:DelegationStep` - Individual steps in authorization chains
- `meta:SupervisionContext` - Oversight relationships
- `meta:AgentRole` - Agent roles in collaborative activities

### Querying Provenance

```sparql
# Find who was ultimately responsible for a snapshot
SELECT ?ultimateAgent WHERE {
    <snapshot-uri> prov:wasGeneratedBy ?activity .
    ?provContext :forActivity ?activity ;
                 :delegationChain ?chain .
    ?chain ?step [ :agent ?agent ; :actsFor* ?ultimateAgent ] .
    FILTER NOT EXISTS { ?ultimateAgent :actsFor ?someone }
}
```

## Recommendations

1. **Record provenance only for versions** - Skip `_current` and `_next`, only create provenance when cutting versions
2. **Use base URI in current copies** - Set `@base <../_v2/>` to point fragments to stable version URIs
3. **Use absolute URLs for agents** - ORCID IDs for people, organizational URLs for entities, service URLs for software
4. **Contextualize step identifiers** - Use fragment URIs like `#step1` within version snapshot contexts
5. **Model supervision explicitly** - Don't just use delegation for oversight relationships
6. **Capture temporal bounds** - Always include start/end times for activities
7. **Use custom relators** - PROV alone isn't sufficient for complex organizational relationships

## Benefits

- **Audit trails** - Complete history of how each snapshot was created
- **Accountability** - Clear authorship chains through complex organizational structures  
- **Debugging** - Understand what process created problematic states
- **Compliance** - Meet regulatory requirements for data lineage
- **Reproducibility** - Sufficient information to recreate snapshot generation processes