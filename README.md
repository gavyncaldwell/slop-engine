# Slop Engine

An AI-powered RPG game engine built from scratch. Type a prompt, get a generated RPG world, and play through it — complete with a custom expression language, tile renderer, ECS, and turn-based combat.

## Why

I'm a frontend engineer with 10 years of React and 4 years of TypeScript. This project fills the gaps: compilers, databases, game architecture, canvas rendering, server-side development, and systems design. The engineers AI can't replace are the ones who understand systems end-to-end.

## The Plan

The engine is built in 7 phases, each teaching a different domain:

1. **Expression Language** — Custom lexer, Pratt parser, tree-walking interpreter. Evaluates game logic expressions like `if(self.hp < 5, flee(self), attack(self, player))` against live game state.

2. **IR Schema** — An intermediate representation for game definitions, validated with Zod. The contract between AI output and the engine.

3. **Database** — SQLite for game state, vector embeddings for semantic search, context builders that assemble AI prompts within token budgets.

4. **Tile Engine** — Canvas 2D renderer with sprite sheets, multi-layer tile maps, camera system, and map generation from IR zone definitions.

5. **ECS** — Entity Component System with collision detection, input handling, and expression-driven NPC behaviors via state machines.

6. **RPG Systems** — Turn-based combat, dialogue trees, inventory, quests, progression — all driven by IR data, rendered on canvas without a framework.

7. **AI Integration** — Multi-tier prompt system: Tier 1 generates worlds, Tier 2 generates zones on demand, Tier 3 patches the world during play as a live DM.

Full plan with pre-reading lists, build sequences, and milestones: [`docs/plans/ai-arcade-learning-plan.md`](docs/plans/ai-arcade-learning-plan.md)

## How AI Is Used

I write all the code. AI (Claude) is used as a learning aid:

- **Rubber ducking** — talking through problems
- **Concept explanations** — how Pratt parsers work, why ECS uses cache-friendly layouts
- **Code review** — spotting bugs and edge cases in my implementations
- **Debugging direction** — after I've been stuck for 30+ minutes, pointing me toward the right area
- **Test case generation** — generating varied inputs and edge cases
- **Scaffolding** — config files, build setup, tooling (not core logic)

AI does not write the lexer, parser, evaluator, ECS, game loop, combat system, renderer, database schema, or any other core system. The learning is the point.

## Tech Stack

- **Runtime:** [Bun](https://bun.sh)
- **Language:** TypeScript
- **Monorepo:** Bun workspaces
- **Linting/Formatting:** [Biome](https://biomejs.dev)

## Getting Started

```bash
bun install
bun test
```

## Current Phase

Phase 1: Expression Language — lexer, Pratt parser, tree-walking interpreter.

## License

MIT
