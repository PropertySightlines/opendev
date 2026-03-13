# OpenDev — Project Context

## Project Overview

OpenDev is an open-source, terminal-native AI coding agent built as a **compound AI system** in Rust. Instead of relying on a single monolithic LLM, it orchestrates a structured ensemble of specialized agents and workflows — each independently bound to a user-configured model from any supported provider.

### Key Characteristics

- **Language:** Rust (workspace with 21 crates)
- **Frontend:** React + Vite + TypeScript (web-ui/)
- **License:** MIT
- **Architecture:** Modular crate structure with clear separation of concerns

### Core Concepts

- **Compound AI:** Multiple models collaborate, each optimized for its role (execution, thinking, critique, compaction, vision)
- **Per-Workflow Model Binding:** Each workflow slot can bind to any model from any provider
- **Concurrent Sessions:** Multiple independent agent sessions running in parallel
- **TUI + Web UI:** Terminal interface for power users, web interface for remote monitoring

## Building and Running

### Prerequisites

- Rust >= 1.94
- Node.js (for web-ui)
- API key for at least one LLM provider (OpenAI, Anthropic, Fireworks, etc.)

### Build Commands

```bash
# Build entire workspace
cargo build --workspace

# Build release binary (opendev-cli)
cargo build --release -p opendev-cli

# Install to ~/.local/bin/
cp target/release/opendev ~/.local/bin/

# Build web UI
cd web-ui && npm ci && npm run build
```

### Run Commands

```bash
# Interactive TUI
opendev

# Non-interactive prompt
opendev -p "explain this codebase"
opendev "do something"

# Web UI
opendev run ui

# Resume most recent session
opendev --continue

# List MCP servers
opendev mcp list
```

### Test Commands

```bash
# Run all tests
cargo test --workspace

# Test specific crate
cargo test -p opendev-tui
cargo test -p opendev-cli

# Run single test by name
cargo test -p opendev-tui test_render_thinking_expanded
```

### Lint and Format

```bash
# Type check
cargo check --workspace

# Lint (zero warnings required)
cargo clippy --workspace -- -D warnings

# Format
cargo fmt --all
```

### Development Workflow (Post-Change)

After every change, complete ALL steps:

1. **Unit & Integration Tests:** `cargo test --workspace`
2. **Lint & Type Checks:** `cargo clippy --workspace -- -D warnings` && `cargo check --workspace`
3. **Rebuild Release:** `cargo build --release -p opendev-cli`
4. **Real Simulation Test:** `echo "hello" | opendev -p "hello"` (verify end-to-end with real LLM)

## Architecture

### Crate Map (21 crates)

| Crate | Purpose |
|-------|---------|
| `opendev-cli` | Binary entry point (clap CLI, dispatches to TUI/REPL/subcommands) |
| `opendev-tui` | Terminal UI (ratatui + crossterm, async event loop) |
| `opendev-web` | Web backend (axum + WebSocket, broadcasts agent events) |
| `opendev-repl` | REPL loop, query enhancement (@file injection), message preparation |
| `opendev-agents` | ReAct loop, thinking/critique phases, prompt composition |
| `opendev-runtime` | Runtime services (approval, cost tracking, modes) |
| `opendev-config` | Hierarchical config loading (project > user > env > defaults) |
| `opendev-models` | Shared data types and models |
| `opendev-http` | HTTP client, auth rotation, provider adapters |
| `opendev-context` | Context engineering (compaction stages, message validation) |
| `opendev-history` | Session persistence (JSON per project, atomic writes) |
| `opendev-memory` | Memory systems (embeddings, reflection, playbook) |
| `opendev-tools-core` | Tool registry, BaseTool trait, dispatch |
| `opendev-tools-impl` | 30+ tool implementations (bash, edit, file ops, web, agents) |
| `opendev-tools-lsp` | LSP integration and language servers |
| `opendev-tools-symbol` | AST-based symbol navigation |
| `opendev-mcp` | Model Context Protocol integration |
| `opendev-channels` | Channel routing |
| `opendev-hooks` | Hook system |
| `opendev-plugins` | Plugin manager |
| `opendev-docker` | Docker runtime support |

### Key Dependencies

```toml
# Serialization
serde, serde_json, chrono, uuid, strum

# Async
tokio, tokio-util, async-trait, futures

# HTTP
reqwest, axum, tower, tower-http

# Logging
tracing, tracing-subscriber

# Error handling
thiserror, anyhow

# TUI
ratatui, crossterm

# CLI
clap
```

## Configuration

### Config Hierarchy (highest priority first)

1. **Project config:** `.opendev/settings.json` in project root
2. **Global config:** `~/.opendev/settings.json`
3. **Defaults:** Built-in default values

### Environment Variables

```bash
# API Keys (any one or multiple for multi-provider)
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export FIREWORKS_API_KEY="fw_..."
export GOOGLE_API_KEY="..."
export GROQ_API_KEY="..."
export MISTRAL_API_KEY="..."
export DEEPINFRA_API_KEY="..."
export OPENROUTER_API_KEY="..."
export AZURE_OPENAI_API_KEY="..."
```

### Workflow Model Slots

| Slot | Purpose | Fallback |
|------|---------|----------|
| Normal | Primary execution (coding, tool calls) | — |
| Thinking | Complex reasoning, planning | Normal |
| Compact | Context summarization | Normal |
| Critique | Self-critique of output | Thinking → Normal |
| VLM | Vision/image processing | Normal (if capable) |

### Example Config

```json
{
  "model_provider": "anthropic",
  "model": "claude-sonnet-4-20250514",
  "model_thinking_provider": "openai",
  "model_thinking": "o3",
  "model_compact_provider": "fireworks",
  "model_compact": "accounts/fireworks/models/kimi-k2-instruct-0905"
}
```

## Development Conventions

### Code Style

- Run `cargo fmt` before committing
- Run `cargo clippy` and fix all warnings
- Follow standard Rust naming conventions (snake_case functions, CamelCase types)
- Config files: `rustfmt.toml` sets `max_width = 100`

### Testing Practices

- Add unit tests for new logic
- Add/update integration tests in `tests/integration.rs` for cross-crate behavior
- **Critical:** Verify features work end-to-end in real TUI with real LLM responses

### Commit Style

- Clear, concise messages focused on "why" not "what"
- Review recent commits with `git log -n 3` for style matching

### File Organization

- Source files in `src/` directories within each crate
- Tests in `tests/` directories (integration) or `#[cfg(test)]` modules (unit)
- Templates in `templates/` directories where applicable

## Key Files

| File | Purpose |
|------|---------|
| `Cargo.toml` | Workspace root, defines all 21 member crates |
| `README.md` | User-facing documentation |
| `CLAUDE.md` | Development workflow and commands |
| `ROADMAP.md` | Feature roadmap (shipped, in-progress, planned) |
| `docs/providers.md` | Provider setup and configuration guide |
| `web-ui/package.json` | Frontend dependencies and scripts |

## Web UI

The frontend is a React + Vite + TypeScript app:

```bash
cd web-ui
npm ci              # Install dependencies
npm run dev         # Development server
npm run build       # Production build (outputs to dist/)
npm run lint        # ESLint
```

**Dependencies:** React, React Router, Zustand (state), TailwindCSS, React Markdown, Lucide icons

## MCP Integration

Model Context Protocol for dynamic tool discovery:

```bash
opendev mcp list                    # List servers
opendev mcp add myserver uvx mcp-server-sqlite
opendev mcp enable myserver
opendev mcp disable myserver
opendev mcp remove myserver
```

## Troubleshooting

- **Binary not found:** Ensure `~/.local/bin/` is in PATH
- **Config issues:** Run `opendev config show` to inspect current config
- **First run:** If no config exists, OpenDev prompts for setup automatically
- **TUI rendering issues:** Check terminal supports ratatui; try different terminal emulator
