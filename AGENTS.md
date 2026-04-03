# AGENTS.md - Agent Skills Repository

This file provides guidance for agentic coding agents (such as Claude Code, OpenCode, Cursor, etc.) working in this repository.

## Repository Overview

This is an **Agent Skills repository** containing specialized skills for AI agents. Each skill follows the [Agent Skills specification](https://agentskills.io/) and provides domain-specific knowledge and workflows.

**Repository Structure:**

```
skills/
├── conventional-commits/    # Git commit message standards
│   └── SKILL.md
├── deep-wiki/              # Codebase documentation generation
│   └── SKILL.md
├── skill-auditor/         # Find and remove duplicate skills
│   └── SKILL.md
├── skill-creator/         # Create and test Agent Skills
│   └── SKILL.md
├── system-performance/     # Performance analysis methodologies
│   └── SKILL.md
├── code-reviewer/          # Conduct thorough code reviews
│   └── SKILL.md
└── security-expert/       # Identify and prevent vulnerabilities
    └── SKILL.md
```

**OpenCode Integration:**

```
.opencode/
├── skills/                 # Skills installed for OpenCode
│   └── deep-wiki/
└── memory/                 # Persistent memory storage
```

## Build/Lint/Test Commands

This repository contains **documentation only** (Markdown files). There are no build, lint, or test commands.

**Workflow:**

1. Edit SKILL.md files directly
2. Validate YAML frontmatter manually
3. Test skills by using them in compatible agent platforms
4. Commit changes using conventional commits format

**Validation:**

- Use the [skills-ref](https://github.com/agentskills/agentskills/tree/main/skills-ref) tool to validate skill format:
  ```bash
  skills-ref validate ./skills/skill-name
  ```

## Code Style Guidelines

### YAML Frontmatter

Every SKILL.md file MUST start with YAML frontmatter:

```yaml
---
name: skill-name-with-hyphens
description: Use when [specific triggering conditions and symptoms]
---
```

**Rules:**

- `name`: 1-64 characters, lowercase letters/numbers/hyphens only, no consecutive hyphens
- `description`: Max 1024 characters, starts with "Use when...", describes ONLY when to use (NOT what it does)
- Must match parent directory name
- Write in third person

### Markdown Formatting

**Headers:**

- Use ATX-style headers (`#`, `##`, `###`)
- Start with `# Skill Name` (matches frontmatter `name`)
- Follow with `## Overview` as second section

**Required Sections:**

1. `## Overview` - Core principle in 1-2 sentences
2. `## When to Use` - Specific use cases and symptoms
3. Implementation sections (varies by skill type)

**Code Blocks:**

- Specify language: ` ```bash`, ` ```yaml`, ` ```markdown`
- Use fenced code blocks (triple backticks)
- Keep examples complete and runnable

**Lists:**

- Use `-` for unordered lists
- Use `1.` for ordered lists
- Indent nested items with 2 spaces

**Links:**

- Use `[text](url)` format
- Prefer HTTPS URLs
- Reference external documentation when appropriate

### File Organization

**Skill Directory:**

```
skill-name/
├── SKILL.md              # Required: Main skill file
├── scripts/              # Optional: Executable scripts
├── references/           # Optional: Additional documentation
└── assets/               # Optional: Templates, images
```

**Keep SKILL.md under 500 lines.** Move heavy reference material to separate files in `references/`.

### Naming Conventions

**Skill Names:**

- Use active voice, verb-first: `creating-skills`, `condition-based-waiting`
- Use hyphens, not underscores: `deep-wiki`, not `deep_wiki`
- Be descriptive: `system-performance`, not `perf`

**Directory Names:**

- Must match `name` in frontmatter exactly
- Lowercase only
- No special characters except hyphens

**File Names:**

- Use `SKILL.md` (uppercase) for main skill file
- Use lowercase with hyphens for supporting files: `api-reference.md`, `code-templates.md`

## Skill Creation Guidelines

### TDD Methodology (from writing-skills)

**RED Phase:**

1. Create pressure scenarios (3+ combined pressures for discipline skills)
2. Run scenarios WITHOUT skill - document baseline behavior verbatim
3. Identify patterns in rationalizations/failures

**GREEN Phase:**

1. Write minimal skill addressing specific baseline failures
2. Run scenarios WITH skill - verify agents now comply
3. Keep skill concise (<500 words for frequently-loaded skills)

**REFACTOR Phase:**

1. Identify NEW rationalizations from testing
2. Add explicit counters (if discipline skill)
3. Build rationalization table from all test iterations
4. Re-test until bulletproof

### Skill Types

**Technique:** Concrete method with steps (e.g., condition-based-waiting)

- Include before/after code comparison
- Provide quick reference table
- Add common mistakes section

**Pattern:** Way of thinking about problems (e.g., flatten-with-flags)

- Explain mental model
- Show recognition scenarios
- Include counter-examples

**Reference:** API docs, syntax guides (e.g., conventional-commits)

- Provide comprehensive tables
- Include examples for common cases
- Link to official documentation

### Description Best Practices

**DO:**

- Start with "Use when..."
- Include specific symptoms and situations
- Describe the problem, not language-specific symptoms
- Keep under 500 characters

**DON'T:**

- Summarize the skill's workflow
- Use first person ("I can help...")
- Be vague ("For async testing")
- Include implementation details

**Example:**

```yaml
# ✅ GOOD
description: Use when tests have race conditions, timing dependencies, or pass/fail inconsistently

# ❌ BAD
description: Use for TDD - write test first, watch it fail, write minimal code, refactor
```

## Testing Skills

### Testing Methodology

**Discipline-Enforcing Skills:**

- Academic questions: Do they understand the rules?
- Pressure scenarios: Do they comply under stress?
- Multiple pressures combined: time + sunk cost + exhaustion

**Technique Skills:**

- Application scenarios: Can they apply the technique correctly?
- Variation scenarios: Do they handle edge cases?
- Missing information tests: Do instructions have gaps?

**Pattern Skills:**

- Recognition scenarios: Do they recognize when pattern applies?
- Application scenarios: Can they use the mental model?
- Counter-examples: Do they know when NOT to apply?

**Reference Skills:**

- Retrieval scenarios: Can they find the right information?
- Application scenarios: Can they use what they found correctly?
- Gap testing: Are common use cases covered?

### Common Rationalizations to Avoid

| Excuse                         | Reality                                                   |
| ------------------------------ | --------------------------------------------------------- |
| "Skill is obviously clear"     | Clear to you ≠ clear to other agents. Test it.            |
| "It's just a reference"        | References can have gaps. Test retrieval.                 |
| "Testing is overkill"          | Untested skills have issues. Always.                      |
| "I'll test if problems emerge" | Problems = agents can't use skill. Test BEFORE deploying. |

## Best Practices

### Token Efficiency

**Frequently-loaded skills (<200 words):**

- Move details to tool help
- Use cross-references
- Compress examples

**Other skills (<500 words):**

- Keep concise
- One excellent example beats many mediocre ones
- Link to heavy reference material

### Cross-Referencing

Use skill name only, with explicit requirement markers:

- ✅ `**REQUIRED SUB-SKILL:** Use superpowers:test-driven-development`
- ✅ `**REQUIRED BACKGROUND:** You MUST understand superpowers:systematic-debugging`
- ❌ `See skills/testing/test-driven-development` (unclear if required)
- ❌ `@skills/testing/test-driven-development/SKILL.md` (force-loads, burns context)

### Anti-Patterns

**Don't:**

- Create narrative examples ("In session 2025-10-03, we found...")
- Implement in 5+ languages
- Create fill-in-the-blank templates
- Write contrived examples
- Use generic labels (helper1, step2, pattern4)

**Do:**

- Use one excellent, complete, runnable example
- Choose most relevant language
- Show pattern clearly
- Make ready to adapt

## Git Commit Guidelines

Follow the **conventional-commits** skill for all commits:

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**Types:** `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

**Examples:**

- `feat: add new skill for API documentation`
- `fix: correct YAML frontmatter validation`
- `docs: update installation instructions`
- `refactor: reorganize skill directory structure`

## Quality Checklist

Before submitting a new skill or updating an existing one:

- [ ] YAML frontmatter is valid (name matches directory, description <1024 chars)
- [ ] Description starts with "Use when..." and describes triggering conditions
- [ ] Overview section explains core principle in 1-2 sentences
- [ ] When to Use section lists specific symptoms and use cases
- [ ] Skill is under 500 lines (move heavy reference to separate files)
- [ ] Code examples are complete and runnable
- [ ] Links are valid and use HTTPS
- [ ] No narrative storytelling or contrived examples
- [ ] Skill has been tested with pressure scenarios
- [ ] Commit message follows conventional commits format

## Resources

- [Agent Skills Specification](https://agentskills.io/specification)
- [Agent Skills Documentation](https://agentskills.io/)
- [skills-ref Validation Tool](https://github.com/agentskills/agentskills/tree/main/skills-ref)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [writing-skills Methodology](./skills/writing-skills/SKILL.md)

## Notes for Agents

1. **This is a documentation repository** - No build system, tests, or linting
2. **Validate YAML frontmatter** - Use skills-ref tool or manual inspection
3. **Test skills manually** - Use them in compatible agent platforms
4. **Follow conventional commits** - All commits should follow the specification
5. **Keep skills concise** - Token efficiency is critical for frequently-loaded skills
6. **Test before deploying** - Follow TDD methodology from writing-skills
7. **Cross-reference properly** - Use skill names with explicit requirement markers
8. **One excellent example** - Better than many mediocre ones
