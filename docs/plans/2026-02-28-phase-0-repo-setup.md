# Phase 0: Repo Setup Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Set up the slop-engine monorepo with Bun workspaces, shared TypeScript configs, Biome, and the first package (`expr-lang`).

**Architecture:** Bun workspaces monorepo with a shared tsconfig package. Turborepo-compatible directory structure but no Turborepo yet. Biome for linting and formatting.

**Tech Stack:** Bun, TypeScript, Biome

---

### Task 1: Initialize Git

**Files:**
- Create: `.gitignore`

**Step 1: Initialize the git repo**

Run: `git init`

**Step 2: Create `.gitignore`**

```gitignore
node_modules/
dist/
*.tsbuildinfo
.DS_Store
```

**Step 3: Commit**

```bash
git add .gitignore CLAUDE.md docs/
git commit -m "chore: initial commit with project plan and CLAUDE.md"
```

---

### Task 2: Root `package.json`

**Files:**
- Create: `package.json`

**Step 1: Create root `package.json`**

```json
{
  "name": "slop-engine",
  "private": true,
  "workspaces": ["packages/*"],
  "scripts": {
    "test": "bun test",
    "lint": "bunx --bun biome check .",
    "format": "bunx --bun biome format --write ."
  }
}
```

Note: `"private": true` prevents accidental npm publish. Bun resolves `workspaces` the same way npm/yarn do.

**Step 2: Commit**

```bash
git add package.json
git commit -m "chore: add root package.json with bun workspaces"
```

---

### Task 3: Shared TypeScript Config Package

**Files:**
- Create: `packages/tsconfig-base/package.json`
- Create: `packages/tsconfig-base/base.json`
- Create: `packages/tsconfig-base/node.json`
- Create: `packages/tsconfig-base/browser.json`

**Step 1: Create `packages/tsconfig-base/package.json`**

```json
{
  "name": "@slop-engine/tsconfig",
  "private": true
}
```

**Step 2: Create `packages/tsconfig-base/base.json`**

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "compilerOptions": {
    "strict": true,
    "target": "ESNext",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noUncheckedIndexedAccess": true,
    "noEmit": true
  }
}
```

Key decisions:
- `moduleResolution: "bundler"` — works with Bun's module resolution
- `noUncheckedIndexedAccess` — forces handling `undefined` on index access, catches real bugs
- `noEmit` — Bun runs TS directly, no compile step needed
- `isolatedModules` — ensures each file can be independently transpiled

**Step 3: Create `packages/tsconfig-base/node.json`**

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "./base.json",
  "compilerOptions": {
    "types": ["bun-types"]
  }
}
```

**Step 4: Create `packages/tsconfig-base/browser.json`**

```json
{
  "$schema": "https://json.schemastore.org/tsconfig",
  "extends": "./base.json",
  "compilerOptions": {
    "lib": ["DOM", "DOM.Iterable", "ESNext"]
  }
}
```

**Step 5: Commit**

```bash
git add packages/tsconfig-base/
git commit -m "chore: add shared tsconfig package with base, node, and browser configs"
```

---

### Task 4: expr-lang Package Scaffold

**Files:**
- Create: `packages/expr-lang/package.json`
- Create: `packages/expr-lang/tsconfig.json`
- Create: `packages/expr-lang/src/index.ts`

**Step 1: Create `packages/expr-lang/package.json`**

```json
{
  "name": "@slop-engine/expr-lang",
  "private": true,
  "version": "0.0.1",
  "type": "module",
  "main": "src/index.ts",
  "devDependencies": {
    "@slop-engine/tsconfig": "workspace:*"
  }
}
```

Note: `"main": "src/index.ts"` — Bun resolves TS directly, no need for a `dist/` path.

**Step 2: Create `packages/expr-lang/tsconfig.json`**

```json
{
  "extends": "@slop-engine/tsconfig/node.json",
  "compilerOptions": {
    "rootDir": "src",
    "outDir": "dist"
  },
  "include": ["src"]
}
```

The `extends` uses the package name, resolved by Bun workspaces.

**Step 3: Create `packages/expr-lang/src/index.ts`**

```typescript
export {};
```

Placeholder entry point. Phase 1 work starts here.

**Step 4: Commit**

```bash
git add packages/expr-lang/
git commit -m "chore: scaffold expr-lang package"
```

---

### Task 5: Install Dependencies

**Step 1: Install `bun-types` at root**

```bash
bun add -d bun-types
```

This installs to the root and makes `bun-types` available to all packages that reference it in their tsconfig.

**Step 2: Install Biome**

```bash
bun add -d @biomejs/biome
```

**Step 3: Install workspace dependencies**

```bash
bun install
```

This links the workspace packages (resolves `workspace:*` references).

**Step 4: Commit**

```bash
git add package.json bun.lock
git commit -m "chore: install bun-types and biome"
```

---

### Task 6: Biome Configuration

**Files:**
- Create: `biome.json`

**Step 1: Create `biome.json`**

```json
{
  "$schema": "https://biomejs.dev/schemas/2.0.0/schema.json",
  "files": {
    "ignore": ["dist/", "node_modules/"]
  },
  "linter": {
    "enabled": true,
    "rules": {
      "recommended": true
    }
  },
  "formatter": {
    "enabled": true,
    "indentStyle": "tab",
    "lineWidth": 100
  },
  "javascript": {
    "formatter": {
      "quoteStyle": "double",
      "semicolons": "always"
    }
  }
}
```

Note: Using Biome defaults (tabs, double quotes, semicolons). Adjust if you have different preferences.

**Step 2: Verify Biome works**

Run: `bunx --bun biome check .`
Expected: No errors (or only on the placeholder `export {};`)

**Step 3: Commit**

```bash
git add biome.json
git commit -m "chore: add biome config for linting and formatting"
```

---

### Task 7: Verify Everything Works

**Step 1: Verify TypeScript**

Run: `bunx tsc --project packages/expr-lang/tsconfig.json --noEmit`
Expected: No errors

**Step 2: Verify Biome linting**

Run: `bun run lint`
Expected: Clean output, no errors

**Step 3: Verify Biome formatting**

Run: `bun run format`
Expected: No files changed (already formatted)

**Step 4: Create a quick smoke test**

Create `packages/expr-lang/src/index.test.ts`:

```typescript
import { describe, it, expect } from "bun:test";

describe("expr-lang", () => {
	it("should exist", () => {
		expect(true).toBe(true);
	});
});
```

**Step 5: Run tests**

Run: `bun test`
Expected: 1 test passing

**Step 6: Commit**

```bash
git add packages/expr-lang/src/index.test.ts
git commit -m "chore: add smoke test to verify repo setup"
```

---

### Task 8: Update CLAUDE.md

**Files:**
- Modify: `CLAUDE.md`

**Step 1: Update the project name reference and any repo-specific details**

Update the project name to "Slop Engine" if desired, and update the project structure section to reflect the actual package layout now that it exists.

**Step 2: Commit**

```bash
git add CLAUDE.md
git commit -m "chore: update CLAUDE.md for new repo structure"
```
