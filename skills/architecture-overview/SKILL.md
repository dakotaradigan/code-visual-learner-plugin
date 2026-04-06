---
name: architecture-overview
description: >
  This skill should be used when the user asks to "write an architecture doc",
  "generate architecture overview", "document the codebase architecture",
  "architecture analysis", "codebase analysis", "explain this codebase",
  "create an onboarding doc for new engineers", "technical overview for PMs",
  "how is this project structured", or wants a structured markdown document
  analyzing language patterns, routes, concurrency model, dependencies, and
  overall architecture with ASCII diagrams and file:line references.
  Produces a markdown file with tables, code trees, and explanatory prose.
---

# Architecture Overview — Structured Codebase Analysis

Generate a comprehensive markdown document that analyzes a codebase's architecture
with tables, ASCII diagrams, import trees, and explanatory prose.
Target audience: product managers, new engineers, executives — readers who understand
software concepts but are not reading the code daily.

Output: `docs/architecture/<project-name>-architecture.md` in the current project.

## Phase 1: Explore the Codebase

1. Read `README.md`, `CLAUDE.md`, `ONBOARDING.md`, and manifest files to understand
   the project's purpose and tech stack
2. Detect the dominant language and framework from manifests and file extensions
3. List all source files with line counts
4. Read entry points and trace the primary request or data flow
5. Grep for language-specific patterns:
   - Python: `@` decorators, `async def`, `from ... import`
   - JavaScript/TypeScript: middleware, `export`, `import`, `async`
   - Go: `func`, `http.Handle`, goroutines
   - Java: `@annotations`, `implements`, `extends`
   - Rust: `#[derive]`, `impl`, `async fn`
6. Identify all HTTP/WebSocket/gRPC endpoints and their handlers
7. Map internal imports between modules
8. Identify all external network calls (HTTP clients, database connections, API calls)

## Phase 2: Generate the Document

Create `docs/architecture/<project-name>-architecture.md` with these sections.
For detailed formatting rules and column headers, read
`${CLAUDE_PLUGIN_ROOT}/skills/architecture-overview/references/section-specs.md`.

### Section 1: Overview

One-paragraph summary: what this project is, tech stack, architecture style
(monolith, microservices, serverless), and primary purpose.

### Section 2: Language Patterns

Enumerate every decorator, annotation, attribute, or structural pattern in the
codebase. Split into production code vs test code. For each entry, include:
file, line number, what it decorates/annotates, and what it does in plain English.

Adapt to the language:
- Python: `@decorator` patterns
- Java/Spring: `@Annotation` patterns
- TypeScript/NestJS: `@Decorator` patterns
- Go: no decorators — skip this section with a note explaining why
- Rust: `#[attribute]` patterns

If no language patterns exist, skip the section with a brief explanation.

### Section 3: Routing Table

All HTTP, WebSocket, gRPC, or CLI command endpoints. Include: method, path,
handler function, file:line, purpose, and any middleware/auth requirements.

If the project is a library or CLI with no routes, replace with:
- **Library:** Public API surface table (exported functions/classes)
- **CLI:** Command registry table (subcommands, arguments, handlers)

### Section 4: Concurrency Model

Classify every function as async or sync (or the language's equivalent).
Group by module. For async functions, explain WHY they are async (I/O bound,
awaits external service, etc.). For sync modules, explain WHY they are sync
(pure computation, no I/O).

Assess whether the concurrency split is correct. Flag any issues found.

Adapt to the language:
- Python: async/await
- JavaScript: async/await, Promises, callbacks
- Go: goroutines, channels
- Java: threads, CompletableFuture, virtual threads
- Rust: async/await, tokio

### Section 5: Internal vs External Dependencies

Two sub-sections:

**Internal imports:** Show the full import tree as an ASCII tree diagram.
Which modules import which, and what symbols they use.

**External calls:** Table of every network call to an external service.
Include: calling file, target service/URL, method, and purpose.

Explicitly state whether this is a monolith (all function calls) or
distributed system (service-to-service HTTP).

### Section 6: Overall Architecture

An ASCII box-and-arrow diagram (max 80 columns wide) showing how all components
fit together. Include: entry points, middleware/pipeline, business logic,
data stores, and external services.

Below the diagram, list 3-5 key architectural decisions and explain each one.

### Section 7: Files Reference

Table of every source file: file path, line count, one-line role description,
and key exports.

## Phase 3: Present

Report the file path. Offer to generate the visual companion doc
(`/codebase-map`) for Mermaid diagram version.

## Quality Checklist

Before writing the file, verify:
- Every source file is accounted for in the Files Reference
- Every route/endpoint is listed in the Routing Table
- The ASCII diagram is under 80 columns
- All file:line references are accurate
- No fabricated rationale (if unsure about a design decision, say so)
- Tone is accessible to non-engineers

## Additional Resources

- **`references/section-specs.md`** — Detailed column headers, formatting rules,
  and a complete example output for reference

## Related

For interactive visual architecture diagrams (Mermaid-based HTML with pastel
colors), use the `codebase-map` skill from this plugin.
