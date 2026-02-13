---
name: improve-skill
description: >
  Use when the user wants to reflect on recent work and improve the skills, tools,
  and documentation in this plugin marketplace.
  Triggers: "improve skills", "improve tools", "make skills better",
  "reflect and improve", "improve the repo".
argument-hint: "[optional focus area: skills, docs, tools, or specific plugin name]"
user-invocable: true
disable-model-invocation: true
allowed-tools: Bash, Read, Write, Edit, Grep, Glob, Task
---

# Improve Skills and Tools

Announce: "I'm using the improve skill to reflect on recent work and improve the skills, tools, and documentation in this repo."

## Step 1: Gather Context

Run in parallel:

- `git rev-parse --show-toplevel` (confirm repo root)
- `git log --oneline -10` (recent work for context)

Then discover repo assets (run Glob calls in parallel):

- `**/skills/*/SKILL.md` (all skills)
- `**/.claude-plugin/plugin.json` (all plugin manifests)
- `**/agents/*.md` (agent definitions)

Read each plugin's `plugin.json` to note current versions (needed for version bumps later).

Read `AGENTS.md` for current repo conventions.

**If not in the code-factory repo:** inform the user this skill is designed for the code-factory plugin marketplace and stop.

## Step 2: Reflect on Recent Experience

Think through what you just worked on in this session. Evaluate across four dimensions:

### What slowed you down?
- Where did you hesitate or get confused by a skill?
- What error messages from tools didn't help?
- What information did you have to hunt for?

### What wasted tokens?
- Verbose documentation you had to wade through
- Skill content that could be more concise
- Redundant information across skills

### What was missing?
- Manual steps that should be automated (Makefile targets, validation)
- A skill that should exist but doesn't
- Gaps in error handling or edge cases

### What was confusing?
- Skill instructions that were ambiguous
- Inconsistent patterns across skills
- Surprising behavior from tools or unclear defaults

If `$ARGUMENTS` specifies a focus area (e.g., a plugin name or "docs"), narrow reflection to that area.

If there is no recent session context, ask the user what area they'd like to improve or offer a general audit of all skills.

## Step 3: Make Improvements

Apply changes directly. You have full authority to improve any file in this repo. Do not ask permission for improvements within existing files.

### Improving Skills

**Location:** `{plugin}/skills/{name}/SKILL.md`

Principles:
- **Concise:** one sentence per concept, tables over paragraphs, no filler words ("simply", "just", "easily")
- **Scannable:** clear headings, bullet points, working examples
- **Complete:** all edge cases addressed, error handling section present
- **Consistent:** follow established patterns — Announce line, numbered Steps, Error Handling section
- **Self-contained:** each skill must work without external context

Structure priorities:
1. Quick-start or most common workflow early
2. Copy-paste ready code blocks
3. Tables for reference, not paragraphs

**Version bump required:** Any change to a skill or agent file requires a version bump in the owning plugin's `.claude-plugin/plugin.json` (patch for fixes, minor for new features).

### Improving Documentation

**Files:** `AGENTS.md`, `README.md`, skill `SKILL.md` files

- Keep instructions actionable and specific
- Remove ambiguity that caused confusion during your session
- Add missing conventions you discovered

### Improving Tools

**Files:** `Makefile`, `init.sh`, config files

- Minimal output on success, clear messages on failure
- Sensible defaults, fail fast
- Add missing validation targets if gaps are found

### Creating New Skills

If a skill should exist but doesn't:

1. Determine which plugin it belongs to (productivity, git, code)
2. Create `{plugin}/skills/{name}/SKILL.md`
3. Use YAML frontmatter: `name`, `description` (starts with "Use when..."), `argument-hint`, `user-invocable: true`
4. Follow the Announce → Steps → Error Handling structure
5. Update the plugin's `.claude-plugin/plugin.json` version (minor bump)

**For significant new features**, suggest running `/execplan` first instead of implementing inline.

### Creating New Plugins

If a new plugin is warranted:

1. Create `{name}/.claude-plugin/plugin.json` with name, version, description, author
2. Create skill directories under `{name}/skills/`
3. Add the plugin to `.claude-plugin/marketplace.json`
4. Confirm with the user before proceeding (per AGENTS.md boundaries)

## Step 4: Validate

Run after all changes:

```bash
make all
```

All checks must pass. Fix any failures before proceeding.

Also verify:
- Updated skills read clearly to someone encountering them for the first time
- No broken cross-references between skills
- Any new files follow the naming conventions in AGENTS.md

## Step 5: Report

Present a summary of all changes:

```
## Improvements Made

### Skills Updated
- **{plugin}:{skill-name}**: {what changed and why}

### Documentation Updated
- **{file}**: {what changed and why}

### Tools Improved
- **{tool}**: {what changed and why}

### New Skills Created
- **{plugin}:{skill-name}**: {what it does}

### New Plugins Created
- **{plugin}**: {what it contains}

### Version Bumps
- **{plugin}**: {old} → {new}

### Suggested Follow-ups
- {anything that needs deeper work via /execplan}
```

Omit any section with no entries.

## Error Handling

- **Not in code-factory repo**: inform the user and stop.
- **No recent session context**: ask the user what area to improve, or offer a general audit of all skills.
- **`make all` failure**: fix the issues and re-run validation. Do not report until all checks pass.
- **Significant interface change**: if a change would modify a skill's name, arguments, or behavior substantially, describe the proposed change and ask the user before applying it.
