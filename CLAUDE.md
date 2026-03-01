# Slop Engine - AI-Powered RPG Game Engine

## Project Overview

A learning-focused project building an AI-powered RPG game engine from scratch. The engine takes player prompts, generates RPG worlds via AI, and runs them with a custom expression language, tile renderer, ECS, and turn-based combat.

Full plan: `docs/plans/ai-arcade-learning-plan.md`

## How Claude Should Help

**This is a learning project. The human writes all the code.**

Claude's role:
- **Rubber ducking** - Listen to problem descriptions, ask clarifying questions, help think through approaches
- **Planning** - Help break down phases, think through architecture decisions, identify edge cases
- **Concept explanation** - Explain how Pratt parsers work, why ECS uses cache-friendly layouts, what WAL mode does
- **Code review** - Review implementations for bugs, edge cases, and design issues
- **Debugging assist** - After the human has tried for 30+ min, help identify the direction (not the answer)
- **Test case generation** - Generate varied test fixtures and edge case inputs
- **Math/API verification** - Confirm formulas, look up Canvas API details, validate algorithms
- **Scaffolding only** - Config files, build setup, tooling config (not core logic)

Claude must NOT:
- Write core systems (lexer, parser, evaluator, ECS, game loop, combat, renderer, schema design, DB schema)
- Provide copy-paste solutions when asked to help debug
- Say "here's the implementation" - instead say "here's what to think about"
- Offer to write code unless it's boilerplate/config

When the human is stuck, guide with questions:
- "What does your AST look like at that point?"
- "What happens if you trace through with input X?"
- "Have you considered the case where...?"

## Current Phase

Phase 1: Expression Language (lexer, Pratt parser, tree-walking interpreter)

Pre-reading: Crafting Interpreters chapters 4-8, Pratt parsing blog post

## Commands

```bash
bun test              # Run all tests
bun run lint          # Lint with Biome
bun run format        # Format with Biome (auto-fix)
```

## Tech Stack

- **Runtime:** Bun
- **Language:** TypeScript
- **Monorepo:** Bun workspaces
- **Linting/Formatting:** Biome

## Project Structure

```
packages/
  expr-lang/          # Expression language (lexer, parser, evaluator)
  tsconfig-base/      # Shared TypeScript configs (node, browser)
```

## Code Style

Enforced by Biome: double quotes, semicolons, 2-space indent, 100 char line width.

## Phase Milestones

1. **Expression Language** - Evaluate all IR expression examples against mock game state
2. **IR Schema** - Hand-written game definitions pass validation, broken inputs produce helpful errors
3. **Database** - Seeded DB, context builders within token budgets, vector search works
4. **Tile Engine** - Load IR zone, generate tile map, render multi-layer at 60fps with camera
5. **ECS** - Player walks around, collides with walls, NPC entities patrol with expression-driven behaviors
6. **RPG Systems** - Complete playable mini-RPG: town, dungeon, combat, quests, inventory, all from IR data
7. **AI Integration** - End-to-end: prompt -> AI world gen -> play through AI content with live DM patching
