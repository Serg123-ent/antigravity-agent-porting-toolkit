# Antigravity Agent Porting Toolkit & Integration Protocol

[English documentation](README_EN.md) | [Русская документация](README.md)

A comprehensive protocol, architectural specification, and production-ready plugin template for porting and seamlessly integrating AI agents from various Agent Harnesses (Claude Code, OpenAI Codex, Cursor, Qoder, OpenCode, Windsurf, Devin) into the **Google Antigravity** ecosystem (Antigravity CLI `agy` and Antigravity IDE).

---

## Table of Contents

1. [Overview](#overview)
2. [The Challenge of Importing Agents from Codex and Claude into Antigravity](#the-challenge-of-importing-agents-from-codex-and-claude-into-antigravity)
3. [How the Antigravity Agent Porting & Integration Protocol Solves These Challenges](#how-the-antigravity-agent-porting--integration-protocol-solves-these-challenges)
4. [Quick Start in Antigravity](#quick-start-in-antigravity)
5. [Repository Structure](#repository-structure)
6. [License](#license)

---

## Overview

Modern AI agents are built across diverse agent harnesses with incompatible packaging and lifecycle primitives:
* **Claude Code**: Plugin marketplace, background shell hooks (`SessionStart`, `UserPromptSubmit`, `SubagentStart`) executing Node.js scripts, and proprietary environment variables (`${CLAUDE_PLUGIN_ROOT}`).
* **OpenAI Codex**: Lifecycle hooks, execution trust models (`/hooks`).
* **Google Antigravity**: Secure, deterministic, tool-centric architecture driven by strict directory layouts (`skills/`, `rules/`, `plugin.json`), native agent orchestration (`invoke_subagent`, `ask_question`), and the Model Context Protocol (MCP).

This repository provides a standardized protocol and clean plugin template to make agent migration frictionless.

---

## The Challenge of Importing Agents from Codex and Claude into Antigravity

When attempting to import plugins or agent configurations directly from Claude Code or Codex into Antigravity, developers encounter key structural incompatibilities:

### 1. Incompatible Lifecycle Shell Hooks
* **In Claude Code / Codex**: Plugins execute arbitrary shell scripts (e.g. running Node.js scripts on every user prompt submit or session start to inject instructions dynamically).
* **In Antigravity**: Background shell interceptors do not exist by design to maintain determinism and security. Copying `claude-codex-hooks.json` or `cursor-hooks.json` directly causes either spam of `node: command not found` or is ignored.

### 2. Proprietary Path & Environment Variable Assumptions
* Plugins from Claude/Codex assume environment variables such as `${CLAUDE_PLUGIN_ROOT}`, `$CODEBUDDY_CONFIG_DIR`, or persist state into unportable host paths (`~/.claude/.ponytail-active`).
* Antigravity does not set these variables, leading to script crashes.

### 3. Divergent `plugin.json` Manifest Schemas
* Claude/Codex manifests often include unstructured fields (`skills`, `rules`, `hooks`, `author`, `homepage`).
* Antigravity strictly validates `plugin.json`. Unknown fields are dropped, and invalid schemas can prevent plugin indexing.

### 4. Subagent Context Injection Model
* Claude Code passes context to subagents via background hook JSON outputs.
* Antigravity requires explicit, native delegation through `invoke_subagent` or declarative agent system prompts.

---

## How the Antigravity Agent Porting & Integration Protocol Solves These Challenges

The [ANTIGRAVITY_AGENT_PORTING_PROTOCOL.md](ANTIGRAVITY_AGENT_PORTING_PROTOCOL.md) outlines a deterministic 5-step methodology:

1. **Source Audit**: Uncovers hidden runtime dependencies, proprietary shell calls, and hardcoded state files.
2. **Hook Emulation Patterns**:
   * `SessionStart` $\rightarrow$ Mapped to `Prerequisites` in `AGENTS.md` or dedicated initialization skills.
   * `UserPromptSubmit` (e.g. mode toggling `lite / full / ultra`) $\rightarrow$ Mapped to interactive question tools (`ask_question`), command parameters, or local project configs (`.antigravity/settings.json`).
   * `SubagentStart` $\rightarrow$ Mapped to explicit prompt inheritance via `invoke_subagent`.
   * `PreToolUse` / `PostToolUse` $\rightarrow$ Encapsulated as deterministic `Critical Rules` in `AGENTS.md`.
3. **Clean Rule Migration**: Removes vendor-specific XML tags and ensures `rules/AGENTS.md` has no frontmatter.
4. **Tool Translation**: Replaces proprietary file and command calls with standard Antigravity primitives.
5. **Manifest & Namespace Validation**: Formats clean manifests and prevents MCP naming collisions.

---

## Quick Start in Antigravity

### Option 1. Global Plugin Installation

1. Copy the `plugin` folder into your Antigravity plugins directory:
   * **Windows**:
     ```powershell
     Copy-Item -Recurse -Force plugin "$HOME\.gemini\config\plugins\antigravity-porting-toolkit"
     ```
   * **Linux / macOS**:
     ```bash
     cp -r plugin ~/.gemini/config/plugins/antigravity-porting-toolkit
     ```

2. Restart or reload Antigravity (Antigravity IDE or `agy` CLI).

3. Trigger the skill:
   ```text
   /agent-port
   ```

---

### Option 2. Project-Level (Workspace) Installation

1. Copy `plugin/rules/AGENTS.md` into your project root as `AGENTS.md`.
2. Copy `plugin/skills/` into your project's `.agents/skills/` or `.gemini/skills/`.
3. The skills and rules will automatically be active for your current workspace.

---

## Repository Structure

```text
├── .gitignore                               # Standard ignore file
├── LICENSE                                  # MIT License
├── README.md                                # Russian documentation (Primary)
├── README_EN.md                             # English documentation
├── ANTIGRAVITY_AGENT_PORTING_PROTOCOL.md     # Full architectural porting protocol
└── plugin/                                  # Antigravity plugin template
    ├── plugin.json                          # Strict plugin manifest
    ├── rules/
    │   └── AGENTS.md                        # Always-on rules (no frontmatter)
    └── skills/
        └── agent-port/
            └── SKILL.md                     # /agent-port skill definition
```

---

## License

Released under the [MIT License](LICENSE).
