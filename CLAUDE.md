# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository is the **AI Coding Agent Skills** package for the Shioaji trading API (by Sinopac/永豐金證券). It is **not** the Shioaji library source code — the library is installed via `pip install shioaji`. This repo provides structured documentation that AI coding agents (Claude Code, Cursor, Copilot, Codex, etc.) use to understand and assist with Shioaji API development.

- **Official docs**: https://sinotrade.github.io/
- **LLM reference**: https://sinotrade.github.io/llms-full.txt
- **Current version**: stored in `_version` file (update this + `plugin.json` + `marketplace.json` + Dockerfiles when releasing)

## Architecture

```
.claude-plugin/          # Claude plugin metadata
  plugin.json            # Plugin definition (name, version, skills path)
  marketplace.json       # Marketplace listing metadata
skills/shioaji/          # AI agent skill documentation (the core content)
  SKILL.md               # Main entry point — quick start, constants, patterns
  PREPARE.md             # Account setup, API keys, simulation mode
  CONTRACTS.md           # Stock/futures/options contract lookup
  ORDERS.md              # Place, modify, cancel, combo orders
  RESERVE.md             # Reserve orders for disposition stocks
  STREAMING.md           # Real-time tick & bidask subscriptions
  MARKET_DATA.md         # Historical data, snapshots, scanners
  ACCOUNTING.md          # Balance, margin, P&L, trading limits
  WATCHLIST.md           # Custom stock list management
  ADVANCED.md            # Quote binding, non-blocking, stop orders
  TROUBLESHOOTING.md     # Common issues and solutions
Dockerfile               # python:3.10-slim + shioaji
Dockerfile-jupyter       # Jupyter Lab environment with tutorial notebook
```

## Versioning & Release

Four files must stay in sync when updating the version:
1. `_version`
2. `.claude-plugin/plugin.json` → `"version"` field
3. `.claude-plugin/marketplace.json` → `"version"` field in plugins array
4. `Dockerfile` and `Dockerfile-jupyter` → `pip install shioaji==X.Y.Z`

## CI/CD

- `.github/workflows/docker.yml` — triggers on git tags, builds and pushes multi-arch (amd64/arm64) Docker images to `sinotrade/shioaji:latest` and `sinotrade/shioaji:jupyter` on DockerHub.

## Plugin Installation Commands

```bash
# Claude Code
claude plugin marketplace add Sinotrade/Shioaji
claude plugin install shioaji

# Universal (Cursor, Windsurf, Copilot, Codex)
npx skills add Sinotrade/Shioaji
```

## Content Guidelines

- All skill docs are bilingual (English + 繁體中文) with code examples
- `SKILL.md` is the entry point — it provides navigation to all other files
- Each skill file is self-contained for its topic with full code examples
- Constants should use `sj.constant.*` enum style (e.g., `sj.constant.Action.Buy`)
- Rate limits: Quote 50 req/5s, Accounting 25 req/5s, max 5 connections per person ID
