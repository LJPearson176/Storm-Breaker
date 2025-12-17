# AGENTS.md

## 1. The Prime Directive
You are an expert autonomous software engineer working in a **Spec-Driven Development (SDD)** environment.
**Rule #1:** The Code is *not* the source of truth. The Specification is the source of truth.
**Rule #2:** You must never generate implementation code without an approved Technical Plan located in a live spec file.

---

## 2. Directory Structure & Context
We organize our work using a strict directory state machine. You must analyze these folders to understand your current task.

* **`specs/active/`**: Contains Markdown files for features or fixes currently in progress. This is your primary workspace.
* **`specs/archive/`**: Contains completed specifications. Read these only for historical context or to understand previous decisions.

---

## 3. The Spec-Driven Workflow
For every request, determine which stage the task is in and follow the corresponding steps.

### Stage 1: Specification (The "What")
*Trigger: User asks for a new feature or fix.*
1.  **Search**: Check `specs/active/` for an existing file regarding this topic.
2.  **Create**: If none exists, create a new file: `specs/active/XXX-feature-name.md` (where XXX is a sequential number).
3.  **Content**: The spec must include:
    * **Context**: Why are we doing this?
    * **User Stories**: Who is this for?
    * **Acceptance Criteria**: Checkbox list of verification steps.
    * **Legacy Impact**: Which existing legacy modules will be touched?

### Stage 2: Technical Planning (The "How")
*Trigger: Spec is written, but code is not started.*
1.  **Analyze**: Read the legacy code relevant to the request. Identify risks (e.g., tight coupling, lack of tests).
2.  **Update Spec**: Append a `## Technical Plan` section to the active spec file.
3.  **Detail**:
    * List specific files to be created or modified.
    * Define data model changes or new interfaces.
    * **Strategy**: Explain how you will isolate your changes from the legacy "spaghetti."
4.  **Pause**: Ask the user to review and approve the Technical Plan.

### Stage 3: Execution (The "Work")
*Trigger: User approves the Technical Plan.*
1.  **TDD First**: Write a test case that fails (or mimics the bug) based on the Acceptance Criteria.
2.  **Implement**: Write the code to satisfy the test and the plan.
3.  **Verification**: Ensure all tests pass. Do not break existing legacy functionality.

### Stage 4: Completion (The "Cleanup")
*Trigger: Feature is verified and accepted.*
1.  **Move**: Move the markdown file from `specs/active/` to `specs/archive/`.
2.  **Update**: Update `GLOSSARY.md` or `ARCHITECTURE.md` if you introduced new domain terms or patterns.

---

## 4. Legacy Code Guidelines
This repository contains legacy code. Adhere to these safety rails:
* **Observation**: If you see code that is confusing or undocumented, add comments explaining what it does as you read it.
* **Containment**: Prefer writing new, clean modules over heavily modifying complex legacy files. Use adapter patterns if necessary.
* **Refactoring**: Do not refactor legacy code strictly for aesthetics. Only refactor if it is required to make the new Spec work or if explicitly requested.

---

## 5. Technical Context
* **Stack**: Python 3 / PHP
* **Testing Framework**: PyTest
* **Linting/Formatting**: Black / Flake8
