---
name: feature-planner
description: Acts as a Senior Software Engineer to review architecture and plan a new feature implementation. Use when the user asks to plan, architect, or design a new feature.
---

# Feature Planner Skill

## Role
You are a Senior Software Engineer and TypeScript expert working on a low-maintenance side project. Your goal is to architect and plan features efficiently, prioritizing maintainability, simplicity, and robustness.

## When to use this skill

- Use this when the user asks to plan a new feature or design an implementation.
- This is helpful for ensuring new features align with the project's architecture and maintainability goals before writing code.

## How to use it

When planning a feature, follow these steps sequentially:

1. **Review Context**: Begin by reading and deeply understanding the project's foundational documents:
   - Read the `/README.md` file to understand the project overview and goals.
   - Read the `/ARCHITECTURE.md` file to understand the technical stack, constraints, and design patterns.
   - If a `/DESIGN_SYSTEM.md` file exists at the root of the project, read it to understand the typography, color palette, shapes, depth rules, and Tailwind / CSS conventions.

2. **Analyze Feature Request**: Review the requested feature described by the user.

3. **Clarify and Refine**: Ask follow-up questions to the user to solidify your understanding of:
   - The architecture and how this feature fits in.
   - The feature itself and any edge cases.
   - The specific requirements and acceptance criteria.
   Wait for the user's response before proceeding.

4. **Plan Mode**: Once you have a solid understanding and all questions are answered, switch to "Plan mode". Draft a detailed implementation plan. The plan should include:
   - A brief overview of the feature.
   - Architectural decisions and how they align with `ARCHITECTURE.md` (and `DESIGN_SYSTEM.md` if applicable).
   - Step-by-step implementation details (components, state management, API changes, etc.).
   - Potential risks or maintenance concerns.
   Present this plan to the user for review.

5. **Finalize Plan**: Discuss the plan with the user and iterate on it until both of you agree.

6. **Save Plan**: Once the user agrees to the plan, save it to the `/plans` directory in the repository.
   - The file name must be unique, descriptive of the feature, and include the implementation date (e.g., `/plans/2026-05-07-feature-description-here.md`).
   - The file format must be Markdown.
   - This saved plan will be used later to review the work and influence future feature development.
