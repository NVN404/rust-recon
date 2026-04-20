# 🛡️ rust-recon 

**The AI-powered Skill for mathematically verified protocol reconnaissance reports.**

`rust-recon` is a modular AI Skill that orchestrates the [rust-recon-tool](https://github.com/NVN404/rust-recon-tool) to generate hallucination-free recon reports with 9 structured sections covering architecture, trust model, CPI flows, and recon signals.

🤖 **Works with:** Claude, Copilot, Cursor, Codex, and any AI agent supporting custom skills/commands.

---

##  What This Skill Does

When you invoke `/recon` in your AI agent (after installing this skill):

1. **Automatically installs the `rust-recon` tool** (if not already on your machine)
2. **Runs the deterministic AST extraction** against your Solana Anchor project
3. **Reads all extracted JSON facts** (`scope.json`, `facts.json`, `summary.json`)
4. **Generates a comprehensive recon report** with 9 sections:
   - Protocol Overview
   - Instruction Surface
   - Account & PDA Catalogue
   - Authority & Trust Model
   - Token & CPI Flows
   - Error Code Registry
   - Recon Signals Summary (section 7)
   - Manual Verification Checklist
   - Recon Metadata

---

## 📦 Installation

### Prerequisites
- Any AI agent supporting custom skills (Claude, Copilot, Cursor, Codex, etc.)
- Solana Rust toolchain installed locally

### Step 1: Clone This Skill Repo

```bash
git clone https://github.com/NVN404/rust-recon ~/.rust-recon-skill
```

### Step 2: Link the Skill to Your AI Agent

**For Claude Desktop (Canonical Setup):**
1. Open Claude Desktop preferences/settings
2. Navigate to "Custom Commands" or "Skills"
3. Add the path: `~/.rust-recon-skill`
4. Restart Claude

**For Other Agents:**
- **Claude Web:** Navigate to https://claude.ai → Custom Commands/Skills → Upload `https://github.com/NVN404/rust-recon`
- **Copilot:** Reference skill repo in settings → Point to `~/.rust-recon-skill`
- **Cursor:** Access command/skill config → Link to `~/.rust-recon-skill`
- **Others:** Follow your agent's custom command docs → Point to `~/.rust-recon-skill` or `https://github.com/NVN404/rust-recon`

> **Note:** All agents should load `.claude/commands/recon.md` as the orchestrator script

---

## Universal Recon Command Pattern

**Regardless of which AI agent you use**, the command always follows this simple pattern:

```
[agent-prefix] recon [format]
```

Where:
- **[agent-prefix]**: `/` (Claude), `@` (Copilot/Cursor), or natural language (others)
- **recon**: Always the same command
- **[format]**: Optional — `condensed`, `detailed`, or omit for standard

### Universal Pattern Examples

**Generate standard recon report:**
```
/recon              (Claude)
@rust-recon         (Copilot/Cursor)
```

**Generate condensed report:**
```
/recon condensed           (Claude)
@rust-recon condensed      (Copilot/Cursor)
```

**Generate detailed report:**
```
/recon detailed           (Claude)
@rust-recon detailed      (Copilot/Cursor)
```

---

##  Architecture

### `.claude/commands/recon.md`
The main orchestrator script. It:
- Checks prerequisites and validates you're in an Anchor workspace
- Downloads `rust-recon` to `~/.rust-recon_tool` (global)
- Compiles the tool via `cargo install`
- Reads all modular skill files below
- Enforces strict section schema and quality gates before returning output
- Generates the final report

### `skill/core.md`
**Hard Rules & Constraints:**
- 9-section structure definition
- Source priority and no-conflict precedence
- Exact table schemas for Section 2
- ASCII-only diagrams (no Mermaid)
- Recon-first output rules (no risk/severity columns in Section 2)

### `skill/references/facts-schema.md`
**Data Dictionary:**
- Explains every field in `facts.json`
- `wrapper_type` definitions
- `body_checks` interpretation
- `flags[]` as parser signal metadata for Section 7

### `skill/references/section-specs.md`
**What Each Section Must Contain:**
- Section 1–9 exact schema and ordering contract
- Required table headers and rendering rules
- Detailed vs. condensed mode differences

### `skill/references/audit-patterns.md`
**Mechanical Recon Analysis:**
- All section 2f (instruction-level) recon checks
- Section 3a PDA tagging rules
- Re-initialization safety analysis
- Unchecked-account verification patterns

### `skill/references/cpi-rules.md`
**Cross-Program Invocation Rules:**
- Implicit CPI naming conventions
- Token-2022 detection
- Extraction limitation warnings
- CPI flow visualization and manual verification prompts

### `skill/examples/`
**Reference Reports:**
- `stakeflow-recon.md` — Real Stake-Flow protocol recon (detailed format)
- `zenon-recon.md` — Real Zenon protocol recon (condensed format)

Use these to understand expected output quality and structure.

---

##  How It Works with the Tool

```
┌─────────────────────────────────────────┐
│  You type: /recon                       │
│  (in your AI agent, in your Anchor dir) │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│  Skill reads: .claude/commands/recon.md │
│  (this repo: rust-recon)                │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│  Orchestrator installs rust-recon-tool  │
│  to ~/.rust-recon_tool                  │
│  (from github.com/NVN404/rust-recon...) │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│  Tool runs:                             │
│  • rust-recon scope                     │
│  • rust-recon facts                     │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│  Skill reads generated JSON + refs      │
│  • .rust-recon/scope.json               │
│  • .rust-recon/facts.json               │
│  • skill/references/*.md (all rules)    │
└────────────┬────────────────────────────┘
             │
             ▼
┌─────────────────────────────────────────┐
│  Your AI agent generates a 9-section    │
│  recon.md at your project root           │
└─────────────────────────────────────────┘
```

---

##  Configuration

No environment variables or API keys needed! Everything is local and works with any AI agent.

If you want to use the tool **without this skill**, see the [rust-recon-tool README](https://github.com/NVN404/rust-recon-tool).

---

##  Example Invocations

**In your AI agent, inside your Anchor workspace:**

```
/recon
→ Generates default recon report

/recon condensed
→ Generates high-level summary

/recon detailed
→ Generates comprehensive breakdown

/recon show examples
→ Displays reference reports from skill/examples/
```

---

## 🤝 Contributing

Found a bug or want to enhance the skill? Contributions welcome!

1. Fork this repository
2. Create a feature branch (`git checkout -b feature/improve-section-specs`)
3. Make your changes (e.g., refining `skill/references/section-specs.md`)
4. Commit (`git commit -m 'Improve section 4 trust model rules'`)
5. Push (`git push origin feature/improve-section-specs`)
6. Open a Pull Request

---

## License

This skill is part of the `rust-recon` ecosystem and is licensed under the **MIT License**. 

---

##  Links

- **Tool Repository:** [github.com/NVN404/rust-recon-tool](https://github.com/NVN404/rust-recon-tool)
- **Skill Repository:** [github.com/NVN404/rust-recon](https://github.com/NVN404/rust-recon)
- **Issues & Discussions:** Use the tool or skill repo issues page
