# Claude PPTX Skill (Structured Argument Fork)

A fork of the [Anthropic PPTX skill](https://github.com/anthropics/skills/tree/main/skills/pptx) with an added **Structured Argument mode** — for board decks, investor updates, strategy reviews, policy briefings, and any presentation where the audience is evaluating your reasoning, not your graphic design.

**What's new in this fork:**
- Auto-routing: Claude detects deck type (structured vs. visual) from your request
- Action titles on every slide (complete sentence = the takeaway, not a topic label)
- Ghost deck test: titles alone must form a coherent argument
- SCR / Pyramid Principle storylining
- Exhibit discipline: one chart/table per slide, tied directly to the action title
- Source citations on all data exhibits
- Grayscale + one accent color, one font, white backgrounds

See [structured.md](structured.md) for the full guidelines.

---

## Setup (one-time)

### 1. Clone this repo

```bash
git clone https://github.com/inprealpha/claude-pptx-skill.git
cd claude-pptx-skill
```

### 2. Install Python dependencies

```bash
pip install "markitdown[pptx]" Pillow defusedxml
```

> On Arch Linux or managed Python environments, add `--user --break-system-packages` if pip complains.

### 3. Install Node.js dependencies

```bash
npm install -g pptxgenjs react react-dom react-icons sharp
```

### 4. Install system packages

**Ubuntu/Debian:**
```bash
sudo apt install libreoffice poppler-utils
```

**Arch Linux:**
```bash
sudo pacman -S libreoffice-still poppler
```

**macOS:**
```bash
brew install libreoffice poppler
```

### 5. Verify everything works

```bash
# Python
python3 -c "import markitdown; import PIL; import defusedxml; print('Python deps OK')"

# Node (swap in your `npm root -g` path)
node -e "require('$(npm root -g)/pptxgenjs'); console.log('pptxgenjs OK')"

# System
libreoffice --version
pdftoppm -v
```

### 6. Load the skill in Claude Code

**Global (available in every project):**

```bash
echo "@$(pwd)/SKILL.md" >> ~/.claude/CLAUDE.md
```

**Per-project only:**

```bash
echo "@/absolute/path/to/claude-pptx-skill/SKILL.md" >> /your/project/CLAUDE.md
```

### 7. Confirm it loaded

Open Claude Code and ask: *"What PPTX skill capabilities do you have?"*

Claude should describe both structured and visual modes.

---

## How to Use It

### Structured deck (board meeting, investor update, strategy, policy brief)

Just describe what you need — Claude defaults to structured mode for business contexts:

> "Build me a 10-slide deck summarizing our Q3 results for the board."

> "Create a strategy presentation recommending we expand into the EU market."

> "Make a policy briefing deck on carbon pricing for the minister."

Claude will: draft action titles first → run ghost deck test → map exhibits → build the deck → add source citations → run visual QA.

### Visual deck (pitch, marketing, product launch)

Claude auto-detects visual contexts, or signal it explicitly:

> "Build a pitch deck for our Series A — make it look great."

> "Create a product launch deck with bold visuals."

### Edit an existing PPTX

> "Here's our template — update it with the Q3 data." *(attach a .pptx)*

### Read / extract content from a PPTX

> "What's in this presentation?" *(attach a .pptx)*

---

## File Structure

```
claude-pptx-skill/
├── SKILL.md          # Main entrypoint loaded by Claude Code
├── structured.md     # Structured argument mode (new in this fork)
├── editing.md        # Template-based editing workflow
├── pptxgenjs.md      # PptxGenJS API reference (create from scratch)
├── README.md         # This file
├── LICENSE.txt       # Anthropic proprietary license (covers scripts/)
└── scripts/
    ├── add_slide.py       # Duplicate or create slides
    ├── clean.py           # Remove orphaned files after edits
    ├── thumbnail.py       # Generate slide thumbnail grid
    └── office/
        ├── unpack.py      # Extract PPTX to editable XML
        ├── pack.py        # Repack XML back to PPTX (with validation)
        ├── soffice.py     # LibreOffice wrapper for PDF conversion
        ├── validate.py    # PPTX structure validation
        ├── helpers/       # XML merge/redline utilities
        ├── validators/    # Per-format validators
        └── schemas/       # XSD schemas for validation
```

---

## What the Structured Mode Adds

| Feature | Original skill | This fork |
|---|---|---|
| Deck type auto-routing | ✗ | ✅ Structured vs. Visual |
| Action titles (sentence, not label) | ✗ | ✅ Required on every slide |
| Ghost deck test | ✗ | ✅ Explicit QA step |
| Storyline frameworks (SCR, pyramid) | ✗ | ✅ |
| Exhibit discipline (one exhibit + so what) | ✗ | ✅ |
| Source citation rules | ✗ | ✅ |
| Grayscale + one accent formatting | ✗ | ✅ |
| Pre-submission checklist | ✗ | ✅ |

See [structured.md](structured.md) for full details.

---

## Attribution

Scripts and base skill files adapted from [anthropics/skills](https://github.com/anthropics/skills/tree/main/skills/pptx).
New content (`structured.md`, additions to `SKILL.md`) by this fork.
