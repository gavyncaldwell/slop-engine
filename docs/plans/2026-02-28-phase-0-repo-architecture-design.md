# Phase 0: Repo Architecture Design

## Decision: Monorepo with Bun Workspaces

Monorepo using Bun workspaces (`packages/*`). Directory structure is Turborepo-compatible from day one, but Turborepo itself gets added later when build orchestration actually matters.

## Package Structure

```
slop-engine/
├── biome.json
├── package.json                  # workspaces: ["packages/*"]
├── .gitignore
├── packages/
│   ├── tsconfig-base/
│   │   ├── package.json          # @slop-engine/tsconfig
│   │   ├── base.json             # shared compiler options (strict, ESNext)
│   │   ├── node.json             # extends base, Bun/Node types
│   │   └── browser.json          # extends base, DOM lib (unused until Phase 4)
│   └── expr-lang/
│       ├── package.json          # @slop-engine/expr-lang
│       ├── tsconfig.json         # extends @slop-engine/tsconfig/node.json
│       └── src/
│           └── index.ts
├── docs/
│   └── plans/
└── CLAUDE.md
```

New packages added as each phase begins:

| Phase | Package | Config extends |
|-------|---------|---------------|
| 1 | `@slop-engine/expr-lang` | `node.json` |
| 2 | `@slop-engine/ir-schema` | `node.json` |
| 3 | `@slop-engine/database` | `node.json` |
| 4 | `@slop-engine/tile-engine` | `browser.json` |
| 5 | `@slop-engine/ecs` | `node.json` |
| 6 | `@slop-engine/rpg-systems` | `browser.json` |
| 7 | `@slop-engine/ai-integration` | `node.json` |

## TypeScript

Shared config package (`@slop-engine/tsconfig`) with three configs:

- **`base.json`** — strict mode, ESNext target, module resolution, common compiler options
- **`node.json`** — extends base, adds `bun-types`
- **`browser.json`** — extends base, adds `"lib": ["DOM", "ESNext"]`

Each package extends via package name (e.g., `"extends": "@slop-engine/tsconfig/node.json"`), resolved by Bun workspaces.

## Testing

Bun's built-in test runner. Colocated test files (`src/lexer.test.ts` next to `src/lexer.ts`). No extra dependencies.

## Linting & Formatting

Biome. Single `biome.json` at root with default settings. Covers both linting and formatting.

## Root Scripts

- `bun run test` — runs tests across packages
- `bun run lint` — `biome check`
- `bun run format` — `biome format --write`

## Explicitly Not Included

- **Turborepo** — added when build orchestration is needed
- **Bundler** — Bun runs TS directly; build step added per-package when needed (Phase 4+)
- **CI/CD** — not needed for a solo learning project
- **Pre-commit hooks / Husky** — unnecessary overhead
