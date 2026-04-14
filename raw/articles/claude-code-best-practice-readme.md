<!-- source_url: https://github.com/shanraisshan/claude-code-best-practice -->
# GitHub - shanraisshan/claude-code-best-practice

Repository for mastering Claude Code (Anthropic's agentic CLI).

## Key Concepts
- **Subagents**: Autonomous actors in fresh isolated context.
- **Commands**: Knowledge injected into existing context (slash commands).
- **Skills**: Configurable, auto-discoverable knowledge folders.
- **Hooks**: Lifecycle event handlers (scripts/prompts).
- **MCP Servers**: Model Context Protocol for external tools.

## Core Workflows
Research -> Plan -> Execute -> Review -> Ship.

## Critical Tips
- Keep CLAUDE.md < 200 lines.
- Use skills for progressive disclosure.
- Manual /compact at 50% context.
- Start with Plan Mode.
- Use subagents to offload tasks and keep main context clean.
