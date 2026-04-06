# Section Specifications

Detailed formatting rules, column headers, and example patterns for each section
of the architecture overview document.

## Section 2: Language Patterns — Table Format

### Production Code Table

```markdown
| Pattern | File : Line | Decorates | What it does |
|---|---|---|---|
| `@dataclass` | `app.py:80` | `Session` | Auto-generates __init__, __repr__ for data holder |
| `@app.get(...)` | `app.py:262` | `health_check()` | Registers handler for GET /api/health |
```

### Test Code Table

```markdown
| Pattern | Files | What it does |
|---|---|---|
| `@pytest.fixture` | All test files | Marks reusable test setup function |
| `@patch()` | `test_chat.py` | Temporarily replaces object with mock |
```

### Detection Patterns by Language

**Python:** Grep for lines starting with `@` (excluding comments and strings).
Common: `@dataclass`, `@property`, `@staticmethod`, `@classmethod`, `@app.route`,
`@app.get`, `@app.post`, `@pytest.fixture`, `@pytest.mark.*`, `@patch`, `@override`

**TypeScript/NestJS:** Grep for `@Controller`, `@Get`, `@Post`, `@Injectable`,
`@Module`, `@Param`, `@Body`, `@UseGuards`

**Java/Spring:** Grep for `@RestController`, `@Service`, `@Autowired`,
`@GetMapping`, `@PostMapping`, `@Entity`, `@Override`, `@Test`

**Rust:** Grep for `#[derive(`, `#[test]`, `#[tokio::main]`, `#[cfg(test)]`

Always end the section with a plain-language summary: "Decorators here serve N
purposes: (1) ..., (2) ..., (3) .... There are no custom decorators/annotations."

## Section 3: Routing Table — Table Format

```markdown
| Method | Path | Handler | File : Line | Purpose |
|---|---|---|---|---|
| `GET` | `/api/health` | `health_check()` | `app.py:262` | Returns system status |
| `POST` | `/api/chat` | `chat_endpoint()` | `app.py:273` | Accepts chat message |
| `WebSocket` | `/ws/{session_id}` | `websocket_endpoint()` | `app.py:296` | Real-time chat |
```

Include middleware below the table:
```markdown
**Middleware** (applied to all routes):
- CORS (`app.py:248-255`) — allows requests from specified origins
```

Note if both REST and WebSocket feed into a shared pipeline.

### Framework Detection

**FastAPI:** Look for `@app.get`, `@app.post`, `@app.websocket`, `APIRouter`
**Flask:** Look for `@app.route`, `Blueprint`
**Express:** Look for `app.get(`, `router.get(`, `app.use(`
**NestJS:** Look for `@Controller`, `@Get(`, `@Post(`
**Spring:** Look for `@GetMapping`, `@PostMapping`, `@RequestMapping`
**Go net/http:** Look for `http.HandleFunc`, `mux.Handle`

## Section 4: Concurrency Model — Table Format

### Async Functions Table

```markdown
| Function | File : Line | Why async? |
|---|---|---|
| `process_message()` | `chat.py:332` | Awaits OpenAI chat completions |
| `classify_input()` | `security.py:80` | Awaits GPT-4o-mini classifier |
```

### Sync Modules Table

```markdown
| Module | Why sync? |
|---|---|
| `benchmarks.py` (~16 functions) | Pure computation — all data in memory |
| `minimum_calculator.py` (~15 functions) | Pure math — no I/O |
```

### Assessment

End with a clear verdict: "The async/sync split is [correct/has issues]."
If issues exist, explain what should change and why.

## Section 5: Dependencies — Format

### Internal Import Tree (ASCII)

```
app.py
 ├── from benchmarks import get_service
 ├── from chat import process_message, ChatResult
 ├── from security import sanitize_input, validate_input, ...
 └── from config import ALLOWED_ORIGINS, ...

chat.py
 ├── from benchmarks import get_service, FILTER_FIELDS
 ├── from benchmark_resolver import BenchmarkResolver
 └── from config import OPENAI_API_KEY, ...
```

### External Calls Table

```markdown
| Call | File | Target | Purpose |
|---|---|---|---|
| `client.chat.completions.create()` | `chat.py` | OpenAI API | GPT chat + function calling |
| `client.embeddings.create()` | `chat.py` | OpenAI API | Semantic search vectors |
```

End with: "There are [zero/N] HTTP calls between internal components.
[No database connections. No message queues.]"

## Section 6: Architecture Diagram — Rules

### ASCII Diagram Constraints
- Maximum 80 columns wide
- Use box-drawing characters: `┌ ┐ └ ┘ │ ─ ├ ┤ ┬ ┴ ┼`
- Or simple ASCII: `+`, `-`, `|`, arrows `──►`, `──▶`
- User-facing surfaces at top, data stores at bottom
- Label edges with protocols where relevant
- Group related components in a shared box

### Key Decisions List

Below the diagram, list 3-5 key architectural decisions:

```markdown
### Key architectural decisions

1. **Monolith, not microservices.** All modules run in one process.
   Communication is via function calls, not HTTP.

2. **Embedded frontend.** The backend serves HTML/CSS/JS directly.
   No separate frontend server.

3. **In-memory everything.** Data loaded from JSON at startup into
   Python dicts. No database.
```

## Section 7: Files Reference — Table Format

```markdown
| File | Lines | Role |
|---|---|---|
| `backend/app.py` | 375 | Routes, sessions, shared pipeline, middleware |
| `backend/chat.py` | ~420 | OpenAI client, function calling loop |
```

Include every source file. Sort by directory, then by importance within directory.

## Tone and Audience

- Write for someone who understands software concepts but is not reading the code
- Define jargon on first use: "decorators (the `@something` syntax in Python)"
- Use concrete language: "sends an HTTP request to OpenAI" not "interfaces with the LLM provider"
- Tables should be scannable — keep cell content short
- ASCII diagrams should be labeled clearly enough to stand alone
