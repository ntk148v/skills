---
name: conventional-commits
description: Use when creating git commits, writing commit messages, or when the user mentions conventional commits, semantic versioning, or automated changelog generation.
---

# Conventional Commits

## Overview

Conventional Commits is a specification for adding human and machine-readable meaning to commit messages. It provides a standardized format that enables automated tools to generate changelogs, determine semantic version bumps, and communicate the nature of changes clearly.

## When to Use

Use this skill when:

- Creating git commits for any codebase
- Writing commit messages that need to follow a standard format
- The user mentions conventional commits, semantic versioning, or automated changelogs
- Working on projects that use automated release pipelines
- Contributing to open-source projects that require conventional commit format

## Commit Message Format

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Required Elements

1. **Type**: A noun indicating the category of change
2. **Description**: A short summary of the change

### Optional Elements

3. **Scope**: Context for the change (in parentheses)
4. **Body**: Detailed explanation (separated by blank line)
5. **Footer(s)**: Metadata like breaking changes or issue references

## Commit Types

### Core Types (Semantic Versioning Impact)

| Type                                            | Meaning         | Version Bump |
| ----------------------------------------------- | --------------- | ------------ |
| `feat`                                          | New feature     | MINOR        |
| `fix`                                           | Bug fix         | PATCH        |
| `feat!` or `feat` with `BREAKING CHANGE` footer | Breaking change | MAJOR        |

### Additional Types (No Version Impact)

| Type       | Meaning                                                 |
| ---------- | ------------------------------------------------------- |
| `build`    | Build system or external dependencies                   |
| `chore`    | Maintenance tasks, no production code changes           |
| `ci`       | CI/CD configuration and scripts                         |
| `docs`     | Documentation only changes                              |
| `style`    | Code style (formatting, semicolons, etc.)               |
| `refactor` | Code refactoring without fixing bugs or adding features |
| `perf`     | Performance improvements                                |
| `test`     | Adding or modifying tests                               |
| `revert`   | Reverting a previous commit                             |

## Examples

### Simple Commit

```
docs: update README with installation instructions
```

### Commit with Scope

```
feat(api): add user authentication endpoint
```

### Commit with Body

```
fix: prevent race condition in request handler

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.
```

### Breaking Change (Footer Style)

```
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

### Breaking Change (Shorthand Style)

```
feat!: drop support for Node 6

BREAKING CHANGE: use JavaScript features not available in Node 6.
```

### Multiple Footers

```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Reviewed-by: Z
Refs: #123
```

## Writing Good Descriptions

### Do's

- Use imperative mood ("add" not "added" or "adds")
- Start with lowercase letter
- No period at the end
- Be concise but descriptive
- Explain WHAT and WHY, not HOW

### Don'ts

- Don't use vague descriptions like "fix bug" or "update code"
- Don't include issue numbers in the description (use footer instead)
- Don't write long descriptions (use body instead)

## Common Patterns

### Feature Addition

```
feat(auth): implement OAuth2 login flow

Add support for OAuth2 authentication with Google and GitHub providers.
Includes token refresh logic and session management.

Closes #234
```

### Bug Fix

```
fix(parser): handle empty input gracefully

Parser was throwing NullPointerException when receiving empty string.
Added null check and return empty result set instead.

Fixes #567
```

### Breaking Change

```
refactor(api)!: rename user endpoints

BREAKING CHANGE: All /user/* endpoints renamed to /users/*
- /user/profile -> /users/profile
- /user/settings -> /users/settings

Migration guide: Update API client base paths
```

### Documentation

```
docs(contributing): add PR submission guidelines

Add step-by-step guide for submitting pull requests including:
- Branch naming conventions
- Commit message format
- Required CI checks
```

## Workflow Integration

### Pre-commit Hooks

Use tools like commitlint to enforce conventional commit format:

```json
{
  "extends": ["@commitlint/config-conventional"],
  "rules": {
    "type-enum": [
      2,
      "always",
      [
        "feat",
        "fix",
        "docs",
        "style",
        "refactor",
        "perf",
        "test",
        "build",
        "ci",
        "chore",
        "revert"
      ]
    ]
  }
}
```

### Automated Changelog Generation

Tools like standard-version or semantic-release can automatically:

- Generate CHANGELOG.md from commit messages
- Bump version numbers based on commit types
- Create git tags and GitHub releases

### CI/CD Integration

```yaml
# Example GitHub Actions workflow
- name: Semantic Release
  uses: cycjimmy/semantic-release-action@v3
  with:
    semantic_version: 18
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Best Practices

1. **One concern per commit**: Each commit should address one issue or feature
2. **Atomic commits**: Commits should be self-contained and reversible
3. **Write commits first**: Draft commit messages before writing code
4. **Use scopes wisely**: Scopes should be consistent across the project
5. **Reference issues**: Use footers to link to issues, PRs, or tickets
6. **Test before committing**: Ensure tests pass before creating commits
7. **Review your own diff**: Always review changes before committing

## Common Mistakes

| Mistake                    | Correction                                |
| -------------------------- | ----------------------------------------- |
| `feat: Added new feature`  | `feat: add new feature` (imperative mood) |
| `feat: Add new feature.`   | `feat: add new feature` (no period)       |
| `feat: Add New Feature`    | `feat: add new feature` (lowercase)       |
| `feat(api,auth): ...`      | `feat(api): ...` (one scope)              |
| `feat: fix bug`            | `fix: fix bug` (correct type)             |
| `feat: add feature (#123)` | Use footer: `Refs: #123`                  |

## Specification Compliance

This skill follows the Conventional Commits specification v1.0.0. For the complete specification, see: https://www.conventionalcommits.org/

Key rules from the specification:

1. Commits MUST be prefixed with a type
2. Type MUST be followed by a colon and space
3. Scope is OPTIONAL and enclosed in parentheses
4. Breaking changes MUST be indicated with `!` or `BREAKING CHANGE` footer
5. Types other than `feat` and `fix` are allowed
6. The specification is case-insensitive except for `BREAKING CHANGE`

## Quick Reference

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

**Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`

**Breaking**: Add `!` after type/scope OR add `BREAKING CHANGE:` footer

**Version Impact**: `feat` → MINOR, `fix` → PATCH, `BREAKING CHANGE` → MAJOR
