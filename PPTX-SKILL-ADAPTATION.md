# Adapting the claude-pptx-skill to Any Harness

The [claude-pptx-skill](https://github.com/inprealpha/claude-pptx-skill) was written for **Claude Code** — a CLI tool where Claude has direct shell access and can run arbitrary commands as native tool calls. Most other harnesses don't work that way. This guide explains what the skill actually needs, and how to satisfy those needs in any environment.

---

## What the skill actually requires

Strip away the Claude Code assumption and there are four real dependencies:

| Dependency | What it's for | Provided by |
|---|---|---|
| **Node.js + pptxgenjs** | Generating `.pptx` files from scratch | npm global install |
| **pdftotext / markitdown** | Extracting text from existing `.pptx` or `.pdf` files | poppler-utils or pip |
| **LibreOffice headless** | Converting `.pptx` → PDF for thumbnail generation | system package |
| **pdftoppm** | Converting PDF pages → JPEG slide images | poppler-utils |

The skill's `SKILL.md` and `scripts/` are just orchestration wrappers around these four tools. You don't need the scripts; you need the underlying tools.

---

## Harness comparison

### Claude Code (original target)

Claude Code gives Claude a `bash` tool that runs shell commands directly in your terminal. The skill works as-is:

```
# Load the skill globally
echo "@/path/to/claude-pptx-skill/SKILL.md" >> ~/.claude/CLAUDE.md

# Claude can then call tools directly:
# bash("node build-slide.js")
# bash("pdftoppm -jpeg -r 150 output.pdf slides/slide")
```

**What Claude does:** reads `SKILL.md`, plans the deck, writes JS, runs `node`, runs `pdftoppm`, inspects thumbnails — all in one uninterrupted agent loop.

---

### VS Code Copilot (Orchestrator / Agent mode)

This is what was used to build the SIA Q3 deck. Copilot in agent mode has no direct shell tool — it delegates all execution to **subagents**.

**Adaptation pattern:**

1. **Fetch skill files at runtime** instead of loading from disk.  
   The orchestrator fetches `SKILL.md`, `structured.md`, and `pptxgenjs.md` from the GitHub raw URLs and passes the relevant sections to subagents in their prompts.

2. **Delegate shell execution to Implementer/Explorer subagents.**  
   The Orchestrator never runs shell commands itself. It writes the intent and the subagent runs `node build-pptx.js` and `pdftoppm` inside its own tool call.

3. **Pass full context in every subagent prompt.**  
   Subagents are stateless — they get zero conversation history. The orchestrator must paste:  
   - The pptxgenjs API rules (no `#` in hex colors, no shared option objects, `bullet` at top level, etc.)  
   - The structured argument formatting rules (action titles, source lines, color palette)  
   - The data to visualise  
   - The exact file paths  
   - The acceptance criteria

4. **Visual QA via code-based layout analysis.**  
   The `thumbnail.py` script and subagent visual inspection loop from `SKILL.md` can work here too, but the Reviewer subagent doesn't have vision. Substitute with: (a) coordinate math in the build script itself to verify no overlaps, or (b) render to JPEG (`libreoffice --headless` + `pdftoppm`) and inspect with a capable vision model.

5. **LibreOffice rendering differs from PowerPoint.**  
   Key gotcha: LibreOffice renders text boxes with more internal leading than PowerPoint. A title at `y: 0.28, h: 0.6` that fits in PowerPoint will bleed into the chart below in LibreOffice. **Add 0.15–0.2" extra clearance between any text box and the element below it when you expect LibreOffice to be in the QA loop.**

---

### Claude API (direct, no harness)

You're calling the Anthropic API yourself. Claude has whatever tools you give it.

**Adaptation pattern:**

- Give Claude a `bash` tool (or equivalent) that runs shell commands server-side.  
- Pass `SKILL.md` content in the system prompt, or in a `<document>` user turn before the request.  
- Claude will write the JS, call `bash("node build.js")`, and iterate — same as Claude Code.

If you're running serverless or can't expose a shell tool, use the **pptxgenjs-only approach**: have Claude produce a complete self-contained `.js` file as a text response, then execute it yourself in your backend.

---

### Cursor / Windsurf / other IDE agents

These are Claude Code equivalents for different editors. The skill works identically:

- Load `SKILL.md` via a project-level rules file (e.g. `.cursor/rules`, `.windsurfrules`) instead of `~/.claude/CLAUDE.md`
- The agent has shell access; all commands run directly

The only difference is how to load the skill:

```
# Cursor — add to .cursor/rules
@/path/to/claude-pptx-skill/SKILL.md

# Windsurf — add to .windsurfrules  
@/path/to/claude-pptx-skill/SKILL.md
```

---

### No local shell access (cloud notebooks, sandboxed environments)

If you can't run `node` or `libreoffice` locally, use a hosted runtime:

- **Google Colab / Jupyter**: Install deps in a cell (`!npm install -g pptxgenjs`, `!apt-get install libreoffice poppler-utils -qq`), run the generated JS via `!node build.js`, and display thumbnails inline with `IPython.display.Image`.
- **GitHub Actions / CI**: Same pattern in a workflow step. Artifacts the PPTX.
- **Modal / Fly.io**: Containerise with Node + LibreOffice + poppler. Call the container from your agent as an HTTP tool.

---

## The minimal portable pattern

No matter what harness you're in, these three things never change:

```
1. Write a self-contained build-pptx.js using pptxgenjs
2. Run: node build-pptx.js
3. QA: libreoffice --headless --convert-to pdf output.pptx && pdftoppm -jpeg -r 150 output.pdf slides/slide
```

Everything else — how you get Claude to write the JS, how you run it, how you inspect the output — is harness-specific plumbing.

---

## Checklist when porting to a new harness

- [ ] Node.js available and `pptxgenjs` installed (globally or locally)  
- [ ] Shell execution possible (directly or via subagent/tool)  
- [ ] LibreOffice or equivalent available for PDF conversion  
- [ ] `pdftoppm` (poppler) available for slide thumbnails  
- [ ] Skill content (`SKILL.md`, `structured.md`, `pptxgenjs.md`) loaded into context — either from disk, fetched from GitHub raw URL, or pasted into system prompt  
- [ ] If subagents are stateless: all pptxgenjs rules and formatting spec copied into every Implementer prompt  
- [ ] If LibreOffice is in the QA loop: add +0.15–0.20" clearance below every text box  
