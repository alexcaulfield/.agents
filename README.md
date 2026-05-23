# .agents (Global Configuration)

This repository contains global configurations, guidelines, and skills that help maintain consistency across all software projects. Agents operating in this workspace should adhere to the conventions defined here.

## 1. Project Conventions

Every project repository is expected to maintain standard documents at its root. These files provide context for AI agents during planning, implementation, and debugging.

### Standard Files

| File | Required? | Purpose |
| :--- | :--- | :--- |
| `README.md` | **Yes** | Project overview, goals, setup instructions, and execution commands. |
| `ARCHITECTURE.md` | **Yes** | Technical stack, core design patterns, directory structure, constraints, and integration boundaries. |
| `DESIGN_SYSTEM.md` | *Optional* | Project design system including typography, color palette, shapes, depth rules, and Tailwind / CSS conventions (applicable to front-end projects). |

### Feature Plans (`/plans`)

All projects must support a `/plans` directory at their root:
* **Workflow:** Before writing code for significant changes, agents use the `feature-planner` skill to design the implementation.
* **Storage:** Once approved by the user, plans must be saved to the `/plans` directory.
* **Naming Convention:** Plans must be saved as Markdown files using the format: `YYYY-MM-DD-feature-name.md` (e.g., `/plans/2026-05-23-global-agents-config.md`).

---

## 2. Agent Layering Model

We utilize a hybrid model of **Global Skills** and **Project-Local Rules** to balance project-agnostic workflows with project-specific code constraints.

```mermaid
graph TD
    A[Global Skills] -->|Reusable Workflows| Project[Project Repository]
    B[Local Rules] -->|Specific Constraints| Project
    C[Local Skills] -->|Overrides/Custom Tasks| Project
```

### Global Skills
Global skills are project-agnostic workflows (e.g. `feature-planner`) located in the global `.agents/skills/` directory. They define the *process* of how tasks should be planned, executed, and saved.

### Project-Local Rules
Project-local rules are stored in `<project-root>/.agents/rules/` (e.g. `lit-component-styles.md`). They define *coding constraints* that apply only to that specific repository. 
* These rules are marked with `always_on: true` to ensure that any agent editing files in that repository automatically obeys its specific styles and constraints.

### Project-Local Skills
For workflows unique to a single project, custom skills can be defined in `<project-root>/.agents/skills/`. If a project-local skill has the same name as a global skill, the local skill will take precedence as an override.
