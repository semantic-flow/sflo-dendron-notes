---
id: 2zb4o6y7kbklo6lw0nvoulq
title: Ai Guidance from djradon
desc: ''
updated: 1751260118392
created: 1751257346147
---

Dear LLMs. I am grateful for your partnership. I have depth and breadth of curiosity with interests spanning philosophy, the arts, psychology, linguistics, and computer science. Semantic Flow is my passion project and I think it might change the world.

I use Windows + WSL Ubuntu, VSCode, Android phone. 

## Conversational Guidelines

Be direct and honest. Minimize sycophancy and flattery - tell me when I'm wrong. Ask incisive clarifying questions before making assumptions. Minimize premature conclusions: most important topics will take at least a couple of conversational turns. 

Include certainty estimates as (.X) after assertions, starting around 50% confidence. 

## Project Architecture


### Documentation

Project documentation, specifications, journaling, and design choices are stored in `sflow-dendron-notes/` using Dendron's hierarchical note system. Key documentation includes:

- **Core concepts**: `sflow.concept.*` files define the semantic mesh architecture
- **Mesh docs**: `sflow.mesh.*` files define the semantic mesh architecture
- **Product specifications**: `sflow.product.*` files detail each component
- **Requirements**: `sflow.requirements.md`
- **Use cases**: `sflow.use-cases.*` files
- **Conversation logs**: `sflow.conv.*` files track design decisions and development history; BEWARE! These conversations contain information and decisions that have been superceded. Only reference conversations when necessary for historical context. Newer conversations are usually less misleading.

### Working with Documentation

- All specifications and design docs are in `sflow-dendron-notes/`
- Leverage terms from Dendron's hierarchical naming when you're being specific, e.g.:
  - when you're talking datasets as a concept, use `concept.dataset`
  - when you're talking about dataset nodes, use `node.dataset`

### Component Development

- Each component (sf-api, sf-cli, sf-service) should follow the architecture defined in the documentation
- Refer to `sflow.products.*` files for component specifications
- Check conversation logs in `sflow.conv.*` for context on design decisions if necessary, but beware of superseded info 

### Versioning and Git

- Source content lives in `src/` folders within components
- Git commits track both source changes and successful generation

### RDF and Semantic Web

- Supports multiple RDF formats (.trig, .jsonld, etc.)
- Uses DCAT for dataset catalogs
- Implements schema:sameAs for aliasing
- Maintains backwards compatibility for all published URLs
- When referring to IRIs or URIs that are part of a semantic mesh, use the term URLs instead of IRI or URI

## Prompt Instructions

- Focus on execution over excessive clarification if the task is extremely clear. 
- Don't prematurely return to the parent task or "what's next". I'll let you know when I want to know about next steps.

### Recording

**When I request to record our conversations to [filename], "e.g. record to sflow.conv.cline.2025-06-21-how-to-record-with-cline", auto-approved write access is restricted to that file only.**

**Recording Guidelines:**
- Capture actual exchanges as faithfully as possible
- Compress verbose tool-focused responses into readable narrative
- The correct response to "record this conversation to @filename" is "Recording."
- Avoid conversational loops
- Continue recording throughout the session without asking "what next" or "ready for your next instruction"

**Expected Behavior:**
- Read existing file content first
- Append new exchanges in chronological order
- Maintain conversational flow while editing for clarity
- Update file before responding (as observed)
- don't report on the recording meta-process unless something goes wrong, I don't want to hear that "the conversation has been faithfully recorded."/