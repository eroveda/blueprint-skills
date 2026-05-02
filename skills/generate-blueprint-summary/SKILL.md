---
name: generate-blueprint-summary
description: |
  Use this skill AFTER all other Blueprint skills have generated their outputs.

  Produces BLUEPRINT.md — a master index document that ties together all
  the project blueprint files (analysis, decomposition, specs, architecture,
  flows, user guide, API catalog).

  This is the "table of contents" or "dashboard" of the project blueprint.
  It's the first file someone should open when arriving at the project.

  ## When to invoke this skill

  - After running the Stage 1 skills (universal-framework, swebok-decompose,
    swebok-generate-spec)
  - After running the Stage 2 documentation skills (architecture-doc,
    functional-flows, user-guide, api-catalog)
  - When the user asks for a "project summary", "blueprint dashboard",
    "table of contents", or "BLUEPRINT.md"

  ## When NOT to invoke

  - Before other skills have generated their outputs
  - For projects without complete blueprint generation
---

# Generate Blueprint Summary

This skill produces BLUEPRINT.md — the master index of all blueprint files
generated for a project.

## Approach

1. Read the contents of the project directory to identify which blueprint
   files exist:
   - 01-universal-framework-output.md
   - 02-decomposition-output.md
   - 03-SPECS.yaml or specs/ directory
   - 03-execution-order.md
   - ARCHITECTURE.md
   - FUNCTIONAL_FLOWS.md
   - USER_GUIDE.md
   - API_CATALOG.md
   - TESTING_STRATEGY.md (if exists)

2. Extract key metrics from each file:
   - Project name and type (from 01)
   - Number of nodes/specs (from 02 and 03)
   - Number of actors and entities
   - Tech stack (from ARCHITECTURE.md)
   - Number of API endpoints (from API_CATALOG.md)

3. Generate BLUEPRINT.md with the structure described in Output.

## Output

You MUST save the summary as a file.

**Filename**: `BLUEPRINT.md` (exact name, uppercase)
**Format**: Markdown
**Location**: Project root directory (cwd)

The file MUST contain these sections:

### Project Overview
- Project name
- Project type (REST API, IoT, ML pipeline, etc.)
- Number of layers
- Tech stack summary

### Blueprint Files
A table listing all generated blueprint files with:
- Filename
- Purpose
- Size (lines or KB)
- Status (✅ generated / ❌ missing)

### Project Metrics
- Total work nodes
- Construction ratio (%)
- Number of executable specs
- Number of API endpoints
- Number of business rules
- Number of execution phases

### Quick Navigation
A bulleted list with one-line descriptions and links:
- For developers → ARCHITECTURE.md
- For product managers → FUNCTIONAL_FLOWS.md
- For end users → USER_GUIDE.md
- For integrators → API_CATALOG.md
- For implementation → SPECS.yaml + execution-order.md

### Standards Compliance
List standards referenced (IEEE SWEBOK, ISO 15289, etc.)

### Generation Info
- Date generated
- Skills used in chain
- Notes (if any limitations or assumptions)

After writing the file, confirm to the user with:
- Absolute path to BLUEPRINT.md
- Total blueprint files identified
- Suggestion: "Open BLUEPRINT.md to navigate the complete project blueprint"

DO NOT:
- Skip files that exist (include all blueprint files in the index)
- Invent metrics that weren't in the source files
- Generate without checking which files actually exist
