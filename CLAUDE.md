# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OpenSpec is a CLI tool for spec-driven development with AI coding assistants. It helps humans and AI agree on specifications before implementation by organizing changes as proposals with structured spec deltas.

## Commands

```bash
# Development
pnpm install          # Install dependencies
pnpm run build        # Build TypeScript to dist/
pnpm run dev          # Watch mode for TypeScript compilation
pnpm run dev:cli      # Build and run CLI locally

# Testing
pnpm test             # Run all tests (vitest)
pnpm test:watch       # Watch mode
pnpm test -- test/core/validation.test.ts  # Run single test file

# Linting
pnpm run lint         # ESLint check
```

## Architecture

### Directory Structure
```
src/
├── cli/index.ts           # CLI entry point, command registration (Commander.js)
├── commands/              # Individual command implementations
├── core/
│   ├── configurators/     # AI tool integrations (Claude, Cursor, Cline, etc.)
│   │   └── slash/         # Slash command generators per tool
│   ├── parsers/           # Markdown and change file parsers
│   ├── schemas/           # Zod schemas for Spec, Change, Requirements
│   ├── templates/         # Generated instruction templates
│   ├── validation/        # Spec/change validators
│   └── artifact-graph/    # Experimental workflow system (OPSX commands)
└── utils/                 # File system, interactive prompts, task progress
```

### Key Concepts

**Specs** (`openspec/specs/<capability>/spec.md`): Source-of-truth specifications with Purpose section, Requirements, and Scenarios.

**Changes** (`openspec/changes/<change-id>/`): Proposals containing:
- `proposal.md` - Why and What Changes sections
- `tasks.md` - Implementation checklist
- `specs/<capability>/spec.md` - Delta files with ADDED/MODIFIED/REMOVED/RENAMED Requirements

**Archive**: After completion, changes move to `openspec/changes/archive/YYYY-MM-DD-<name>/`

### Core Flow
1. `InitCommand` scaffolds openspec/ structure and configures AI tool integrations
2. `Validator` validates specs and delta files against Zod schemas
3. `ArchiveCommand` applies spec deltas back to main specs and moves change to archive
4. `MarkdownParser` / `ChangeParser` parse spec and proposal markdown files

### Configurators
Each AI tool has a configurator in `src/core/configurators/` that generates tool-specific instruction files. The `SlashCommandConfigurator` subclasses generate slash command definitions for tools that support them.

## Important Patterns

**Dynamic imports for @inquirer**: Always use `await import('@inquirer/prompts')` instead of static imports to prevent pre-commit hook hangs. See eslint rule and issue #367.

**Validation**: Specs require Purpose and Requirements sections. Changes require Why and What Changes sections. Delta specs must use `## ADDED|MODIFIED|REMOVED|RENAMED Requirements` headers with `### Requirement:` and `#### Scenario:` structure.

**Conventional commits**: Use format `type(scope): subject` for commits.

## OpenSpec Workflow

When working on this codebase, follow the OpenSpec workflow documented in `openspec/AGENTS.md`:
1. For significant changes, create a proposal in `openspec/changes/<change-id>/`
2. Include spec deltas for any capability changes
3. Run `openspec validate <change-id> --strict` before implementation
4. Archive completed changes with `openspec archive <change-id>`
