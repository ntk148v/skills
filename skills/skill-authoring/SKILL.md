---
name: skill-authoring
description: Use when creating, editing, auditing, or validating Agent Skills. Covers TDD-based creation and specification compliance auditing.
---

# Skill Authoring

## Overview

Skill Authoring is the complete lifecycle of creating and maintaining high-quality Agent Skills. It combines Test-Driven Development (TDD) for creation with rigorous auditing for specification compliance and quality.

**Core principle:** If you didn't watch an agent fail without the skill, you don't know if the skill teaches the right thing.

## When to Use

- Creating new Agent Skills
- Editing or refactoring existing skills
- Auditing skills for duplicates or spec violations
- Verifying skills work before deployment

---

## 1. Creation (TDD for Skills)

Writing skills is TDD applied to process documentation.

| TDD Concept  | Skill Creation                                                               |
| :----------- | :--------------------------------------------------------------------------- |
| **RED**      | Run baseline scenario WITHOUT skill. Document exact failure/rationalization. |
| **GREEN**    | Write minimal skill addressing those specific violations. Watch it pass.     |
| **REFACTOR** | Close loopholes and consolidate logic while maintaining compliance.          |

### The Iron Law

**NO SKILL WITHOUT A FAILING TEST FIRST.** Delete any untested skill code.

---

## 2. Auditing & Compliance

Maintain a clean, efficient library by identifying redundant skills and specification violations.

### Specification Requirements

- **YAML Frontmatter**: `name` (kebab-case, max 64 chars) and `description` (max 1024 chars).
- **Description**: Must start with "Use when..." and describe triggers, not workflow.
- **Structure**: Required `## Overview` and `## When to Use` sections.
- **Directory**: Directory name must match the skill name in frontmatter.

### Duplicate Detection

Skills are duplicates if they share the **same name**, **same purpose**, or **overlapping functionality**.

- **Resolution**: Keep the highest quality version, merge unique improvements, and remove inferior versions.

---

## 3. Implementation Checklist

- [ ] **RED**: Baseline failure documented?
- [ ] **Frontmatter**: Valid `name` and `description`?
- [ ] **Triggers**: Description uses "Use when..."?
- [ ] **Discovery**: Keywords included for search?
- [ ] **Efficiency**: Under 500 words (200 for frequent skills)?
- [ ] **Verification**: Pass scenario WITH skill?
- [ ] **Audit**: No duplicates or spec violations?

## Quick Reference: Validating a Skill

```bash
# Check name matches directory
name=$(grep "^name:" SKILL.md | cut -d' ' -f2)
[ "$name" == "$(basename $PWD)" ] || echo "Error: Name mismatch"

# Check for required sections
grep -q "^## Overview" SKILL.md && grep -q "^## When to Use" SKILL.md || echo "Error: Missing sections"
```
