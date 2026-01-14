# agents-context

Automatically bring [AGENTS.md](https://agents.md/) conventions to Claude Code
by converting `AGENTS.md` files into Claude Code's [path-specific
rules](https://code.claude.com/docs/en/memory#path-specific-rules).

## Overview

The plugin `agents-context` automatically discovers
[AGENTS.md](https://agents.md/) files in your repository and converts them to
Claude Code's [path-specific
rules](https://code.claude.com/docs/en/memory#path-specific-rules) format. On
every session start, it finds all `AGENTS.md` files, replicates them to
`.claude/rules/agents/`, and prepends YAML `frontmatter` that defines which
paths each file applies to.

This allows you to use the popular [AGENTS.md](https://agents.md/) convention
while seamlessly integrating with Claude Code's native [rules
system](https://code.claude.com/docs/en/memory#modular-rules-with-claude).

## How It Works

1. **SessionStart Hook**: Runs automatically when you start a Claude Code session
2. **Discovery**: Finds all `AGENTS.md` files in your repository (respects `.gitignore`)
3. **Transformation**: Copies each `AGENTS.md` to `.claude/rules/agents/` with
   YAML `frontmatter`
4. **Path Mapping**: Adds path patterns based on the `AGENTS.md` location:
   - Root `AGENTS.md` → applies to `**/*` (all files)
   - `src/api/AGENTS.md` → applies to `src/api/**/*` (API directory and subdirectories)
   - `docs/AGENTS.md` → applies to `docs/**/*` (docs directory and subdirectories)

The plugin uses `git ls-files` when in a git repository for efficiency and
proper `.gitignore` handling. In non-git directories, it falls back to `find`.

## Installation

### From Marketplace

```bash
# Add the claude-contrib marketplace
/plugin marketplace add claude-contrib/claude-plugins

# Install the plugin
/plugin install agents-context@claude-plugins
```

### Local Development

```bash
# From the repository root
/plugin marketplace add .
/plugin install agents-context@claude-plugins
```

## Setup

After installing the plugin, add the generated rules directory to your `.gitignore`:

```
# Claude Code - Generated rules from AGENTS.md files
.claude/rules/agents/
```

These files are automatically generated on each session start, so they
shouldn't be committed to version control. Your source `AGENTS.md` files should
be tracked by git, but the transformed files in `.claude/rules/agents/` should
not.

## Usage

Once installed, the plugin works automatically. Simply create `AGENTS.md` files
in your repository:

```markdown
# Your Project AGENTS.md

You are an expert in this codebase. When working in this area:

- Follow the existing patterns
- Run tests before committing
- Update documentation
```

On the next Claude Code session start, this will be automatically available as
a path-specific rule.

## Examples

### Example 1: Root-Level Guidelines

**File**: `AGENTS.md` (repository root)

```markdown
# Project Guidelines

This is a TypeScript project. Always:

- Use strict mode
- Write tests for new features
- Follow the ESLint configuration
```

**Result**: Creates `.claude/rules/agents/AGENTS.md` with:

```yaml
---
paths:
  - "**/*"
---
# Project Guidelines

This is a TypeScript project. Always:
  - Use strict mode
  - Write tests for new features
  - Follow the ESLint configuration
```

This rule applies to **all files** in the repository.

### Example 2: Directory-Specific Guidelines

**File**: `src/api/AGENTS.md`

```markdown
# API Guidelines

When working with API code:

- All endpoints must have OpenAPI documentation
- Validate input with Zod schemas
- Use error handling middleware
```

**Result**: Creates `.claude/rules/agents/src/api/AGENTS.md` with:

```yaml
---
paths:
  - "src/api/**/*"
---
# API Guidelines

When working with API code:
  - All endpoints must have OpenAPI documentation
  - Validate input with Zod schemas
  - Use error handling middleware
```

This rule applies only to files in `src/api/` and its subdirectories.

### Example 3: Multiple AGENTS.md Files

You can have multiple AGENTS.md files for different areas:

```
your-repo/
├── AGENTS.md                    # General project guidelines
├── src/
│   ├── api/AGENTS.md           # API-specific guidelines
│   └── ui/AGENTS.md            # UI-specific guidelines
└── docs/AGENTS.md              # Documentation guidelines
```

Each will be converted with appropriate path patterns, and Claude Code will
apply the relevant rules based on which files you're working with.

## Directory Structure

### Before (Your Repository)

```
your-repo/
├── AGENTS.md
├── src/
│   └── api/
│       └── AGENTS.md
└── docs/
    └── AGENTS.md
```

### After (Generated Rules)

```
your-repo/
├── .claude/
│   └── rules/
│       └── agents/
│           ├── AGENTS.md              # paths: ["**/*"]
│           ├── src/
│           │   └── api/
│           │       └── AGENTS.md      # paths: ["src/api/**/*"]
│           └── docs/
│               └── AGENTS.md          # paths: ["docs/**/*"]
```

The `.claude/rules/agents/` directory is regenerated on every session start, so
you should never edit files there directly. Always edit the source `AGENTS.md`
files in your repository.

## Benefits

### Why Use Agents-Context Instead of Manual Rules?

1. **Convention Over Configuration**: Use the familiar `AGENTS.md` convention
   without learning Claude Code's rules syntax
2. **Automatic Path Mapping**: No need to manually configure path patterns -
   they're derived from file location
3. **Source of Truth**: Keep your `AGENTS.md` files alongside your code, not
   hidden in `.claude/`
4. **Version Control**: Your `AGENTS.md` files are part of your repository and
   can be versioned, reviewed, and shared
5. **No Duplication**: Write guidelines once, sync them automatically

## Technical Details

### Nested Git Repositories (Submodules)

When working with nested git repositories (e.g., git submodules), each repository is processed independently:

- **Parent repo**: Only processes AGENTS.md files in the parent repository
- **Submodule repos**: Each submodule processes its own AGENTS.md files
- **Rule scope**: Rules from parent don't apply to submodule files (and vice versa)

This behavior matches git's philosophy that submodules are independent repositories.

**Example**:
```
parent-repo/              (git repo)
├── AGENTS.md            → Creates parent-repo/.claude/rules/agents/AGENTS.md
└── submodule/           (git repo)
    └── AGENTS.md        → Creates submodule/.claude/rules/agents/AGENTS.md
```

Each repository maintains its own `.claude/rules/agents/` directory with rules that apply only to files within that repository.

**Working with Submodules**:

If you need to share guidelines between parent and submodule:

1. **Separate Installation**: Install the plugin in each repository's Claude Code workspace
2. **Copy Files**: Manually copy AGENTS.md files between repos if needed
3. **Symbolic Links**: Create symlinks to share AGENTS.md (will be treated as the repo's own)
4. **Shared Guidelines**: Place common guidelines in parent's root AGENTS.md, and reference them in submodule AGENTS.md

**Why This Behavior?**

Git submodules are independent repositories with their own history, branches, and commits. The agents-context plugin respects this independence by processing each repository separately. This prevents:
- Rule conflicts between parent and submodule
- Unexpected behavior when opening repos as separate workspaces
- Confusion about which rules apply to which files

## Troubleshooting

### Submodule AGENTS.md Not Found

**Symptom**: AGENTS.md files in submodules aren't being processed

**Explanation**: This is expected behavior. Each git repository is processed independently. When the plugin runs in a parent repo, it only processes AGENTS.md files in the parent (not in submodules).

**Solution**:
- Run Claude Code in the submodule directory to process its AGENTS.md
- Or, install the plugin in a workspace opened at the submodule level

## Contributing

Issues and pull requests welcome at the [claude-plugins repository](https://github.com/claude-contrib/claude-plugins).

## License

MIT

## Version

**1.0.0** - Initial release
