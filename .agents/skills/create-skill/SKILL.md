---
name: create-skill
description: Scaffolds and generates new agent skills adhering to standard open standard conventions. Use when the user requests to create, build, draft, or scaffold a new skill package.
---

# Create Skill

When tasked with creating a new skill, your objective is to generate a well-structured skill directory containing a compliant `SKILL.md` and any necessary auxiliary folders. 

## Execution Steps

Follow these sequential steps when generating a new skill:

1. **Gather Requirements:** If the user's request is too vague, ask clarifying questions to determine the skill's exact purpose, expected inputs, and desired outputs.
2. **Determine Scope (Single Responsibility):** Ensure the requested skill does only one thing well. If the user asks for a "do everything" skill, advise them to split it into multiple focused skills.
3. **Draft the Directory Structure:** Propose the standard skill directory layout based on the user's needs (including optional `scripts/`, `examples/`, or `resources/` folders).
4. **Generate the `SKILL.md`:** Write the complete markdown file utilizing the best practices and templates outlined below.

## Frontmatter & Description Guidelines

Every `SKILL.md` you create MUST begin with YAML frontmatter. The `description` field is critical for skill discovery.
* Write the description in the **third person**.
* Include highly specific keywords so the agent knows exactly when to trigger it.
* **Good Example:** `description: Generates unit tests for Python code using pytest conventions. Use when asked to test Python scripts.`
* **Bad Example:** `description: A skill to help you test things.`

## Standard SKILL.md Template

Use the following template structure when generating the `SKILL.md` for the user:

` ` `markdown
---
name: [skill-name-lowercase-hyphenated]
description: [Third-person description outlining exactly what it does and when to use it]
---

# [Skill Title]

[Brief introductory paragraph explaining the core objective of the skill.]

## Standard Operating Procedure (SOP)
[Numbered list of explicit, step-by-step instructions the agent must follow.]
1. Step one...
2. Step two...

## Checklists & Best Practices
[A checklist of rules, style conventions, or boundaries the agent must respect.]
* **Criteria 1:** [Detail]
* **Criteria 2:** [Detail]

## Decision Tree (If Applicable)
[For complex tasks, instruct the agent how to handle different edge cases or scenarios.]
* **If X occurs:** Do [Action A].
* **If Y occurs:** Do [Action B].

## Script Handling (If Applicable)
[If the skill uses external scripts, explicitly instruct the agent to run `script_name --help` first, treating the script as a black box rather than reading its source code.]
` ` `

## Review Checklist

Before finalizing the skill creation for the user, verify the following:
* [ ] Does the `SKILL.md` have valid YAML frontmatter?
* [ ] Is the `name` field lowercase with hyphens?
* [ ] Is the `description` in the third person and context-rich?
* [ ] Does the skill follow the Single Responsibility Principle?
* [ ] Are the instructions explicit and structured logically?