# code-visual-learner

A Claude Code plugin that generates visual and written architecture documentation for any codebase. Point it at a project and get diagrams, tables, and explanatory prose that make the architecture legible to engineers, PMs, and executives.

## Skills

### `codebase-map`

Generates a single self-contained HTML file with Mermaid diagrams, educational prose, and  code snippets. Produces progressive diagrams from system context (L1) down to state lifecycles (L4) using a pastel color palette.

**Trigger phrases:** "map this codebase", "visual architecture", "mermaid diagrams for this project", "visualize the code"

**Output:** `docs/architecture/<project-name>-visual-guide.html`

### `architecture-overview`

Generates a structured markdown document analyzing language patterns, routing tables, concurrency models, dependency graphs, and overall architecture with ASCII diagrams and `file:line` references.

**Trigger phrases:** "write an architecture doc", "architecture analysis", "explain this codebase", "how is this project structured"

**Output:** `docs/architecture/<project-name>-architecture.md`

## Install

Add the plugin to your Claude Code configuration:

```
claude plugin add /path/to/code-visual-learner
```

Or clone and add:

```
git clone https://github.com/dakotaradigan/code-visual-learner-plugin.git
claude plugin add ./code-visual-learner-plugin
```

## Usage

From any project directory with Claude Code running:

```
/codebase-map
/architecture-overview
```

Both skills auto-detect the project's language, framework, and structure. No configuration needed.

## Agent Usage

Agents can invoke these skills directly. The skills are designed to be triggered by natural language — any of the trigger phrases listed above will activate the corresponding skill. Both skills:

- Accept no required arguments (they analyze the current working directory)
- Produce deterministic output paths under `docs/architecture/`
- Include `file:line` references for traceability back to source code
- Handle multi-language and multi-framework projects

## How It Works

Each skill follows a three-phase pattern:

1. **Explore** — Read manifests, entry points, and trace data/request flows through the codebase
2. **Generate** — Produce the output document using reference templates and formatting rules
3. **Present** — Write the file and report the path (codebase-map also opens it in the browser)

## License

MIT
