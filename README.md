# Agent Skills Collection

A collection of Agent Skills following the [Agent Skills specification](https://agentskills.io/). These skills provide specialized capabilities for AI agents like Claude Code, OpenCode, Cursor, and other compatible platforms.

## Overview

Agent Skills are folders of instructions, scripts, and resources that agents can discover and use to perform tasks more accurately and efficiently. This repository contains curated skills for common development workflows.

## Available Skills

### 1. conventional-commits

Follow the [Conventional Commits specification](https://www.conventionalcommits.org/) for writing standardized commit messages.

**Features:**

- Structured commit message format
- Semantic versioning integration
- Automated changelog generation
- Pre-commit hook integration

**Use when:** Creating git commits, writing commit messages, or working with semantic versioning.

### 2. skill-creator

Create and test Agent Skills following the Test-Driven Development methodology.

**Features:**

- TDD approach to skill creation
- Skill validation and testing
- Best practices for skill documentation
- Anti-patterns to avoid

**Use when:** Creating new skills, editing existing skills, or verifying skills work before deployment.

### 3. deep-wiki

Generate comprehensive wiki documentation for any codebase, similar to [deepwiki.com](https://deepwiki.com).

**Features:**

- Code structure analysis
- Architecture documentation
- API reference generation
- Interactive navigation
- Multiple output formats (HTML, Markdown)

**Use when:** Documenting a codebase, creating wiki pages, or generating architecture documentation.

### 4. system-performance

Apply Brendan Gregg's performance analysis methodologies for investigating system performance issues.

**Features:**

- USE Method (Utilization, Saturation, Errors)
- RED Method (Rate, Errors, Duration)
- Flame graphs visualization
- Performance analysis workflows
- Linux performance tools reference

**Use when:** Analyzing system performance, debugging performance issues, or investigating bottlenecks.

### 5. skill-auditor

Find and remove duplicate skills across installed skill directories, validate skill quality, and ensure specification compliance.

**Features:**

- Duplicate skill detection
- Specification compliance validation
- Quality checks and warnings
- Automated audit reports
- Cleanup commands

**Use when:** Auditing skills for duplicates, validating skill quality, or checking specification compliance.

### 6. code-reviewer

Conduct thorough code reviews with systematic checks for code quality, bugs, security issues, performance problems, and adherence to best practices.

**Features:**

- Systematic review process (6 phases)
- Security, quality, performance, testing review
- Code review checklist and feedback guidelines
- Before/after code comparisons
- Issue prioritization

**Use when:** Reviewing pull requests, conducting code reviews, or analyzing code quality.

### 7. security-expert

Identify and prevent security vulnerabilities using OWASP Top 10, secure coding practices, and systematic vulnerability analysis.

**Features:**

- OWASP Top 10 coverage
- Input validation and sanitization
- Authentication and authorization patterns
- API security best practices
- Dependency vulnerability scanning

**Use when:** Analyzing security vulnerabilities, conducting security reviews, or checking for common vulnerabilities.

### 8. frontend-design

Design and implement distinctive, production-ready frontend interfaces with strong aesthetic direction.

**Features:**

- Design thinking + execution approach
- Purpose-driven visual choices
- Support for HTML/CSS/JS, React, Vue, and more
- Accessibility and performance considerations

**Use when:** Creating or restyling web pages, components, dashboards, or application UIs.

### 9. native-web-search

Trigger native web search for fast internet research with concise summaries and full source URLs.

**Features:**

- Fast model with native web search
- Concise research summaries
- Explicit full source URLs
- Command-line interface

**Use when:** Performing internet research, finding current information, or getting quick answers with sources.

### 10. summarize

Fetch URLs or convert local files (PDF/DOCX/HTML/etc.) into Markdown for analysis, optionally with summarization.

**Features:**

- Convert web pages to Markdown
- Binary document conversion (PDF/DOCX/PPTX)
- Optional summarization
- Supports various file formats

**Use when:** Converting documents for analysis, pulling down web pages as Markdown, or summarizing long documents.

### 11. uv

Use `uv` instead of pip/python/venv for Python package and environment management.

**Features:**

- Fast Python package installer
- Inline script dependencies
- Virtual environment management
- Script execution with `uv run`

**Use when:** Running Python scripts, managing dependencies, or creating reproducible Python environments.

## Installation

### Claude Code

Claude Code supports Agent Skills natively. Skills are stored in `~/.claude/skills/` directory.

#### Option 1: Clone Repository

```bash
# Clone the repository
git clone https://github.com/ntk148v/skills.git

# Copy skills to Claude Code directory
cp -r skills/skills/* ~/.claude/skills/
```

#### Option 2: Manual Installation

```bash
# Create skills directory if it doesn't exist
mkdir -p ~/.claude/skills

# Copy individual skills
cp -r skills/conventional-commits ~/.claude/skills/
cp -r skills/skill-creator ~/.claude/skills/
cp -r skills/deep-wiki ~/.claude/skills/
cp -r skills/system-performance ~/.claude/skills/
cp -r skills/skill-auditor ~/.claude/skills/
cp -r skills/code-reviewer ~/.claude/skills/
cp -r skills/security-expert ~/.claude/skills/
cp -r skills/frontend-design ~/.claude/skills/
cp -r skills/native-web-search ~/.claude/skills/
cp -r skills/summarize ~/.claude/skills/
cp -r skills/uv ~/.claude/skills/
```

#### Option 3: Symlink (Recommended for Development)

```bash
# Create symlink to repository
ln -s $(pwd)/skills ~/.claude/skills
```

### OpenCode

OpenCode supports Agent Skills through the `.opencode/skills/` directory.

```bash
# Clone the repository
git clone https://github.com/ntk148v/skills.git

# Copy skills to OpenCode directory
cp -r skills/skills/* .opencode/skills/
```

### Claude Plugin Marketplace

Claude Code supports installing skills via the Claude Plugin Marketplace. This allows you to install the entire skill collection or individual skill packages with a single command.

#### Install Entire Collection

```bash
claude plugin install ntk148v/skills
```

#### Install Individual Packages

```bash
# Install development skills (conventional-commits, code-reviewer, security-expert, system-performance, uv)
claude plugin install ntk148v/skills#development-skills

# Install documentation skills (deep-wiki, summarize)
claude plugin install ntk148v/skills#documentation-skills

# Install skill authoring skills (skill-creator, skill-auditor)
claude plugin install ntk148v/skills#skill-authoring

# Install utility skills (frontend-design, native-web-search)
claude plugin install ntk148v/skills#utility-skills
```

#### Verify Installation

After installation, verify skills are available:

```bash
# List installed skills
ls ~/.claude/skills/

# Expected output includes:
# conventional-commits/
# code-reviewer/
# security-expert/
# system-performance/
# deep-wiki/
# skill-creator/
# skill-auditor/
# frontend-design/
# native-web-search/
# summarize/
# uv/
```

### Cursor

Cursor supports Agent Skills through the `.cursor/skills/` directory.

```bash
# Clone the repository
git clone https://github.com/ntk148v/skills.git

# Copy skills to Cursor directory
cp -r skills/skills/* .cursor/skills/
```

### Other Platforms

Most Agent Skills-compatible platforms follow a similar pattern:

1. **Locate the skills directory** (usually `~/.<platform>/skills/` or `.<platform>/skills/`)
2. **Copy the skill folders** to that directory
3. **Restart the platform** if required

Check your platform's documentation for specific instructions.

## Verification

After installation, verify that skills are recognized:

### Claude Code

```bash
# List installed skills
ls ~/.claude/skills/

# Expected output:
# conventional-commits/
# deep-wiki/
# system-performance/
# code-reviewer/
# security-expert/
# skill-auditor/
# skill-creator/
# frontend-design/
# native-web-search/
# summarize/
# uv/
```

### OpenCode

```bash
# List installed skills
ls .opencode/skills/

# Expected output:
# conventional-commits/
# deep-wiki/
# system-performance/
# code-reviewer/
# security-expert/
# skill-auditor/
# skill-creator/
# frontend-design/
# native-web-search/
# summarize/
# uv/
```

## Usage

Skills are automatically discovered and loaded by compatible platforms. When you ask the agent to perform a task related to a skill's domain, the skill will be activated.

### Example Prompts

**Conventional Commits:**

```
"Create a commit for the new authentication feature"
"Write a commit message following conventional commits format"
```

**Skill Creator:**

```
"Create a new skill for API documentation"
"Help me write a skill for database migrations"
```

**Deep Wiki:**

```
"Generate documentation for this codebase"
"Create a wiki for the authentication module"
```

**System Performance:**

```
"Analyze why the server is slow"
"Investigate CPU performance issues"
"Use the USE method to check for bottlenecks"
```

**Frontend Design:**

```
"Create a landing page for my product"
"Design a dashboard with charts"
"Improve the typography on this page"
```

**Native Web Search:**

```
"Search for the latest React news"
"Find information about Docker containers"
"Look up the newest Python release"
```

**Summarize:**

```
"Summarize this article"
"Convert this PDF to Markdown"
"Turn this URL into readable text"
```

**UV:**

```
"Run this Python script with dependencies"
"Add requests to the project"
"Create a new Python environment"

## Skill Structure

Each skill follows the Agent Skills specification:

```

skill-name/
├── SKILL.md # Required: metadata + instructions
├── scripts/ # Optional: executable code
├── references/ # Optional: documentation
└── assets/ # Optional: templates, resources

````

### SKILL.md Format

```yaml
---
name: skill-name
description: Use when [specific triggering conditions]
---

# Skill Name

## Overview
[Core principle in 1-2 sentences]

## When to Use
[Specific use cases]

## Implementation
[Detailed instructions]
````

## Contributing

Contributions are welcome! Please follow these guidelines:

1. **Follow the Agent Skills specification** - Ensure your skill follows the [official specification](https://agentskills.io/specification)
2. **Test your skill** - Use the skill-creator methodology to test your skill before submitting
3. **Document thoroughly** - Include clear descriptions, examples, and use cases
4. **Keep it focused** - Each skill should address a specific domain or workflow

### Adding a New Skill

1. Create a new directory in `skills/` with your skill name
2. Create `SKILL.md` with proper frontmatter and content
3. Add any supporting files (scripts, references, assets)
4. Test the skill with compatible platforms
5. Submit a pull request

## Development

### Prerequisites

- Git
- Compatible agent platform (Claude Code, OpenCode, Cursor, etc.)
- Text editor

### Local Development

```bash
# Clone the repository
git clone https://github.com/ntk148v/skills.git
cd skills

# Create symlink for testing
ln -s $(pwd)/skills ~/.claude/skills

# Make changes and test
# Skills will be automatically reloaded by compatible platforms
```

### Testing Skills

Follow the TDD approach from skill-creator:

1. **RED**: Run baseline scenario WITHOUT skill
2. **GREEN**: Write skill addressing baseline failures
3. **REFACTOR**: Close loopholes and re-test

## Resources

- [Agent Skills Specification](https://agentskills.io/specification)
- [Agent Skills Documentation](https://agentskills.io/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Brendan Gregg's Performance Analysis](https://www.brendangregg.com/methodology.html)
- [Deep Wiki](https://deepwiki.com)
- [UV Package Manager](https://github.com/astral-sh/uv)
- [Markitdown](https://github.com/mrowa44/markitdown)

## Supported Platforms

These skills are compatible with:

- [Claude Code](https://claude.ai/code)
- [OpenCode](https://opencode.ai/)
- [Cursor](https://cursor.com/)
- [GitHub Copilot](https://github.com/features/copilot)
- [VS Code](https://code.visualstudio.com/)
- [And many more...](https://agentskills.io/)

See the [Agent Skills homepage](https://agentskills.io/) for a complete list of compatible platforms.

## License

MIT License - See [LICENSE](LICENSE) for details.

## Acknowledgments

- [Anthropic](https://www.anthropic.com/) for creating the Agent Skills specification
- [Brendan Gregg](https://www.brendangregg.com/) for performance analysis methodologies
- [Conventional Commits](https://www.conventionalcommits.org/) for the commit message specification
- [Open Source Community](https://github.com/anthropics/skills) for skill examples and best practices

## Support

- **Issues**: [GitHub Issues](https://github.com/ntk148v/skills/issues)
- **Discussions**: [GitHub Discussions](https://github.com/ntk148v/skills/discussions)
- **Agent Skills Community**: [Discord](https://discord.gg/MKPE9g8aUy)

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for detailed changelog.
