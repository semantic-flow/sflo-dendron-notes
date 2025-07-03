---
id: idskfqctfpjria2byki92pt
title: 2025 06 30 Finalizing Design and Concept Docs
desc: ''
updated: 1751565550599
created: 1751267731956
---

# AI Guidance Documentation Review and Coding Standards Creation

## Initial Task

**djradon**: I'm finalizing my design and concept documentation and my prompts. Can you digest '.ai-guidance.md' and the files it mentions and let me know if anything stands out as counter-productive, confusing, or otherwise lame.

The `.ai-guidance.md` file was a simple pointer to two other files:
- `sflo-dendron-notes/sflo.contributors.general-guidance.md` 
- `sflo-dendron-notes/sflo.contributors.djradon.ai-guidance.md`

## Documentation Review Analysis

After reviewing the AI guidance structure, I identified several issues:

**Fragmented Guidance Structure**: The pointer-file approach created friction requiring AIs to read 3 files for complete guidance.

**Incomplete Technical Foundation**: The "Coding Standards" section was just "//TODO" - a significant gap for code development.

**Terminology Inconsistencies**: Mixed usage of "mesh-repo" vs "mesh repo", "URLs" vs "URIs/IRIs", and "Datasets Nodes" vs "Dataset Nodes".

**Overly Detailed Recording Instructions**: The recording section in personal guidance was extremely detailed (8 bullet points + behavioral expectations), potentially confusing rather than helpful.

**Conversational Contradictions**: Instructions to "minimize excessive clarification if the task is extremely clear" but also ask for "incisive clarifying questions before making assumptions" could conflict.

**Unclear Confidence Calibration**: The request for "certainty estimates as (.X) after assertions, starting around 50% confidence" was vague about when to include estimates.

## User Clarifications

**djradon**: The thought with .ai-guidance was, exactly to have a pointer so that I could separate out shared instructions vs contributor-specific ones. "mesh-repo" might refer to a concept that defined in documentation. Made some improvements. Help me get started with coding standards.

**Key technical preferences provided:**
- Mostly TypeScript for coding
- Deno where possible 
- RDF.js libraries likely, but open to others
- Data primarily in .trig syntax, may switch to JSON-LD
- JSON-LD should be fully supported
- No weird style preferences - just modern best practices

## Coding Standards Development Approach

Given 100+ documentation files in sflo-dendron-notes, I proposed a targeted approach rather than reading everything:

**High Priority Files:**
- Core mesh concepts
- Resource structure definitions
- Identifier/URI conventions
- Dataset handling patterns
- Component specifications

**Medium Priority:**
- Resource types and structure details
- Namespace management
- Build/processing workflow

This would cover 15-20 key files to draft comprehensive standards covering TypeScript/Deno conventions, RDF data handling, semantic mesh structure patterns, documentation-driven development, and component architecture consistency.

## Coding Standards Creation

I successfully created comprehensive coding standards covering 10 key areas:

### 1. Language & Runtime
- TypeScript with strict configuration and modern ES2022+ features
- Deno preference over Node.js
- Deno-style URL imports

### 2. RDF Data Handling  
- .trig files as primary format
- Full JSON-LD support required
- RDF.js ecosystem libraries
- URL-based identifiers following project patterns
- Validation of underscore-prefixed reserved names

### 3. Semantic Mesh Architecture
- Clear distinction between Nodes, Elements, and Resources
- Mesh folder structure validation
- System vs user element handling
- Weave operation integration

### 4. Documentation-Driven Development
- Code comments must reference docs by filename
- Interface definitions link to concept documentation
- Cross-reference validation between code and docs
- TSDoc-generated API documentation

### 5. Component Architecture
- Clear boundaries between sf-api, sf-cli, sf-service
- Consistent error handling patterns
- Async/await for RDF operations and file I/O
- TypeScript type system for validation

### 6. File Organization & Naming
- TypeScript modules with feature organization
- Co-located test files with .test.ts suffix
- Mesh resource naming convention compliance
- UPPER_SNAKE_CASE for constants

### 7. Code Style
- Deno built-in formatter and linter
- 100 character line length maximum
- 2-space indentation
- Required semicolons

### 8. Error Handling
- Custom semantic mesh error types
- Mesh resource structure validation
- Structured logging for weave operations
- Proper async error propagation

### 9. Testing
- Deno built-in test runner
- End-to-end mesh operation tests
- RDF validation for both .trig and JSON-LD
- Mock data following documentation patterns

### 10. Performance
- Streaming for large RDF files
- Caching parsed RDF during weave operations
- Consistent async file operations
- Memory-conscious dataset loading

## Technical Interruption and Recovery

During the update process, the operation was interrupted mid-edit, leaving just a "T" character where the coding standards should have been. I detected this issue and successfully completed the full standards replacement.

**djradon**: What happened? I saw you draft the coding standards but then they disappeared?

The interruption occurred during the file update, but I was able to recover and complete the comprehensive coding standards successfully. The standards are now properly integrated into the general guidance document, directly referencing the project's specific documentation files and semantic mesh architecture concepts.

## Outcome

The AI guidance documentation is now complete with comprehensive coding standards tailored specifically to the Semantic Flow project's semantic mesh architecture, TypeScript/Deno technology stack, and documentation-driven development approach. The standards provide clear direction for development work across all project components while maintaining consistency with the project's URL-based identifier system and mesh resource concepts.
