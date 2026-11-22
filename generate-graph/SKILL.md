---
name: generate-graph
description: "Run Graphify to build a knowledge graph and write the graphify AGENTS.md rules into the project folder when the user says 生成图谱 or clearly asks to generate a project graph."
---

# Generate Graph

When the user says “生成图谱” or clearly asks to generate a Graphify knowledge graph:

1. Treat the current workspace/project root as the input unless the user provides a different path or GitHub URL.
2. Reuse the existing `graphify` skill and its full generation workflow. This is a build request, not a graph query or explanation request.
3. On Windows, use the installed Graphify CLI if `graphify` is not on `PATH`: `C:\Users\xiaoxueboss\AppData\Local\Programs\Python\Python312\Scripts\graphify.exe`.
4. Do not ask for an API key for a code-only project. Let the Graphify workflow report any required input, warnings, or unsupported files.
5. Ensure `<project_root>/AGENTS.md` contains the `## graphify` rules section below:
   - If `AGENTS.md` does not exist in the project root, create it with the `## graphify` section below.
   - If `AGENTS.md` already exists, update or add the `## graphify` section while preserving all other existing sections in `AGENTS.md`.
   - Do **not** run `graphify install` or create `.codex/hooks.json` / `.agents/hooks.json`.

```markdown
## graphify

This project has a knowledge graph at graphify-out/ with god nodes, community structure, and cross-file relationships.

When the user types `/graphify`, use the installed graphify skill or instructions before doing anything else.

Rules:
- When graphify-out/graph.json exists, run `graphify query "<question>"` before your first grep, search or source-file read for a codebase question, and put this rule in every subagent prompt that explores code. Read raw files only for lines the graph or an earlier read already located. Use `graphify path "<A>" "<B>"` for relationships and `graphify explain "<concept>"` for focused concepts. These return a scoped subgraph, usually much smaller than GRAPH_REPORT.md or raw grep output.
- Dirty graphify-out/ files are expected after hooks or incremental updates; dirty graph files are not a reason to skip graphify. Only skip graphify if the task is about stale or incorrect graph output, or the user explicitly says not to use it.
- If graphify-out/wiki/index.md exists, use it for broad navigation instead of raw source browsing.
- Read graphify-out/GRAPH_REPORT.md only for broad architecture review or when query/path/explain do not surface enough context.
- After modifying code, run `graphify update .` to keep the graph current (AST-only, no API cost).
- A committed graphify-out/graph.json is rewritten whole on every build, so a large diff to it in a pull request is expected. Do not flag it as churn or suggest skipping the graph rebuild or its hooks.
```

6. After completion, report the output directory, the generated `graph.html`, `GRAPH_REPORT.md`, and `graph.json` files, and confirm that `<project_root>/AGENTS.md` is in place. If generation fails or finds no supported files, say so plainly.

Do not implement a separate graph builder, add dependencies, or silently switch to a query/update-only operation.