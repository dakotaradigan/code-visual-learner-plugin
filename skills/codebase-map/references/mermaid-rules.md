# Mermaid Diagram Rules

## Rendering Approach

Use a `<script type="module">` block with the Mermaid ESM CDN import.
Define all diagrams in a JavaScript object mapping div IDs to template literal strings.
Render with the `mermaid.render()` API loop.

```javascript
import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs';
mermaid.initialize({ startOnLoad: false, theme: 'default', securityLevel: 'loose' });

const diagrams = {
  "d-level1": `flowchart TB
      A["Node"] --> B["Node"]`,
};

for (const [id, content] of Object.entries(diagrams)) {
  try {
    const { svg } = await mermaid.render(id.replace(/-/g, '_') + '_svg', content);
    document.getElementById(id).innerHTML = svg;
  } catch (err) {
    document.getElementById(id).innerHTML =
      '<p style="color:red;">Diagram error: ' + err.message + '</p>' +
      '<pre style="font-size:0.75rem;background:#f4f4f4;padding:12px;">' +
      content.replace(/</g, '&lt;') + '</pre>';
  }
}
```

**Why NOT `<pre class="mermaid">`:** HTML entity encoding issues. `&` becomes `&amp;`
in HTML context, `<` starts a tag. Template literals bypass all encoding problems.

## Syntax Rules (CRITICAL)

These rules prevent the most common rendering failures:

1. **`&` must be `&amp;` in flowchart labels.** Mermaid renders to SVG (XML), where
   bare `&` is invalid. Example: `S&P 500` must be `S&amp;P 500` in flowchart nodes
   and edge labels. (Does NOT apply to sequence diagram messages.)

2. **Avoid `<` and `>` in flowchart labels.** Use "above", "below", "greater than",
   "less than" instead. These characters conflict with SVG/XML parsing.

3. **Cross-subgraph edges MUST be defined OUTSIDE all subgraph blocks.** If node A
   is in subgraph X and node B is in subgraph Y, the edge `A --> B` must appear
   AFTER both subgraphs close. Defining it inside a subgraph pulls the target node
   into the wrong subgraph.

   ```
   subgraph X
       A["Node A"]
   end
   subgraph Y
       B["Node B"]
   end
   A --> B   %% Correct: outside both subgraphs
   ```

4. **Sequence diagram participants do NOT support `<br/>`.** Keep participant labels
   single-line. Multi-line labels only work in flowchart node labels.

5. **No emojis in diagram labels.** Some renderers fail on Unicode emoji characters.
   Use text labels instead.

6. **Use `\\n` for line breaks in flowchart node labels** (inside template literals,
   the backslash is already escaped, so write `\\n`).

7. **Maximum 15 nodes per diagram.** Comprehension drops sharply past this. Split
   into multiple diagrams if needed.

8. **Put labels with spaces in quotes:** `NodeID["Label with spaces"]`

9. **Avoid special characters in node IDs.** Use alphanumeric + underscore only.

10. **Use `direction LR` inside subgraphs** for horizontal layout when appropriate.

## Pastel Color Palette (Excalidraw-inspired)

Apply these consistently across ALL diagrams using `classDef`:

| Role | Fill | Text | Stroke | classDef name |
|------|------|------|--------|---------------|
| Frontend / UI | #a5d8ff | #1e1e1e | #339af0 | `frontend` |
| Backend / Logic | #b2f2bb | #1e1e1e | #51cf66 | `backend` |
| Security | #ffc9a9 | #1e1e1e | #fd7e14 | `security` |
| AI / LLM | #d0bfff | #1e1e1e | #7950f2 | `ai` |
| Data / Storage | #ffec99 | #1e1e1e | #fab005 | `data` |
| External Service | #ffc9c9 | #1e1e1e | #fa5252 | `external` |
| User / Neutral | #e9ecef | #1e1e1e | #adb5bd | `user` |

Ready-to-copy classDef declarations:

```
classDef frontend fill:#a5d8ff,color:#1e1e1e,stroke:#339af0,stroke-width:2px
classDef backend fill:#b2f2bb,color:#1e1e1e,stroke:#51cf66,stroke-width:2px
classDef security fill:#ffc9a9,color:#1e1e1e,stroke:#fd7e14,stroke-width:2px
classDef ai fill:#d0bfff,color:#1e1e1e,stroke:#7950f2,stroke-width:2px
classDef data fill:#ffec99,color:#1e1e1e,stroke:#fab005,stroke-width:2px
classDef external fill:#ffc9c9,color:#1e1e1e,stroke:#fa5252,stroke-width:2px
classDef user fill:#e9ecef,color:#1e1e1e,stroke:#adb5bd,stroke-width:2px
```

Assign roles based on what the component DOES, not what it IS.
A Python file that handles security gets `security`, not `backend`.

## Diagram Type Guidance

### Flowchart TB (top-to-bottom)
Best for: system context, module maps, security layers, data flow.
Use subgraphs to group related nodes. Apply `direction LR` inside subgraphs
when horizontal layout is clearer.

### Flowchart LR (left-to-right)
Best for: function dispatch, decision trees, pipelines with many sequential steps.

### Sequence Diagram
Best for: request flows, time-ordered interactions between components.
- Participant order matters — put the initiator on the left
- Use `rect rgb(R, G, B)` to group related steps (e.g., security gate, LLM loop)
- Use `+`/`-` for activation bars: `A->>+B` starts, `B-->>-A` ends
- Pastel rect colors: `rgb(255, 224, 195)` (peach), `rgb(210, 248, 215)` (mint)

### Class Diagram
Best for: showing classes, attributes, methods, and relationships.
- `+` = public, `-` = private, `#` = protected
- `A <|-- B` = B inherits A
- `A *-- B` = A contains B (composition)
- `A o-- B` = A has B (aggregation)
- `A --> B` = A uses B (dependency)
- Use `direction LR` for horizontal layout

### stateDiagram-v2
Best for: session lifecycle, connection states, job states.
- `[*]` = start/end state
- `state "Label" as S { ... }` = composite state
- `S1 --> S2 : trigger` = transition with label
- `note right of S : text` = annotation
- Wrap independent state machines in their own `state` blocks

## Common Mistakes

**Mistake:** `A -->|"Score > 0.80"| B` in a flowchart
**Fix:** `A -->|"Score above 0.80"| B`
**Why:** `>` can break SVG rendering in some environments.

**Mistake:** Edge `T1 --> T2` defined inside `subgraph Tier1` but T2 is in `subgraph Tier2`
**Fix:** Move `T1 --> T2` to after both subgraphs close.
**Why:** Mermaid pulls T2 into Tier1's subgraph, breaking the layout.

**Mistake:** `participant FE as Frontend<br/>(app.js)` in a sequence diagram
**Fix:** `participant FE as Frontend`
**Why:** `<br/>` is not supported in sequence diagram participant labels.

**Mistake:** `A["S&P 500"]` in a flowchart
**Fix:** `A["S&amp;P 500"]`
**Why:** SVG/XML requires `&` to be escaped as `&amp;`.
