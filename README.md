# g-slave

A Claude Code slash command that dispatches Google's Gemini CLI as a subordinate analysis tool. Leverage Gemini's 1M token context window for large codebase analysis while keeping Claude's context clean.

**The hierarchy is clear:** Claude commands, Gemini does the grunt work, you get distilled intel.

## Why?

| Problem | Solution |
|---------|----------|
| Large codebases overflow Claude's context | Gemini's 1M tokens handles entire projects |
| Raw analysis dumps waste tokens | g-slave distills results before returning |
| You want Claude to analyze, not Gemini | Explicit `/g-slave` invocation = you control when |

## Prerequisites

1. **Claude Code** - [Install](https://docs.anthropic.com/en/docs/claude-code)
2. **Gemini CLI** - [Install](https://github.com/google-gemini/gemini-cli)

```bash
# Verify Gemini CLI is installed
which gemini
```

## Installation

### Option 1: Global (all projects)

```bash
# Create Claude commands directory if it doesn't exist
mkdir -p ~/.claude/commands

# Download g-slave.md
curl -o ~/.claude/commands/g-slave.md https://raw.githubusercontent.com/0xDarkMatter/g-slave/main/g-slave.md
```

### Option 2: Project-specific

```bash
# In your project root
mkdir -p .claude/commands
curl -o .claude/commands/g-slave.md https://raw.githubusercontent.com/0xDarkMatter/g-slave/main/g-slave.md
```

## Usage

### Modes

```bash
# Analyze - Full codebase analysis
/g-slave src/
/g-slave .

# Ask - Direct questions
/g-slave ask "Where is user authentication implemented?"
/g-slave ask "How does the payment flow work?"

# Verify - Yes/No checks
/g-slave verify "All API endpoints have rate limiting"
/g-slave verify "SQL injection protection is in place"

# Compare - Diff analysis
/g-slave compare ./main-branch ./feature-branch
/g-slave compare src/v1 src/v2
```

### Focus Flags

Target specific analysis areas:

```bash
/g-slave . --arch        # Architecture, structure, design patterns
/g-slave . --security    # Vulnerabilities, auth, injection risks
/g-slave . --deps        # Dependencies, imports, coupling
/g-slave . --quality     # Code quality, tech debt, antipatterns
/g-slave . --test        # Test coverage, testing patterns, gaps
/g-slave . --perf        # Performance bottlenecks, optimization
```

### Output Control

```bash
/g-slave . --brief       # ~500 char executive summary
/g-slave . --detailed    # ~5000 char comprehensive breakdown
/g-slave . --raw         # Unfiltered Gemini output (no distillation)
/g-slave . --save report.md  # Export analysis to file
```

### Scope Control

```bash
# Exclude token-wasters
/g-slave . --exclude node_modules,dist,.git,coverage

# Full project scan
/g-slave --all

# Custom analysis prompt
/g-slave . --prompt "Find all API endpoints and document their HTTP methods"
```

### Combined Examples

```bash
# Security audit, excluding noise, saved to file
/g-slave . --security --exclude node_modules,dist --save security-audit.md

# Quick architecture overview
/g-slave src/ --arch --brief

# Deep dive with full output
/g-slave . --detailed --save full-analysis.md

# Verify with evidence
/g-slave verify "Authentication uses JWT tokens"
```

## How It Works

```
You (invoke /g-slave)
    |
    v
Claude (parses command, constructs Gemini prompt)
    |
    v
Gemini CLI (analyzes with 1M token context)
    |
    v
Claude (distills results, extracts insights)
    |
    v
You (receive concise, actionable intel)
```

### Key Behaviors

1. **Read-only enforcement** - All Gemini prompts include instructions to never execute code or modify files
2. **Agent file detection** - Checks for AGENTS.md, GEMINI.md, CLAUDE.md, WARP.md, COPILOT.md, CURSOR.md, CODEX.md
3. **Framework detection** - Tailors analysis based on detected stack (package.json, requirements.txt, Cargo.toml, etc.)
4. **Smart exclusions** - Recommends excluding node_modules, .git, dist, vendor, etc.
5. **Distillation by default** - Summarizes unless `--raw` is specified

## Output Formats

### Standard Analysis
```markdown
## Gemini Analysis: src/

**Scope:** src/ directory (47 files)
**Focus:** Architecture

### Architecture
Express.js backend with layered architecture...

### Key Patterns
- Repository pattern for data access
- Middleware chain for auth/validation

### Issues (prioritized)
1. **High:** No rate limiting on auth endpoints
2. **Medium:** Circular dependency in utils/

### Recommendations
1. Add rate limiting middleware
2. Extract shared utils to break cycle
```

### Ask Mode
```markdown
## Answer: Where is authentication implemented?

Authentication is handled in `src/middleware/auth.js` using JWT tokens...

**Evidence:**
- `src/middleware/auth.js:15` - JWT verification
- `src/routes/auth.js:42` - Login endpoint
```

### Verify Mode
```markdown
## Verification: All API endpoints have rate limiting

**Result:** NO

**Evidence:**
- `/api/auth/*` routes lack rate limiting
- `/api/public/*` has rate limiting via express-rate-limit

**Location(s):** `src/routes/auth.js`, `src/middleware/rateLimit.js`
```

## Configuration Tips

### Optimize with GEMINI.md

Create a `GEMINI.md` in your project root for persistent Gemini instructions:

```markdown
# Project Context

This is a Node.js/Express API with PostgreSQL.

## Key Directories
- src/routes - API endpoints
- src/services - Business logic
- src/models - Database models

## Conventions
- All routes require auth middleware except /public/*
- Use async/await, not callbacks
```

Gemini CLI automatically reads this file.

### Default Exclusions

Add to your GEMINI.md to always exclude:

```markdown
## Ignore
- node_modules/
- dist/
- coverage/
- *.log
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "Gemini CLI not installed" | Install from [google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli) |
| Analysis timeout | Use `--exclude` to reduce scope |
| Rate limited | Wait or reduce analysis frequency |
| Output too verbose | Use `--brief` flag |
| Missing context | Ensure path is correct, check GEMINI.md |

## Contributing

PRs welcome. Keep the hierarchy intact.

## License

MIT

---

*Claude commands. Gemini obeys. You win.*
