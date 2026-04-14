<!-- source_url: https://skillsplayground.com/guides/claude-code-plugins/ -->
Claude Code Plugins: The Complete Guide to Skills, MCP Servers & Extensions
Claude Code Plugins: The Complete Guide to Skills, MCP Servers & Extensions
Updated February 2026 · 15 min read
Claude Code plugins are modular extensions that give Claude Code new capabilities. Whether you need Claude to follow your team's code review standards, connect to a database, auto-format files on save, or run a deployment checklist -- there is a plugin for that. And if there is not, you can create one in minutes.
This guide covers everything about
Claude Code plugins
: what they are, how to find them, how to install them, and how to build your own. If you are new to Claude Code, start with our
Claude Code tutorial
first, then come back here to supercharge your setup with plugins.
Looking for plugins to install right now?
Browse the
Skills Directory
for ready-to-use skills, or check the
MCP Servers Directory
for tool integrations. You can
test any skill
before installing it.
What Are Claude Code Plugins?
A Claude Code plugin is any extension that modifies or enhances Claude Code's behavior. The term "plugin" is an umbrella that covers four distinct component types, each serving a different purpose:
Skills
Markdown system prompts that give Claude specialized expertise. The most common type of Claude Code plugin.
MCP Servers
External tool integrations via the Model Context Protocol. Connect Claude to APIs, databases, and services.
Hooks
Event-driven scripts that fire before or after Claude takes actions. Enforce rules and automate workflows.
Slash Commands
Custom commands invoked with
/
. Create shortcuts for multi-step workflows.
You might hear people say "Claude Code extensions," "Claude Code skills," or "Claude Code plugins" interchangeably. They all refer to the same ecosystem. The official terminology from Anthropic uses "skills" for system prompt files and "MCP servers" for tool integrations, but the community broadly uses "plugins" to describe any add-on.
Types of Claude Code Plugins
Understanding the four plugin types helps you pick the right tool for every situation. Here is a breakdown of each type, when to use it, and how it works.
1. Skills (System Prompts)
Skills
are Markdown files that contain system prompt instructions. When activated, they are injected into Claude's context and shape how it behaves for the duration of a conversation. Skills are the simplest and most popular type of Claude Code plugin.
Skills live in your project's
.claude/skills/
directory. Each skill is a
.md
file with instructions that tell Claude what to do and how to do it:
# .claude/skills/code-reviewer.md

You are a senior code reviewer. When reviewing code:

1. Check for security vulnerabilities first
2. Look for performance bottlenecks
3. Verify error handling is comprehensive
4. Ensure naming conventions are consistent
5. Flag any missing tests

Be direct. If code is good, say so briefly. If it needs
work, explain exactly what to change and why.
Skills are ideal when you want Claude to adopt a specific persona, follow specific guidelines, or have domain expertise. Browse hundreds of community-created skills in our
Skills Directory
, organized by
category
.
Skills vs CLAUDE.md:
Skills are modular and activatable -- you choose when to load them. CLAUDE.md is always loaded. Put universal project rules in CLAUDE.md and specialized behaviors in skills. Learn more in our
best practices guide
.
2. MCP Servers (Tool Integrations)
MCP servers
give Claude entirely new tools. While skills change how Claude thinks, MCP servers change what Claude can
do
. They run as external processes that communicate with Claude Code via the
Model Context Protocol
.
Common MCP server use cases:
Database access
-- Query PostgreSQL, SQLite, or MongoDB directly
API integrations
-- Post to Slack, create GitHub issues, manage deployments
File system tools
-- Read from remote filesystems, cloud storage, or containers
Browser automation
-- Navigate web pages, scrape content, fill forms
Search
-- Web search, semantic code search, documentation lookup
MCP servers are configured in your project's
.claude/settings.json
or globally in
~/.claude/settings.json
:
{
  "mcpServers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://localhost:5432/mydb"
      }
    },
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"]
    }
  }
}
Explore available MCP servers in our
MCP Servers Directory
, which catalogs servers across dozens of categories. For a deep dive, read the
MCP Servers Guide
.
3. Hooks (Automation)
Hooks
are event-driven scripts that fire at specific points during Claude Code's execution. They are the enforcement layer of the plugin system -- while skills are advisory, hooks are guaranteed to run.
Hook events include:
PreToolUse
-- Runs before Claude uses a tool (block or modify actions)
PostToolUse
-- Runs after a tool completes (format files, validate output)
Stop
-- Runs when Claude finishes a turn (run tests, check quality)
Notification
-- Runs when Claude sends a notification
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hook": "npx prettier --write $CLAUDE_FILE_PATH 2>/dev/null; exit 0"
      }
    ],
    "Stop": [
      {
        "hook": "npm test -- --bail 2>&1 | tail -20"
      }
    ]
  }
}
The most impactful hooks are auto-formatters (PostToolUse on Write/Edit), test runners (Stop), and file guards (PreToolUse to block edits to critical files). See the full
Hooks Guide
for event reference and examples.
4. Slash Commands (Workflow Shortcuts)
Slash commands
are custom commands you trigger by typing
/
in Claude Code. They combine a prompt template with optional instructions, letting you turn complex multi-step workflows into a single command.
Slash commands live as
.md
files in
.claude/commands/
:
# .claude/commands/deploy.md

Review the current branch for production readiness:

1. Run the full test suite
2. Check for console.log statements in production code
3. Verify environment variables are not hardcoded
4. Confirm database migrations are reversible
5. Summarize any concerns before deploying
Now typing
/deploy
in Claude Code runs this entire workflow. Combine slash commands with skills for even more powerful automation.
How to Install Claude Code Plugins
There are several ways to add Claude Code plugins to your workflow, depending on the plugin type and source.
Method 1: The /install Command
The simplest way to install a plugin from GitHub. In Claude Code, type:
/install-plugin owner/repository
This clones the repository and registers all components declared in its
plugin.json
manifest. For plugins with multiple skills, you can install a specific one:
/install-plugin owner/repository --skill skill-name
Method 2: npx playbooks add
The
playbooks
CLI tool provides another way to install community skills:
npx playbooks add code-reviewer
npx playbooks add frontend-design
npx playbooks add tdd-workflow
This downloads and installs the skill files directly into your project's
.claude/skills/
directory.
Method 3: Manual Installation
For maximum control, you can manually install any plugin component:
For skills:
Copy the
.md
file into
.claude/skills/
mkdir -p .claude/skills
curl -o .claude/skills/code-reviewer.md https://raw.githubusercontent.com/owner/repo/main/skill.md
For MCP servers:
Add the server configuration to
.claude/settings.json
For hooks:
Add hook definitions to
.claude/settings.json
under the
hooks
key
For slash commands:
Copy the
.md
file into
.claude/commands/
Method 4: Import from the Skills Playground
Browse the
Skills Directory
on Skills Playground. Each skill page has a copy-paste install command. You can also
test skills interactively
in the playground before committing to an install.
After installing any plugin,
restart Claude Code or start a new conversation for the changes to take effect. Skills load at session start, and MCP servers initialize on connection.
Top 10 Most Popular Claude Code Plugins
Based on community usage and downloads, these are the most popular Claude Code plugins and skills. All are available in our
Skills Directory
.
Code Reviewer
Turns Claude into a senior code reviewer that checks security, performance, error handling, and style. The most-installed skill in the directory.
Frontend Design System
Gives Claude expertise in React, Tailwind, accessibility, and responsive design patterns. Ideal for frontend-heavy projects.
TDD Workflow
Enforces test-driven development: write tests first, implement to pass, then refactor. Works with Jest, Vitest, pytest, and Go testing.
API Designer
Guides Claude through RESTful API design following OpenAPI spec, consistent error formats, and proper HTTP status codes.
Git Workflow
Teaches Claude your team's git conventions: branch naming, commit message format, PR templates, and merge strategies.
Security Auditor
Scans code for OWASP Top 10 vulnerabilities, insecure dependencies, secrets in code, and injection risks.
Documentation Writer
Creates comprehensive documentation: README files, API docs, inline comments, and architecture decision records.
DevOps Engineer
Expertise in Docker, CI/CD pipelines, infrastructure-as-code, and deployment automation.
Performance Optimizer
Identifies bottlenecks in database queries, rendering, bundle size, and algorithm complexity.
Refactoring Expert
Systematically improves code structure: extracts functions, reduces duplication, applies design patterns, and improves naming.
This is just a small sample. Browse the full
Skills Directory
for hundreds more, filterable by
category
-- from
backend development
to
data science
to
mobile development
.
The MCP Server Ecosystem
The
MCP server ecosystem
is one of the most powerful aspects of Claude Code plugins. MCP (Model Context Protocol) is an open standard that lets AI models connect to external tools and data sources through a unified interface.
The ecosystem has grown rapidly, with servers available for nearly every major developer tool and service:
Category
Example MCP Servers
What They Enable
Databases
PostgreSQL, SQLite, MongoDB
Direct database queries and schema inspection
Version Control
GitHub, GitLab, Bitbucket
PR management, issue tracking, code review
Cloud Platforms
AWS, Cloudflare, Vercel, Render
Deploy, configure, and manage infrastructure
Communication
Slack, Discord, Email
Send messages, read channels, manage notifications
Browsers
Puppeteer, Playwright
Web scraping, testing, form automation
Search
Brave Search, Exa, web fetch
Web search, semantic code search, documentation lookup
Explore the full catalog in our
MCP Servers Directory
, which includes installation commands, configuration examples, and compatibility notes for each server. For a comprehensive overview of the protocol itself, see the
MCP Servers Guide
.
How to Create Your Own Plugin
Creating a Claude Code plugin is straightforward. The simplest plugin is a single Markdown file; the most complex can bundle skills, hooks, MCP servers, and commands into a shareable package.
Quick Start: Create a Skill in 60 Seconds
The fastest way to create a Claude Code plugin:
# Create the skills directory
mkdir -p .claude/skills

# Create your skill file
cat > .claude/skills/my-skill.md
<
<
'EOF'
# My Custom Skill

You are an expert at [your domain]. When the user asks you to
work on [specific tasks], follow these guidelines:

1. [First principle or step]
2. [Second principle or step]
3. [Third principle or step]

## Important Rules
- [Rule 1]
- [Rule 2]
- [Rule 3]
EOF
That is it. Restart Claude Code, and your skill is active. Test it in the
Skills Playground
to refine the prompt before using it in a real project.
Full Plugin: Create a plugin.json Package
For plugins with multiple components that you want to share via GitHub, create a
plugin.json
manifest at the root of your repository:
{
  "name": "my-team-plugin",
  "version": "1.0.0",
  "description": "Our team's Claude Code standards and tools",
  "author": "your-github-username",
  "skills": [
    {
      "name": "code-review",
      "description": "Review code against our team standards",
      "path": "skills/code-review.md"
    },
    {
      "name": "api-design",
      "description": "Design APIs following our conventions",
      "path": "skills/api-design.md"
    }
  ],
  "hooks": [
    {
      "event": "PostToolUse",
      "matcher": "Write|Edit",
      "path": "hooks/auto-format.sh"
    }
  ],
  "commands": [
    {
      "name": "review",
      "description": "Run full code review",
      "path": "commands/review.md"
    }
  ],
  "mcpServers": {
    "our-internal-api": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/mcp/api-server.js"]
    }
  }
}
Writing Effective Skill Prompts
The quality of a skill depends entirely on the quality of its system prompt. Here are the principles that produce the best results:
Be specific about the role.
"You are a senior backend engineer specializing in PostgreSQL performance" is better than "You are a database expert."
Include concrete examples.
Show Claude what good output looks like for your use case.
Set explicit constraints.
"Always use parameterized queries, never string concatenation" is enforceable; "be careful with SQL" is not.
Keep it under 500 lines.
Longer skills consume more context, leaving less room for actual work. If you need more, split into multiple focused skills.
Test iteratively.
Use the
Skills Playground
to test and refine your prompt before deploying.
For a deeper dive into prompt engineering for skills, see our
best practices guide
and the
system prompt guide
.
Publishing Your Plugin
Push your plugin to GitHub and share the install command:
# Push to GitHub
git init
git add plugin.json skills/ hooks/ commands/
git commit -m "Initial plugin release"
gh repo create my-team-plugin --public --push

# Others install with:
/install-plugin your-username/my-team-plugin
Consider submitting your plugin to the
Skills Playground Directory
so the community can discover and use it.
Plugin vs Extension vs Skill: Terminology Guide
The Claude Code ecosystem uses several terms that can be confusing. Here is a definitive guide to what each one means:
Term
Definition
Example
Plugin
Umbrella term for any Claude Code extension. Can contain skills, hooks, MCP servers, and commands.
A GitHub repo with plugin.json
Skill
A Markdown file with system prompt instructions that shape Claude's behavior.
.claude/skills/reviewer.md
MCP Server
An external process giving Claude new tools via the Model Context Protocol.
PostgreSQL MCP server
Hook
A script that runs automatically before or after Claude performs an action.
Auto-format on file save
Slash Command
A custom command invoked with
/
that triggers a predefined prompt template.
/deploy
,
/review
Extension
Informal synonym for plugin, commonly used in the community.
Same as plugin
Playbook
Community term for a curated collection of skills and commands for a specific workflow.
A "React playbook" with 5 skills
In practice, "Claude Code plugins," "Claude Code extensions," and "Claude Code skills" all point to the same ecosystem. The important distinction is between
skills
(prompt-based, change behavior) and
MCP servers
(process-based, add tools). For a detailed comparison, read our
Skills vs MCP Servers guide
.
Where to Find Claude Code Plugins
The Claude Code plugin ecosystem is distributed across several sources. Here is where to look:
Skills Playground (This Site)
The
Skills Playground directory
is the largest curated collection of Claude Code skills, organized by
category
. You can:
Browse all skills
-- filterable, searchable, with install commands
Browse MCP servers
-- cataloged with configuration examples
Test skills interactively
-- try before you install
Browse by category
-- find plugins for your specific domain
GitHub
Search GitHub for
claude-code-skills
,
claude-code-plugin
, or
claude-code-playbook
to find community repositories. Many developers publish their personal skill collections as open-source repos.
MCP Server Registries
For MCP servers specifically, check the
MCP Servers Directory
on this site and the official Model Context Protocol repositories on GitHub. New servers are published regularly as the ecosystem grows.
Community Forums
The Anthropic Discord and Claude Code GitHub discussions are active sources for plugin recommendations and new releases.
Pro tip:
Combine multiple plugins for maximum impact. A typical productive setup includes 2-3 skills for your core workflow, 1-2 MCP servers for tool access, and a handful of hooks for enforcement. Start with the
Skills Directory
and build from there.
Advanced Plugin Patterns
Combining Skills with Hooks
The most powerful plugin setups combine advisory skills with enforcing hooks. For example, a "Clean Code" plugin might include:
A
skill
that teaches Claude your code style preferences
A
PostToolUse hook
that auto-formats every file Claude writes
A
Stop hook
that runs your linter and feeds violations back to Claude
A
slash command
(
/clean
) that triggers a full codebase cleanup
Per-Project Plugin Configuration
Plugins can store per-project settings using local configuration files. This lets one plugin behave differently across projects:
# .claude/plugin-settings.local.md
---
framework: react
testRunner: vitest
formatter: prettier
strictMode: true
---

Additional project-specific notes for the plugin to consider.
Using the Claude Code SDK with Plugins
For programmatic control, the
Claude Code SDK
lets you build custom integrations that leverage plugins. You can invoke skills, trigger commands, and manage MCP server connections through the SDK API -- useful for CI/CD pipelines and automated workflows.
Troubleshooting Plugins
Plugin not loading after install
Restart Claude Code or start a new session -- plugins load at session start
Verify your
plugin.json
is valid JSON (no trailing commas, correct syntax)
Check that all file paths referenced in the manifest actually exist
Inspect
~/.claude/plugins/
to confirm the plugin was installed
Skill not activating
Confirm the
.md
file is in
.claude/skills/
(not a subdirectory)
Check that the skill file is not empty and contains valid Markdown
If using plugin.json, ensure the skill has both
name
and
description
MCP server not connecting
Use
${CLAUDE_PLUGIN_ROOT}
for file paths in plugin.json, not relative paths
Ensure dependencies are installed (
npm install
for Node.js servers)
Check that the MCP server binary is executable
Review the
MCP Servers Guide
for transport-specific issues
Hooks not firing
Verify the event name is exact:
PreToolUse
,
PostToolUse
,
Stop
,
Notification
Make shell scripts executable:
chmod +x hooks/my-hook.sh
Test the hook script in isolation before adding it to the plugin
See the
Hooks Guide
for detailed debugging steps
Next Steps
You now have a comprehensive understanding of the Claude Code plugin ecosystem. Here is where to go from here:
Browse the Skills Directory
-- find and install skills for your workflow
Explore MCP Servers
-- connect Claude to your tools and services
Test in the Playground
-- try skills interactively before installing
Read the Best Practices Guide
-- optimize your entire Claude Code setup
Learn about Hooks
-- add automated enforcement to your workflow
Explore the SDK
-- build custom integrations programmatically
Master the CLI
-- learn every command, flag, and shortcut