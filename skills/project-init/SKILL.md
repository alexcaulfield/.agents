---
name: project-init
description: Standardizes the bootstrapping of a new side-project repository. Scaffolds required configurations, folder structures, and foundational documents.
---

# Project Initialization Skill

## Role
You are a Staff Software Architect helping the user bootstrap a new side project with an ultra-clean foundation. Your goal is to establish proper directory structure, foundational documentation, and agent configurations from day one.

## When to use this skill
- Use this when the user asks to initialize, setup, bootstrap, or start a new repository or project.
- Use this when a project has just been created but lacks foundational files like `README.md`, `ARCHITECTURE.md`, or the `.agents` config directories.

## How to use it

Perform the following steps sequentially to initialize the project:

### 1. Gather Project Details
Ask the user the following clarifying questions (or inspect the workspace to deduce answers):
- What is the project name and short description?
- What is the primary tech stack (e.g., Lit + TypeScript + Vite, Next.js + React, Python + FastAPI, etc.)?
- Is this a front-end project (requiring a `DESIGN_SYSTEM.md`)?
- What are the main hosting/infrastructure requirements (e.g., Vercel, Netlify, fly.io, etc.)?

### 2. Scaffold Directories
Create the following directories at the root of the project:
- `/plans/` (for keeping approved feature/refactoring plans)
- `.agents/rules/` (for project-local rules that are always on)
- `.agents/skills/` (for project-specific skill overrides)

### 3. Generate Foundational Files

Generate standard files at the root using custom content matching the user's stack:

#### `README.md`
Should contain:
- Project Name and Description
- Prerequisites and Setup Instructions (install commands, node version/`.nvmrc` if applicable)
- Development commands (e.g., run locally, build, test, deploy)
- Project Structure overview

#### `ARCHITECTURE.md`
Should contain:
- **System Overview**: High-level block diagram or explanation of the system.
- **Technology Stack**: Enumeration of tools, frameworks, and packages.
- **Core Architectural Patterns**: Specific design conventions (e.g., client-side routing, state management, security boundaries).
- **Proposed/Actual Directory Structure**: ASCII tree of the project layout.

#### `DESIGN_SYSTEM.md` (Optional - only for front-end projects)
Should contain:
- Typography rules and font pairings.
- Color Palette (defined using hex or HSL variables, e.g., primary, secondary, neutral, accents).
- Layout and depth conventions (spacing scales, shadows, glassmorphism tokens).
- Tailwind CSS or Vanilla CSS conventions.

### 4. Create Initial Local Rule
Create a default, empty template or a relevant local coding standard in `.agents/rules/` based on the tech stack (e.g., styling guides, code formatting guides).

### 5. Final Review
Present the bootstrapped file structure and files to the user for confirmation.
