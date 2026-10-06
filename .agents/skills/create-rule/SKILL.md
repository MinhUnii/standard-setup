---
name: create-rule
description: Scaffolds, generates, and validates agent rules and workspace instructions (AGENTS.md, GEMINI.md, and .agents/rules/*.md). Use when asked to create, draft, configure, or optimize rules, guidelines, coding standards, or triggers for the agent.
---

# Create Rule

This skill guides the creation, configuration, and validation of workspace rules in the Antigravity agent system. Rules enforce coding guidelines, architectural constraints, API conventions, and operational workflows.

---

## Standard Operating Procedure (SOP)

Follow these sequential steps when authoring a new rule:

1. **Determine Rule Scope and Format**:
   - For **general, directory-wide, always-active rules**: Use `AGENTS.md` (or `GEMINI.md`) at the repository root or within a specific subdirectory.
   - For **modular, conditional, or trigger-based rules**: Create dedicated markdown files in `.agents/rules/<rule-name>.md`.

2. **Select the Appropriate Activation Trigger**:
   - Evaluate memory and token efficiency to choose the right `trigger` mode: `model_decision`, `glob`, `always_on`, or `manual`.

3. **Format Frontmatter (Mandatory for `.agents/rules/*.md`)**:
   - Ensure the YAML frontmatter block contains all required fields based on the chosen trigger.

4. **Author Rule Content**:
   - Write actionable, clear, and unambiguous constraints, guidelines, and code patterns.
   - Use advanced inclusions (`@[label](path)` for content inlining, `@path` for canonical path references) when referencing external files.

5. **Verify Size and Token Limits**:
   - Check that the rule file stays well under the 24 KB (24,000 bytes) per-file limit.
   - Keep `always_on` rules concise to stay within the 20,000-token shared budget.

6. **Validate Placement**:
   - Ensure rule files are placed directly at the top level of `.agents/rules/` (flat scanning), or registered explicitly in `.agents/rules.json` if nested in subdirectories.

---

## 1. Storage Locations & Naming Conventions

There are two primary ways to store rules:

### A. Directory-Level Rule Files (`AGENTS.md` / `GEMINI.md`)
* **Location**: Workspace root or any subdirectory (e.g., `./AGENTS.md`, `src/frontend/AGENTS.md`).
* **Behavior**: Always active (`always_on`) for that directory and its child directories.
* **Frontmatter**: **Do NOT include YAML frontmatter**. These files are read as pure Markdown.

### B. Modular Rule Files (`.agents/rules/*.md`)
* **Location**: `.agents/rules/<rule-name>.md` (e.g., `.agents/rules/typescript.md`, `.agents/rules/api-conventions.md`).
* **Naming**: Lowercase, hyphen-delimited descriptive names ending in `.md`.
* **Flat Directory Scanning**:
  > [!IMPORTANT]
  > The discovery engine scans **only the top-level `.md` files** directly under `.agents/rules/`. Subdirectory files (e.g., `.agents/rules/frontend/react.md`) are ignored by default unless explicitly declared in `.agents/rules.json`.

---

## 2. Frontmatter Specifications

Every `.agents/rules/*.md` file **must** start with valid YAML frontmatter between `---` delimiters.

> [!WARNING]
> Missing, malformed, or invalid frontmatter will cause the rule to be silently ignored by the discovery engine.

### Frontmatter Schema

| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `trigger` | `string` | **Yes** | Activation mode: `always_on`, `model_decision`, `glob`, or `manual`. |
| `description` | `string` | **Yes** (for `model_decision`) / Recommended | Short summary explaining when the rule applies. Used by the agent to determine relevance. |
| `globs` (or `glob`) | `string` | **Yes** (for `glob`) | Comma-separated file match patterns enclosed in quotes (e.g., `"*.ts, *.tsx"`). |

---

## 3. Activation Modes & Decision Guide

Choose the trigger based on context and token budget:

```
                      Is this rule needed on every single turn?
                                   /             \
                                 YES              NO
                                 /                 \
                     Use `always_on`        Does it apply only to specific file types?
               (or place in AGENTS.md)             /             \
                                                 YES              NO
                                                 /                 \
                                           Use `glob`       Is it run only on explicit user demand?
                                     (e.g., "*.proto")             /             \
                                                                 YES              NO
                                                                 /                 \
                                                           Use `manual`       Use `model_decision`
                                                       (e.g., Release checklist) (Progressive Disclosure)
```

### Detailed Trigger Breakdown

1. **`model_decision` (Recommended for comprehensive/detailed guidelines)**:
   - **How it works**: Leverages **Progressive Disclosure**. The agent only loads the rule's `description` into initial context. When the user task matches the description, the agent fetches and reads the full rule content on demand.
   - **Best for**: Architectural patterns, API styleguides, testing standards, complex framework conventions.

2. **`glob`**:
   - **How it works**: Automatically loaded into context only when the agent touches, opens, or edits files matching the glob pattern.
   - **Best for**: Language-specific or extension-specific rules (e.g., Protobuf, GraphQL, SQL, Dockerfiles).
   - **Syntax**: `globs: "*.proto"` or `globs: "*.ts, *.tsx"`.

3. **`always_on`**:
   - **How it works**: Injected into the agent's context window on every turn unconditionally.
   - **Best for**: Critical non-negotiable workspace-wide guardrails.
   - *Tip*: If an always-on rule applies to a specific folder, consider putting it in `AGENTS.md` directly.

4. **`manual`**:
   - **How it works**: Never automatically loaded. Loaded only when the user explicitly references the rule in the chat prompt using `@rule-name`.
   - **Best for**: Pre-release checklists, security audits, database migration runbooks, quarterly review tasks.

---

## 4. Size & Token Limits

Keep rules lean and modular to avoid automated truncation or demotion:

* **Per-File Size Limit (24 KB / 24,000 bytes)**:
  - Each individual rule file is capped at 24,000 bytes (evaluated after expanding all `@[label](path)` inclusions).
  - Files exceeding 24 KB will be automatically truncated along line boundaries.
* **Aggregate Rules Budget (20,000 tokens)**:
  - All `always_on` rules share a 20,000-token context budget.
  - If the total exceeds 20,000 tokens, the system automatically demotes the largest rules into file path pointers (links) so the agent reads them on demand rather than pre-loading them.

---

## 5. Advanced File Inclusions (Transclusion vs Reference)

When referencing other project files within a rule, use the appropriate syntax:

### A. Full Inlining / Transclusion (`@[label](path)`)
* Injects the raw file contents directly into the rule body during loading.
* The system automatically strips any YAML frontmatter from the referenced source file.
* **Example**:
  ```markdown
  See standard schema definition:
  @[Schema Definition](src/types/schema.json)
  ```

### B. Canonical Path Reference (`@filename` or `@path`)
* Supplies the agent with a canonical file path pointer without inlining its content.
* The agent can inspect the file only when needed using standard read tools.
* **Example**:
  ```markdown
  Refer to configuration template at @config/base.yaml when creating new environments.
  ```

---

## Rule Templates

### Template 1: `model_decision` Rule
```markdown
---
trigger: model_decision
description: Enforces error handling and logging standards across backend services. Use when writing backend API handlers, error classes, or service loggers.
---

# Backend Error Handling & Logging Guidelines

## Error Handling Standards
* Always return typed custom error instances (e.g., `AppError`, `NotFoundError`).
* Never swallow errors in catch blocks without logging stack traces.

## Structured Logging
* Use JSON structured logging with `trace_id` and `timestamp`.
* Redact sensitive information (passwords, tokens, PII).
```

### Template 2: `glob` Rule
```markdown
---
trigger: glob
globs: "*.ts, *.tsx"
description: TypeScript coding conventions and strict type safety standards.
---

# TypeScript Guidelines

* Do not use `any`; use `unknown` with type narrowing.
* Prefer `interface` over `type` for object signatures.
* Enable explicit return types on exported functions.
```

### Template 3: `manual` Rule
```markdown
---
trigger: manual
description: Pre-release verification checklist before deploying to production.
---

# Production Release Checklist

1. [ ] Run full test suite: `npm test`
2. [ ] Verify database migrations are backwards-compatible
3. [ ] Check environment variables in `.env.example`
4. [ ] Build assets without warnings: `npm run build`
```

### Template 4: `AGENTS.md` (Directory-Level)
```markdown
# Repository Guidelines

* Maintain test coverage above 80%.
* Use conventional commits (`feat:`, `fix:`, `docs:`, `refactor:`).
* Follow repository directory structure conventions.
```

---

## Review Checklist

Before finalizing any rule file, verify:
* [ ] Is the file in the right location (`AGENTS.md` for folder-wide, `.agents/rules/*.md` for modular rules)?
* [ ] Is the file at the root level of `.agents/rules/` (avoiding unstructured subdirectories unless registered in `.agents/rules.json`)?
* [ ] Does `.agents/rules/*.md` include valid YAML frontmatter with a valid `trigger` (`always_on`, `model_decision`, `glob`, `manual`)?
* [ ] If `trigger: model_decision`, is `description` clear, context-rich, and written in third person?
* [ ] If `trigger: glob`, is `globs` defined and enclosed in quotes?
* [ ] Is the file size well below the 24 KB limit?
* [ ] Are file includes using `@[label](path)` for inlining or `@path` for pointers?
