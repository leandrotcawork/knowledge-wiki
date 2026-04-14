<!-- source_url: https://skillsplayground.com/guides/claude-code-best-practices/ -->
Claude Code Best Practices: CLAUDE.md, Skills, Hooks, and Workflows
Claude Code Best Practices
Updated February 2026 · 12 min read
Claude Code is most effective when you invest a little time setting up your project context. The difference between a default Claude Code installation and a well-configured one is dramatic -- like the difference between hiring a capable engineer and giving them no onboarding vs. a thorough one.
This guide covers the practices that make the biggest impact, from essential CLAUDE.md setup to advanced workflow patterns.
1. Write a Good CLAUDE.md
The
CLAUDE.md
file is the single most important configuration in your project. It sits at the root of your repository (or in
~/.claude/CLAUDE.md
for global settings) and is loaded into Claude's context at the start of every conversation.
What to include
Project architecture overview
-- a brief description of the stack, key directories, and how things connect. Claude can explore your codebase, but knowing where to look saves time.
Build and test commands
--
npm test
,
cargo build
, whatever your project uses. Be specific:
npm run test:unit
is better than "run tests."
Code conventions
-- naming patterns, import ordering, error handling patterns. Be explicit about things a new team member would need to know.
Things to avoid
-- known pitfalls, deprecated APIs, patterns you've moved away from.
# CLAUDE.md

## Project
React + TypeScript frontend, Express API backend.
Monorepo managed with Turborepo.

## Key directories
- apps/web - React frontend (Vite)
- apps/api - Express API (Node 20)
- packages/shared - Shared types and utilities

## Commands
- `turbo run build` - Build all packages
- `turbo run test` - Run all tests
- `npm run dev -w apps/web` - Start frontend dev server
- `npm run dev -w apps/api` - Start API dev server

## Conventions
- Use named exports, not default exports
- Error handling: wrap async route handlers with asyncHandler()
- API responses follow { data, error, meta } shape
- Tests colocated with source files as *.test.ts

## Avoid
- Don't use `any` type -- use `unknown` and narrow
- Don't add dependencies without checking bundle size impact
- Don't modify the shared package without updating both apps
Keep it concise
CLAUDE.md is loaded into every conversation, so every line costs context. Aim for density over length. If you find your CLAUDE.md exceeding 200 lines, consider splitting specialized instructions into skills instead.
You can have multiple CLAUDE.md files. One at the repo root, one in
~/.claude/CLAUDE.md
for global preferences, and directory-specific ones like
apps/api/CLAUDE.md
that only load when Claude is working in that directory.
2. Use Skills for Repeatable Workflows
If you find yourself giving Claude the same instructions across multiple sessions ("review this code for security issues", "write tests in our TDD style", "create a new API endpoint following our pattern"), extract those instructions into a
skill
.
Skills live in
.claude/skills/
and are invoked with
slash commands
:
# .claude/skills/api-endpoint.md
---
name: New API Endpoint
command: /api
---

When creating a new API endpoint, follow this structure:

1. Create route handler in apps/api/src/routes/
2. Define request/response types in packages/shared/
3. Add input validation using zod schemas
4. Write integration test with supertest
5. Update the OpenAPI spec in docs/api.yaml
This turns a multi-paragraph instruction into
/api create a user profile endpoint
. Test your skills before committing them using the
Skills Playground
.
3. Set Up Hooks for Enforcement
CLAUDE.md instructions are advisory -- Claude follows them most of the time, but they're not guaranteed. For things that
must
happen, use
hooks
.
The most valuable hooks for most teams:
Auto-format on write
-- PostToolUse hook on Write/Edit to run Prettier or your formatter
Test feedback loop
-- Stop hook that runs tests and feeds failures back to Claude
Protected file guard
-- PreToolUse hook on Write/Edit that blocks changes to critical files
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
4. Be Specific in Your Prompts
The quality of Claude's output is directly proportional to the specificity of your input. Compare:
Vague:
"Fix the login bug"
Specific:
"Users report getting a 401 error when logging in with email+password. The issue started after commit abc123. Check the auth middleware in apps/api/src/middleware/auth.ts."
Specific prompts save Claude from spending time exploring and guessing, and they produce better results on the first try.
Point Claude to the right files
If you know where the relevant code is, say so. Claude will explore your codebase if needed, but direct references save context and time:
Look at apps/api/src/routes/users.ts and fix the
pagination bug -- the offset calculation is wrong
when page=1 with pageSize=20.
5. Use Plan Mode for Complex Tasks
For tasks that span multiple files or require architectural decisions, start with
/plan
. This puts Claude in planning mode where it outlines its approach before making any changes. You can review, adjust, and approve the plan before Claude executes.
Plan mode is especially valuable for:
Refactoring across many files
Adding a new feature that touches multiple layers
Migrating from one pattern or library to another
Any task where you want to review the approach first
6. Commit Frequently
Ask Claude to commit after completing discrete units of work. This gives you clean rollback points if something goes wrong. A good pattern:
Ask Claude to implement feature X
Review the changes
Ask Claude to commit with a descriptive message
Move to the next task
Avoid letting Claude make many changes across a long session without committing. If you need to undo later, you'll have to untangle everything manually.
7. Leverage Multi-Turn Conversations
Claude Code maintains context across turns in a conversation. Use this to iterate:
"Write a function to parse CSV files with header detection"
"Add error handling for malformed rows"
"Now write tests covering the edge cases we discussed"
"The test for empty files is wrong -- it should throw, not return empty"
Each turn builds on the previous context. This is more effective than trying to specify everything in one massive prompt.
8. Review What Claude Writes
Claude is a powerful tool, not a replacement for engineering judgment. Always review changes before committing, especially for:
Security-sensitive code
-- auth, encryption, input validation
Database migrations
-- schema changes are hard to undo in production
Public API changes
-- breaking changes affect downstream consumers
Performance-critical paths
-- hot loops, database queries, memory allocation
Use
git diff
to review changes before committing. Claude can help here too -- ask it to explain what it changed and why.
9. Customize Your Permission Mode
Claude Code has three permission modes that control how much autonomy Claude has:
Suggest
-- Claude proposes changes but doesn't execute anything without approval
Auto-edit
-- Claude can read/write files but asks before running shell commands
YOLO/Autonomous
-- Claude executes everything without asking (use with caution, and combine with hooks for safety)
Start with auto-edit mode. As you build trust and set up appropriate hooks for guardrails, you can move toward more autonomous operation.
10. Build a Team Skill Library
The highest-leverage practice for teams: maintain a shared collection of skills in your repository. Common skills to create:
/review
-- code review with your team's standards
/test
-- test writing in your team's style
/deploy
-- deployment checklist and steps
/debug
-- systematic debugging workflow
/onboard
-- project overview for new Claude sessions
When everyone on the team uses the same skills, Claude's output becomes consistent across the team. New team members benefit immediately from the encoded knowledge.