---
name: skill-auditor
description: Use when auditing skills for duplicates, validating skill quality, checking specification compliance, or when the user mentions "duplicate skills", "skill audit", or "skill validation".
---

# Skill Auditor

## Overview

Skill Auditor finds and removes duplicate skills across installed skill directories, validates skill quality, and ensures compliance with the Agent Skills specification. It helps maintain a clean, efficient skill library by identifying redundant skills and specification violations.

## When to Use

Use this skill when:

- Auditing skills for duplicates across directories
- Validating skill quality and specification compliance
- The user mentions "duplicate skills", "skill audit", or "skill validation"
- Cleaning up skill installations
- Preparing skill directories for production use
- Troubleshooting skill conflicts

## Skill Directory Locations

Skills can be installed in multiple locations depending on the agent platform:

### Common Locations

| Platform         | Default Location                   |
| ---------------- | ---------------------------------- |
| Claude Code      | `~/.claude/skills/`                |
| OpenCode         | `.opencode/skills/`                |
| Cursor           | `.cursor/skills/`                  |
| Generic          | `~/.agents/skills/`                |
| Project-specific | `./skills/` or `./.agents/skills/` |

### Finding Skill Directories

```bash
# Find all skill directories in common locations
find ~ -type d -name "skills" 2>/dev/null | grep -E "(claude|opencode|cursor|agents)"

# Find skills in current project
find . -type d -name "skills" 2>/dev/null

# List installed skills
ls -la ~/.claude/skills/ 2>/dev/null
ls -la .opencode/skills/ 2>/dev/null
ls -la .cursor/skills/ 2>/dev/null
```

## Duplicate Detection

### What Constitutes a Duplicate

Skills are considered duplicates if they have:

- **Same name** in frontmatter (e.g., both have `name: conventional-commits`)
- **Same purpose** with different names (e.g., `git-commits` and `conventional-commits`)
- **Overlapping functionality** (e.g., `api-docs` and `documentation-generator`)

### Detection Process

#### Step 1: Scan All Skill Directories

```bash
# Scan all skill directories
for dir in ~/.claude/skills .opencode/skills .cursor/skills; do
  if [ -d "$dir" ]; then
    echo "Scanning $dir..."
    find "$dir" -name "SKILL.md" -type f
  fi
done
```

#### Step 2: Extract Skill Metadata

For each SKILL.md file, extract:

- `name` from frontmatter
- `description` from frontmatter
- Directory location
- File size and last modified date

```bash
# Extract skill names
for file in $(find ~/.claude/skills -name "SKILL.md"); do
  name=$(grep "^name:" "$file" | cut -d' ' -f2)
  echo "$name: $file"
done | sort
```

#### Step 3: Identify Duplicates

**Exact Duplicates (Same Name):**

```bash
# Find skills with same name
for file in $(find ~/.claude/skills .opencode/skills -name "SKILL.md" 2>/dev/null); do
  grep "^name:" "$file"
done | sort | uniq -c | awk '$1 > 1 {print $2}'
```

**Functional Duplicates (Same Purpose):**

- Compare descriptions for similar keywords
- Check for overlapping functionality
- Identify skills that solve the same problem

#### Step 4: Analyze Duplicate Impact

For each duplicate found, analyze:

- **Version differences**: Are they different versions of the same skill?
- **Quality differences**: Which one is better maintained?
- **Location**: Which directory is the primary installation?
- **Usage frequency**: Which one is actually being used?

## Duplicate Resolution

### Resolution Strategies

#### Strategy 1: Keep Most Recent

Keep the most recently updated version:

```bash
# Find modification dates
ls -lt ~/.claude/skills/conventional-commits/SKILL.md
ls -lt .opencode/skills/conventional-commits/SKILL.md

# Remove older version
rm -rf .opencode/skills/conventional-commitals
```

#### Strategy 2: Keep Highest Quality

Evaluate quality based on:

- Completeness of documentation
- Code examples and clarity
- Specification compliance
- Maintenance history

```bash
# Compare file sizes (rough quality indicator)
wc -l ~/.claude/skills/skill-name/SKILL.md
wc -l .opencode/skills/skill-name/SKILL.md
```

#### Strategy 3: Merge Improvements

If both versions have unique improvements:

1. Create a backup of both versions
2. Manually merge improvements
3. Test the merged version
4. Remove the inferior versions

```bash
# Backup
cp -r ~/.claude/skills/skill-name ~/.claude/skills/skill-name.backup
cp -r .opencode/skills/skill-name .opencode/skills/skill-name.backup

# Merge manually in editor
# Then remove backups after verification
```

#### Strategy 4: Keep Project-Specific

If one version is project-specific and another is global:

- Keep project-specific in project directory
- Keep global in user directory
- Document the difference in project README

### Resolution Workflow

```
For each duplicate found:
  1. Identify all instances
  2. Compare versions and quality
  3. Determine resolution strategy
  4. Document decision
  5. Remove inferior versions
  6. Verify remaining skill works
  7. Update any references
```

## Specification Compliance

### Validation Checks

#### YAML Frontmatter

Check that each SKILL.md has valid frontmatter:

```bash
# Validate frontmatter exists
for file in $(find ~/.claude/skills -name "SKILL.md"); do
  if ! head -1 "$file" | grep -q "^---$"; then
    echo "ERROR: Missing frontmatter in $file"
  fi
done

# Validate required fields
for file in $(find ~/.claude/skills -name "SKILL.md"); do
  if ! grep -q "^name:" "$file"; then
    echo "ERROR: Missing 'name' in $file"
  fi
  if ! grep -q "^description:" "$file"; then
    echo "ERROR: Missing 'description' in $file"
  fi
done
```

#### Name Validation

- 1-64 characters
- Lowercase letters, numbers, hyphens only
- No consecutive hyphens
- Must match directory name

```bash
# Validate skill names
for file in $(find ~/.claude/skills -name "SKILL.md"); do
  name=$(grep "^name:" "$file" | cut -d' ' -f2)
  dir=$(basename $(dirname "$file"))

  # Check length
  if [ ${#name} -gt 64 ]; then
    echo "ERROR: Name too long in $file"
  fi

  # Check characters
  if ! echo "$name" | grep -qE "^[a-z0-9-]+$"; then
    echo "ERROR: Invalid characters in name: $name"
  fi

  # Check directory match
  if [ "$name" != "$dir" ]; then
    echo "ERROR: Name '$name' doesn't match directory '$dir'"
  fi
done
```

#### Description Validation

- Max 1024 characters
- Starts with "Use when..."
- Describes when to use (not what it does)

```bash
# Validate descriptions
for file in $(find ~/.claude/skills -name "SKILL.md"); do
  desc=$(grep "^description:" "$file" | cut -d' ' -f2-)

  # Check length
  if [ ${#desc} -gt 1024 ]; then
    echo "ERROR: Description too long in $file"
  fi

  # Check format
  if ! echo "$desc" | grep -q "^Use when"; then
    echo "WARNING: Description should start with 'Use when' in $file"
  fi
done
```

### Quality Checks

#### Documentation Completeness

Check for required sections:

```bash
# Check for required sections
for file in $(find ~/.claude/skills -name "SKILL.md"); do
  if ! grep -q "^## Overview" "$file"; then
    echo "WARNING: Missing 'Overview' section in $file"
  fi
  if ! grep -q "^## When to Use" "$file"; then
    echo "WARNING: Missing 'When to Use' section in $file"
  fi
done
```

#### File Size

Check for overly large files:

```bash
# Check file size
for file in $(find ~/.claude/skills -name "SKILL.md"); do
  lines=$(wc -l < "$file")
  if [ $lines -gt 500 ]; then
    echo "WARNING: $file is $lines lines (recommended: <500)"
  fi
done
```

#### Code Examples

Check for code examples:

````bash
# Check for code examples
for file in $(find ~/.claude/skills -name "SKILL.md"); do
  if ! grep -q '```' "$file"; then
    echo "WARNING: No code examples found in $file"
  fi
done
````

## Audit Report

### Generate Audit Report

Create a comprehensive audit report:

```markdown
# Skill Audit Report

**Date:** [timestamp]
**Directories Audited:** [list]

## Summary

- Total skills found: [count]
- Duplicate skills: [count]
- Specification violations: [count]
- Quality warnings: [count]

## Duplicate Skills

### [skill-name]

**Instances:**

- ~/.claude/skills/[skill-name]/SKILL.md
- .opencode/skills/[skill-name]/SKILL.md

**Recommendation:** [resolution strategy]

## Specification Violations

### [skill-name]

**Issue:** [description]
**Location:** [file path]
**Fix:** [recommended action]

## Quality Warnings

### [skill-name]

**Issue:** [description]
**Location:** [file path]
**Recommendation:** [improvement]

## Recommended Actions

1. [Action 1]
2. [Action 2]
3. [Action 3]
```

### Automated Audit Script

```bash
#!/bin/bash
# skill-audit.sh - Audit skills for duplicates and compliance

echo "# Skill Audit Report"
echo "**Date:** $(date)"
echo ""

# Find all skill directories
SKILL_DIRS=$(find ~ -type d -name "skills" 2>/dev/null | grep -E "(claude|opencode|cursor|agents)")

echo "## Directories Audited"
echo "$SKILL_DIRS"
echo ""

# Count total skills
TOTAL_SKILLS=$(find $SKILL_DIRS -name "SKILL.md" 2>/dev/null | wc -l)
echo "## Summary"
echo "- Total skills found: $TOTAL_SKILLS"
echo ""

# Find duplicates
echo "## Duplicate Skills"
find $SKILL_DIRS -name "SKILL.md" -exec grep "^name:" {} \; 2>/dev/null | \
  sort | uniq -c | awk '$1 > 1 {print "- " $2 " (" $1 " instances)"}'
echo ""

# Check specification compliance
echo "## Specification Violations"
for file in $(find $SKILL_DIRS -name "SKILL.md" 2>/dev/null); do
  # Check frontmatter
  if ! head -1 "$file" | grep -q "^---$"; then
    echo "- Missing frontmatter: $file"
  fi

  # Check name
  if ! grep -q "^name:" "$file"; then
    echo "- Missing 'name': $file"
  fi

  # Check description
  if ! grep -q "^description:" "$file"; then
    echo "- Missing 'description': $file"
  fi
done
echo ""

# Quality warnings
echo "## Quality Warnings"
for file in $(find $SKILL_DIRS -name "SKILL.md" 2>/dev/null); do
  lines=$(wc -l < "$file")
  if [ $lines -gt 500 ]; then
    echo "- File too large ($lines lines): $file"
  fi
done
```

## Cleanup Commands

### Remove Duplicate Skills

```bash
# Remove specific duplicate
rm -rf .opencode/skills/conventional-commits

# Remove all duplicates from specific directory
rm -rf .opencode/skills/*

# Keep only skills in primary directory
mv ~/.claude/skills/* /tmp/skills-backup/
rm -rf ~/.claude/skills/*
mv /tmp/skills-backup/* ~/.claude/skills/
```

### Fix Specification Violations

```bash
# Fix missing frontmatter
for file in $(find ~/.claude/skills -name "SKILL.md"); do
  if ! head -1 "$file" | grep -q "^---$"; then
    # Add frontmatter
    sed -i '1i ---\nname: [skill-name]\ndescription: [description]\n---' "$file"
  fi
done
```

## Best Practices

### Skill Organization

1. **Single Source of Truth**: Keep skills in one primary directory
2. **Symlink for Projects**: Use symlinks for project-specific access
3. **Version Control**: Track skills in git repository
4. **Regular Audits**: Run audits monthly or after major changes

### Duplicate Prevention

1. **Document Installations**: Keep a record of where skills are installed
2. **Use Package Managers**: Prefer skill package managers over manual installation
3. **Centralized Storage**: Use a central skill repository
4. **Cleanup After Testing**: Remove test installations promptly

### Quality Maintenance

1. **Regular Validation**: Run specification checks regularly
2. **Update Descriptions**: Keep descriptions current and accurate
3. **Review Examples**: Ensure code examples work correctly
4. **Check Links**: Verify external links are valid

## Quick Reference

| Task                       | Command                                                                                  |
| -------------------------- | ---------------------------------------------------------------------------------------- |
| Find all skill directories | `find ~ -type d -name "skills" 2>/dev/null`                                              |
| List all skills            | `find ~/.claude/skills -name "SKILL.md"`                                                 |
| Find duplicate names       | `find ... -name "SKILL.md" -exec grep "^name:" {} \; \| sort \| uniq -c \| awk '$1 > 1'` |
| Validate frontmatter       | `grep "^name:" SKILL.md && grep "^description:" SKILL.md`                                |
| Check file size            | `wc -l SKILL.md`                                                                         |
| Remove duplicate           | `rm -rf .opencode/skills/skill-name`                                                     |

## The Bottom Line

Skill Auditor maintains a clean, efficient skill library by finding duplicates, validating specification compliance, and ensuring quality. Regular audits prevent skill conflicts and keep your skill library organized and performant.

**Key Actions:**

- Audit skills regularly (monthly or after changes)
- Remove duplicates promptly
- Fix specification violations immediately
- Maintain single source of truth
- Document skill installations
