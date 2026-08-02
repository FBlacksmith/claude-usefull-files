---
name: generate-claude-md
description: Generates a lightweight, optimized CLAUDE.md file for the current project using progressive disclosure, with arguments for project name, architecture directives, and coding conventions.
argument-hint: "[project-name] [architecture-directives-path] [coding-conventions-path]"
disable-model-invocation: true
---

# Generate CLAUDE.md Skill

This skill automates the creation of a highly efficient, lightweight `CLAUDE.md` file for any codebase. It enforces the principle of **Progressive Disclosure** (keeping instructions enxuto to preserve the model's instruction budget) and configures the agent for a disciplined, surgical, and test-driven engineering workflow.

## Arguments
- `$0` (or `$ARGUMENTS[0]`): **Project Name** (Optional. If not provided, falls back to the current folder name using shell execution).
- `$1` (or `$ARGUMENTS[1]`): **Architecture Directives Path** (Optional. Path to a local file defining architectural standards, e.g., `architecture_directives.md`).
- `$2` (or `$ARGUMENTS[2]`): **Coding Conventions Path** (Optional. Path to a conventions file, e.g., `.editorconfig` or `coding_guidelines.md`).

---

## Instructions for the Agent

When this skill is executed, perform the following steps:

### Step 1: Resolve the Project Name
1. If the first argument `$0` is provided, use it as the project name.
2. If `$0` is empty, run the shell command:
   ```bash
   ! basename $(pwd)
   ```
   Use the output of this command as the fallback project name.

### Step 2: Analyze Architecture and Conventions (If Provided)
1. **Architecture Directives (`$1`):**
   - If `$1` is provided, read its content.
   - Extract the 3 to 5 most critical rules or architectural constraints (e.g., layers, dependency flow, framework versions).
   - Prepare a reference mention: `@path-to-file` (e.g., `@architecture_directives.md`).
2. **Coding Conventions (`$2`):**
   - If `$2` is provided, read its content.
   - Extract the 3 to 5 most critical formatting, naming, or clean code rules (e.g., indentation, naming conventions).
   - Prepare a reference mention: `@path-to-file` (e.g., `@.Net editorconfig`).

### Step 3: Discover Core Commands
Analyze the workspace files (e.g., `package.json`, `.csproj`, `cargo.toml`, `requirements.txt`) to dynamically identify the standard commands for:
- **Build / Compile**
- **Test / Verify**
- **Run / Dev**

Keep this list down to the absolute essentials. Do not dump every script or make command into the `CLAUDE.md` file.

### Step 4: Assemble the CLAUDE.md File
Generate and write a `CLAUDE.md` file in the project's root directory. The file **MUST NOT exceed 100 lines** to protect the instruction budget and avoid context window bloat (preventing "context rot").

Use the exact template structure below:

```markdown
# CLAUDE.md (Project Configuration)

## Project: [PROJECT_NAME]
[Brief 1-sentence description of the project stack discovered in Step 3]

## Core Build & Test Commands
- **Build:** [discovered build command]
- **Test:** [discovered test command]
- **Run:** [discovered run/dev command]

## Progressive Disclosure References
[If architecture directives file was provided:]
- **Architecture Guidelines:** Detailed rules are documented in @[ARCHITECTURE_PATH].
  * Core Rules Summary:
    * [Rule 1 extracted in Step 2]
    * [Rule 2 extracted in Step 2]
    * [Rule 3 extracted in Step 2]

[If coding conventions file was provided:]
- **Coding Conventions:** Clean code standards are defined in @[CONVENTIONS_PATH].
  * Core Standards Summary:
    * [Rule 1 extracted in Step 2]
    * [Rule 2 extracted in Step 2]
    * [Rule 3 extracted in Step 2]

## Behavioral Guidelines for the AI Agent
To maximize token economy and maintain code quality, strictly adhere to the following rules:

1. **Surgery Mode:** Always make targeted, minimal modifications to existing files. Never rewrite intact portions of code unless explicitly instructed.
2. **Concision Policy:** "Answers, not essays". Keep all conversational output brief, technical, and free of preambles or polite filler.
3. **Verification Loop (TDD):** Red/Green refactoring is mandatory. Always run existing tests first to establish a baseline, write tests before implementing new logic, and verify all tests pass after edits.
4. **Prototype-First:** For complex algorithms or business rules, build a quick, throwaway logical spike or prototype to verify the math/logic before writing production code.
```

### Step 5: Final Verification
Verify that `CLAUDE.md` has been successfully created at the project root, is properly formatted, and adheres to the strict limit of under 100 lines. Print a concise summary of the generated file to the console.
