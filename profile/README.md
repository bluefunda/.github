# BlueFunda

We build developer tools and AI agents for the SAP ecosystem.

---

## ABAPer

**ABAPer** is a platform for modern ABAP development — write, test, deploy, and manage ABAP code with AI assistance, from the browser, your editor, or the command line.

```
abaper-editor (Web IDE)  ──┐
abaper-vscode (VS Code)  ──┤──► abaper-gw (gateway + AI) ──► SAP ADT
abaper        (CLI/TUI)  ──┤
abaper-eclipse (Eclipse) ──┘
```

All clients share the same backend and AI capabilities.

### Clients

| Repository | What it is | Install |
|------------|-----------|---------|
| [abaper-editor](https://github.com/bluefunda/abaper-editor) | Browser-based ABAP IDE — Monaco editor, no install | [abaper.bluefunda.com](https://abaper.bluefunda.com) |
| [abaper-vscode](https://github.com/bluefunda/abaper-vscode) | AI-powered ABAP assistant for VS Code | [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=bluefunda.abaper) |
| [abaper](https://github.com/bluefunda/abaper) | CLI, TUI, and Go SDK — terminal workflows, scripting, CI/CD | `brew install --cask abaper` |
| [abaper-eclipse](https://github.com/bluefunda/abaper-eclipse) | Eclipse / ADT plugin _(coming soon)_ | — |

### Which client should I use?

| I want to… | Use |
|-----------|-----|
| Try ABAPer with zero setup | **Web IDE** — open [abaper.bluefunda.com](https://abaper.bluefunda.com) |
| Work inside VS Code | **abaper-vscode** extension |
| Use the terminal, write shell scripts, or run in CI/CD | **abaper CLI** |
| Build SAP tooling in Go | **abaper SDK** (`lib/` package) |
| Keep using Eclipse ADT | **abaper-eclipse** _(coming soon)_ |

### Feature comparison

| Feature | Web IDE | VS Code | CLI / TUI |
|---------|:-------:|:-------:|:---------:|
| AI pair-programmer chat | ✓ | ✓ | ✓ |
| Generate & deploy ABAP | ✓ | ✓ | ✓ |
| Syntax check / LSP | ✓ | ✓ | SDK |
| Run unit tests | ✓ | ✓ | ✓ |
| SAP system management | ✓ | ✓ | ✓ |
| No browser required | — | — | ✓ |
| CI/CD automation | — | — | ✓ |
| Embed in Go programs | — | — | ✓ |

### Getting started

```bash
# Web IDE
open https://abaper.bluefunda.com

# VS Code
code --install-extension bluefunda.abaper

# CLI (macOS)
brew tap bluefunda/tap && brew install --cask abaper
abaper login
abaper system add --host https://my-sap:44300 -u USER -p PASS
abaper   # opens interactive TUI
```

---

## BlueFunda AI

**BlueFunda AI** brings intelligent automation to SAP — context-aware agents that automate provisioning, generate OData services from plain-text prompts, answer data questions in natural language, and orchestrate S/4HANA across clouds. BlueFunda AI also powers the AI features in ABAPer.

### Getting started

- **Try it** — [bluefunda.com/login](https://bluefunda.com/login)
- **CLI** — `brew tap bluefunda/tap && brew install --cask bai`
- **Learn more** — [bluefunda.com/bluefunda-ai](https://bluefunda.com/bluefunda-ai/)

| Repository | Description |
|------------|-------------|
| [bluefunda-ai](https://github.com/bluefunda/bluefunda-ai) | CLI for BlueFunda AI — TUI, chat, agentic coding, MCP (`bai` binary) |
| [llmrouter](https://github.com/bluefunda/llmrouter) | LLM router for model selection and request routing |

---

## BlueRequests

**BlueRequests** is an event-driven change and release management platform for SAP — manage transport requests, change orders, approvals, and release workflows from the CLI or a TUI dashboard.

### Getting started

- **CLI** — `brew tap bluefunda/tap && brew install --cask breq`
- **Learn more** — [bluefunda.com](https://bluefunda.com)

| Repository | Description |
|------------|-------------|
| [bluerequests](https://github.com/bluefunda/bluerequests) | CLI for BlueRequests change and release management (`breq` binary) |

---

## Other Projects

| Repository | Description |
|------------|-------------|
| [btp-go](https://github.com/bluefunda/btp-go) | stdlib-only Go libraries for SAP BTP — XSUAA, Connectivity, Destination, service bindings |
| [abap-odata-test](https://github.com/bluefunda/abap-odata-test) | ABAP library for OData endpoint testing and assertions |
| [xpro-free](https://github.com/bluefunda/xpro-free) | XPro Free Edition — XML processing toolchain |
| [homebrew-tap](https://github.com/bluefunda/homebrew-tap) | Homebrew formulae for BlueFunda CLIs |
