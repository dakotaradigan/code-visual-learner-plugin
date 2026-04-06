---
name: codebase-map
description: >
  This skill should be used when the user asks to "map this codebase",
  "generate architecture diagrams", "create a visual guide", "codebase map",
  "visual architecture", "mermaid diagrams for this project",
  "visual onboarding doc", "visualize the code", or wants an HTML document
  with Mermaid diagrams showing system context, module maps, request flows,
  class relationships, data flows, dependency graphs, and state lifecycles.
  Produces a single self-contained HTML file with pastel-colored diagrams,
  educational prose, and real code snippets.
---

# Codebase Map — Visual Architecture Generator

Generate a single self-contained HTML file that teaches a codebase's architecture
through progressive Mermaid diagrams, educational prose, and real code snippets.
Target audience: product managers, new engineers, executives — visual learners
who need to understand a system without reading every file.

Output: `docs/architecture/<project-name>-visual-guide.html` in the current project.

## Phase 1: Explore the Codebase

Launch Explore agents to gather architecture information in parallel:

1. Read `README.md`, `CLAUDE.md`, `ONBOARDING.md`, and manifest files
   (`package.json`, `pyproject.toml`, `Gemfile`, `go.mod`, `Cargo.toml`)
2. List the top-level directory structure and detect the project type:
   full-stack, backend-only, frontend-only, CLI, library, microservices, monorepo
3. Read entry points (main files, route definitions, index files)
4. Trace the primary request or data flow through the system
5. Map imports between modules to understand the dependency graph
6. Identify classes, dataclasses, models, and their relationships
7. Identify security or validation layers (middleware, guards, validators)
8. Select 2-3 of the most important subsystems for deep dives

Do not read files speculatively. Parallelize reads where possible.

## Phase 2: Plan the Diagram Set

Choose 6-10 diagrams from this menu based on what the codebase has:

| Level | Diagram Type | When to Include |
|-------|-------------|-----------------|
| L1 | Flowchart TB — System Context | Always (5 boxes max) |
| L2 | Flowchart TB + subgraphs — Module Map | Always |
| L3 | Sequence diagram — Request/Data Flow | When there is a request pipeline or clear data flow |
| L4 | Flowchart LR — Function dispatch | When there is a routing/dispatch decision tree |
| L4 | Flowchart TB — Resolution cascade | When there is a multi-tier resolution or fallback strategy |
| L4 | Flowchart TB — Security layers | When there is authentication, validation, or security middleware |
| L4 | classDiagram — Class relationships | When there are 3+ classes/models with relationships |
| L4 | Flowchart TB — Data transformations | When data changes shape as it moves through layers |
| L4 | Flowchart TB — Dependency graph | Always (shows which modules import which) |
| L4 | stateDiagram-v2 — State lifecycle | When there are stateful objects (sessions, connections, jobs) |

Adapt to the codebase type:

| Type | Typical Diagram Focus |
|------|----------------------|
| Full-stack web app | All levels; sequence diagram traces browser-to-DB |
| Backend API | Module map, request flow, auth layers, data models |
| Frontend SPA | Component tree, state management, API call flow |
| CLI tool | Command map, execution flow, argument parsing |
| Library/SDK | Public API surface, internal modules, call flow |
| Microservices | Service map, cross-service sequence, shared contracts |

## Phase 3: Generate the HTML File

1. Read `${CLAUDE_PLUGIN_ROOT}/skills/codebase-map/assets/template.html` for the CSS,
   Mermaid rendering script, and HTML structural patterns
2. Read `${CLAUDE_PLUGIN_ROOT}/skills/codebase-map/references/mermaid-rules.md` for
   syntax rules — **do this before writing any diagram**
3. Copy the CSS and Mermaid JS rendering block verbatim from the template
4. Generate all project-specific content:
   - Hero section with project name and one-line description
   - Table of contents with one entry per section
   - Color legend (from template, always the same)
   - For each section:
     - Level tag (L1/L2/L3/L4) with CSS class
     - Title and "What you'll learn" intro sentence
     - 2-3 paragraphs of educational prose explaining the architecture at this level
     - Diagram container `<div>` with a unique ID (e.g., `d-level1`, `d-level2`)
     - 1-2 code snippets (5-15 lines) with `file:line` references
     - A "Key Insight" callout explaining WHY, not just WHAT
5. Define all Mermaid diagrams in the JS `diagrams` object as template literal strings
6. Write the complete file to `docs/architecture/<project-name>-visual-guide.html`
7. Create the `docs/architecture/` directory if it does not exist

## Phase 4: Present

Open the HTML file in the user's default browser:
- macOS: `open <path>`
- Linux: `xdg-open <path>`
- Windows: `start <path>`

Report the file path.

## Critical Rules

- Maximum 15 nodes per diagram
- Read `references/mermaid-rules.md` before generating ANY Mermaid code
- Diagrams must be defined in JS template literals, never in `<pre class="mermaid">` tags
- Code snippets: 5-15 lines, always with `file:line` references
- Key Insight callouts must explain WHY, not just WHAT
- Use the pastel color palette consistently across all diagrams

## Additional Resources

- **`references/mermaid-rules.md`** — Mermaid syntax rules, pastel color palette, diagram type guidance, common mistakes
- **`assets/template.html`** — HTML shell with CSS, JS rendering logic, and structural patterns

## Related

For a detailed written architecture analysis (markdown with tables, ASCII diagrams,
and line-by-line analysis), use the `architecture-overview` skill from this plugin.
