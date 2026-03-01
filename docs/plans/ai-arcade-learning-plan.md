# AI Arcade: Learning & Project Plan

## Where You Are

**Strengths you're bringing:**

- 10 years React — component architecture, state management, rendering lifecycles, hooks, performance optimization
- 4 years TypeScript — type systems, generics, discriminated unions, module systems
- Strong product thinking — the architectural discussions prove you can reason about systems even in unfamiliar domains
- You already instinctively push back when solutions feel wrong (catching me designing a game instead of an AI game-builder, multiple times)

**Gaps this project fills:**

- Compilers/interpreters — how programming languages actually work
- Data structures & algorithms — beyond array methods
- Database design — SQL, schemas, indexing, query optimization
- Server-side development — Bun, WebSocket, request handling
- Game architecture — game loops, ECS, fixed timestep, spatial systems
- Canvas/rendering — imperative drawing, sprite sheets, camera math
- Systems design — multi-component architectures with clear contracts
- DevOps basics — monorepo management, build pipelines, environment config

**Why this matters career-wise:**
The engineers AI can't replace are the ones who understand systems end-to-end. A React developer who also understands parsers, databases, and game loops thinks about problems differently than one who only knows components. Every phase of this project builds a skill that compounds across everything else you'll ever work on.

---

## The Skill Map

Each phase teaches specific skills. Here's what you're actually learning and why it matters beyond this project.

### Phase 1: Expression Language

**New skills:**

- Lexical analysis (tokenization)
- Recursive descent parsing (Pratt parser specifically)
- Abstract Syntax Trees (ASTs)
- Tree-walking interpretation
- Operator precedence and associativity

**Why it matters beyond this project:**

- You'll understand what TypeScript, Babel, ESLint, and Prettier actually do under the hood
- AST manipulation is how codemods work (jscodeshift, ts-morph)
- Every configuration language you'll ever use (SQL, GraphQL, CSS selectors) is parsed this way
- Understanding interpreters makes you better at debugging any language

**Estimated time:** 2-4 weeks

**Pre-reading (do before coding):**

1. "Crafting Interpreters" by Bob Nystrom, Chapters 4-8 (free online at craftinginterpreters.com)
   - Chapter 4: Scanning (lexer) — read thoroughly, this is your foundation
   - Chapter 5: Representing Code (AST) — understand the data structures
   - Chapter 6: Parsing Expressions — this covers Pratt parsing
   - Chapter 7: Evaluating Expressions — tree-walking interpreter
   - Chapter 8: Statements and State — extending the evaluator
2. "Simple but Powerful Pratt Parsing" by Bob Nystrom (blog post) — shorter, focused explanation of the core algorithm
3. Skim the Wikipedia article on "Operator-precedence parser" for the theory

**The build sequence:**

1. Lexer: tokenize `2 + 3 * 4` → `[NUMBER:2, PLUS, NUMBER:3, STAR, NUMBER:4]`
2. Parser: parse into AST with correct precedence (multiply binds tighter than add)
3. Evaluator: walk the AST, compute `2 + (3 * 4) = 14`
4. Add comparison operators: `<`, `>`, `==`, etc.
5. Add boolean logic: `and`, `or`, `not`
6. Add identifiers and property access: `self.hp`, `player.position.x`
7. Add function calls: `distance(self, player)`, `damage(target, 5)`
8. Add statement sequences: `damage(target, 3); apply_status(target, 'poison', 2)`
9. Add conditionals: `if(self.hp < 5, flee(self), attack(self, player))`
10. Full test suite against all IR expression examples from the design doc

**Where AI helps:** Rubber duck your parser design. When you hit a bug where `a.b.c` doesn't parse correctly, describe the problem and think through it together. Don't ask AI to write the parser for you.

**Where AI does NOT help:** Writing the actual lexer, parser, and evaluator code. You need to feel the recursion, struggle with edge cases, and debug the precedence errors yourself. This is where the deep learning happens.

**Milestone:** You can evaluate every expression string from the design doc's IR examples against mock game state and get correct results.

---

### Phase 2: IR Schema & Validation

**New skills:**

- Advanced Zod (discriminated unions, recursive schemas, custom refinements)
- Schema design as system contracts
- Validation pipeline architecture
- Error message design
- Test-driven development with schema fixtures

**Why it matters beyond this project:**

- API contract design (OpenAPI, GraphQL schemas)
- Runtime validation in any TypeScript project
- Thinking about system boundaries and failure modes
- You'll design better TypeScript types everywhere because you understand validation

**Estimated time:** 1-2 weeks

**Pre-reading:**

1. Zod documentation — you probably know basics, but read the advanced section on discriminated unions, transforms, and refinements
2. JSON Schema specification (skim) — understanding the standard helps you think about schema design patterns

**The build sequence:**

1. Define component schemas: TransformSchema, SpriteSchema, StatsSchema, etc.
2. Define entity schema with component composition
3. Define behavior/state machine schema with expression string fields
4. Define zone schema (rooms, connections, features)
5. Define patch operation schema (discriminated union on `op` field)
6. Define rules schema (combat config, progression, zone rules)
7. Define content schema (dialogue trees, items, environmental text)
8. Build the validation pipeline: JSON parse → Zod validate → expression parse per field → report errors
9. Wire in fuzzy function resolution from Phase 1
10. Wire in default substitution for failed expressions
11. Hand-write 3 complete game definitions that pass full validation:
    - A dungeon crawl zone (combat-focused)
    - A town zone (NPC/dialogue-focused)
    - A puzzle zone (rules/interaction-focused)

**Where AI helps:** Reviewing your schema designs for completeness. "Here's my entity schema, what am I missing?" Also useful for generating test fixtures — "give me 10 intentionally broken IR examples I can test my validator against."

**Where AI does NOT help:** The actual schema design decisions. You need to decide what's required vs optional, what the discriminated union keys are, how deep nesting should go. These decisions come from understanding the IR, which you built intuition for in Phase 1.

**Milestone:** Hand-written game definitions pass validation. Intentionally broken inputs produce helpful error messages. Every expression field in a valid definition evaluates correctly via the Phase 1 interpreter.

---

### Phase 3: Database Layer

**New skills:**

- SQL fundamentals (DDL, DML, joins, indexes, aggregations)
- Relational data modeling (normalization, foreign keys, constraints)
- SQLite specifics (WAL mode, pragmas, limitations)
- Vector embeddings conceptually
- Semantic search implementation
- Database migration patterns

**Why it matters beyond this project:**

- Backend development fundamentally requires database knowledge
- Data modeling skills transfer to any storage system (Postgres, DynamoDB, etc.)
- Understanding indexes and query plans makes you better at API design
- Embeddings/vector search is increasingly relevant across the industry

**Estimated time:** 2-3 weeks

**Pre-reading:**

1. "SQLite Tutorial" at sqlitetutorial.net — work through the basics even if it feels elementary
2. Bun SQLite documentation — specifically the API for prepared statements and transactions
3. "The Inner Workings of SQLite" (blog post by Ben Johnson) — helps you understand what indexes actually do
4. Read the README for `@xenova/transformers` — understand what the embedding model does conceptually
5. Skim sqlite-vss documentation for vector search extensions

**The build sequence:**

1. Set up SQLite in a Bun package. Create the database, run a CREATE TABLE, insert a row, query it back. Feel the basics.
2. Implement the full schema from the design doc (games, zones, entity_templates, player_state, events, npc_memory, dm_memory, patches)
3. Write typed query helpers for each table (insertGame, getGame, insertZone, getZonesByGame, etc.)
4. Write a seed script that populates the database with your Phase 2 hand-written game definitions
5. Implement the embedding pipeline: take a text string → embed with all-MiniLM-L6-v2 → store vector
6. Implement vector search: given a query string, find the N most similar records across tables
7. Write the event significance classifier (which events get high significance scores)
8. Write the event log compressor (rolling windows: granular recent, summarized old)
9. Write the context builders for each tier:
   - `buildTier1Context(prompt)` — just the player prompt + system config
   - `buildTier2Context(gameId, targetZone)` — compressed world state + player state + relevant events
   - `buildTier3Context(gameId, zoneState, trigger)` — tight context with semantic search
10. Test context builders against your seeded data — verify token budget compliance

**Where AI helps:** SQL is a language you haven't used. It's totally fine to ask "how do I write a query that joins events with npc_memory filtered by game_id and ordered by significance?" Then study the query, understand why it works, and write the next one yourself. Also good for debugging query performance — "this query is slow, here's the EXPLAIN output, what index am I missing?"

**Where AI does NOT help:** Schema design decisions. You need to decide the relationships, think about what queries you'll need, and design indexes accordingly. If AI designs your schema, you won't understand why it's shaped that way when you need to modify it later.

**Milestone:** Seeded database with full game data. Context builders produce compressed output within token budgets. Vector search surfaces relevant memories for test scenarios. You can explain every table and relationship in the schema from memory.

---

### Phase 4: Tile Engine & Rendering

**New skills:**

- Canvas 2D API (drawing, transforms, compositing)
- Sprite sheet loading and frame extraction
- Tile map rendering (grid-based, multi-layer)
- Camera systems (viewport, scrolling, world-to-screen coordinate transforms)
- Game loop with fixed timestep (requestAnimationFrame + delta time)
- Offscreen canvas and sprite caching (you touched this with the marketing page)
- Performance profiling for canvas

**Why it matters beyond this project:**

- Canvas skills transfer to data visualization, charts, image processing
- Camera/viewport math is the same as any pan-and-zoom interface
- Game loop patterns appear in animations, physics simulations, real-time dashboards
- Performance optimization techniques (caching, culling, batching) apply everywhere

**Estimated time:** 3-4 weeks

**Pre-reading:**

1. MDN Canvas Tutorial — work through all sections, even the basic ones
2. "How to Build a 2D Game Engine" (any of the many blog series, pick one with JS)
3. Study one open-source tile map renderer (like Tiled's TMX format renderer) to see how they handle layers
4. Read about fixed timestep game loops: "Fix Your Timestep!" by Glenn Fiedler

**The build sequence:**

1. Basic canvas setup: render a colored rectangle, handle resize, handle devicePixelRatio
2. Load a sprite sheet image, extract individual tiles by grid position
3. Render a hardcoded tile grid (2D array of tile IDs → rendered tiles on canvas)
4. Add multiple layers (ground, objects, overhead) rendered in order
5. Implement the camera: viewport position, world-to-screen transform, arrow key scrolling
6. Camera follow: camera smoothly follows a target position
7. Map culling: only render tiles visible in the viewport (critical for large maps)
8. Load a zone definition from your IR format → generate a tile map → render it
9. Add the map generator that takes abstract room descriptions and produces tile grids
10. Sprite caching: pre-render tiles with effects to offscreen canvases
11. Frame rate counter and performance monitoring

**Where AI helps:** Canvas API is well-documented but fiddly. Asking "why is my sprite rendering at the wrong position after camera transform" is a legitimate debugging question. Also useful for math questions — "how do I convert screen coordinates to world coordinates given camera offset and zoom?"

**Where AI does NOT help:** Writing the renderer. You need to build intuition for how canvas compositing works, how transforms stack, why draw order matters. If AI writes your camera system, you won't be able to debug it when the camera jitters or tiles render at half-pixel offsets.

**Milestone:** Load a zone definition from your IR, generate a tile map, render it with multiple layers, scroll a camera around it smoothly at 60fps. This is the first time the project looks like a game.

---

### Phase 5: ECS & Core Systems

**New skills:**

- Entity Component System architecture
- System query patterns (iterate entities with specific component sets)
- Collision detection (AABB, tile-based)
- Input abstraction (mapping physical inputs to game actions)
- State machine implementation and evaluation
- Integration between multiple systems (movement affects collision affects rendering)
- The expression evaluator running live in a game loop

**Why it matters beyond this project:**

- ECS is a fundamental architecture pattern used far beyond games
- Collision detection algorithms teach spatial thinking
- State machines appear everywhere (UI flows, workflow engines, protocol handlers)
- System composition (how independent systems interact through shared data) is core systems design

**Estimated time:** 3-4 weeks

**Pre-reading:**

1. "Entity Component System FAQ" on the ECS Wikipedia page — understand the pattern conceptually
2. Read about AABB collision detection (Axis-Aligned Bounding Box)
3. Study one simple ECS implementation in TypeScript (there are several on GitHub)
4. Re-read the behavior state machine section of the design doc

**The build sequence:**

1. Implement entity storage: create entity (returns ID), add component, remove component, destroy entity
2. Implement system registration: systems declare which components they query
3. Movement system: entities with Transform + Velocity get moved each frame
4. Render system: entities with Transform + Sprite get drawn at their position
5. Input system: keyboard state tracking → mapped to game actions → applied to entities with Input component
6. Put them together: a player entity that moves on arrow keys and renders on the tile map from Phase 4
7. Collision system: AABB checks between entities and tile collision layer
8. Player collides with walls (can't walk through solid tiles)
9. State machine evaluator: for entities with Behavior component, evaluate transitions and actions using Phase 1 expression language
10. Create an NPC entity with a patrol behavior defined in IR format — watch it patrol using expression-driven state machine
11. Entity spawning from IR templates (load entity definition → create entity with components)
12. Zone transition trigger (player walks on zone_transition tile → event fires)

**Where AI helps:** When you're stuck on a specific algorithm — "my AABB collision detection is allowing entities to overlap on corners, here's my code, what's wrong?" Also useful for thinking through system interaction order — "should collision run before or after movement? What are the tradeoffs?"

**Where AI does NOT help:** Architecting the ECS itself. The decisions about how to store components, how systems query entities, and how the game loop orchestrates systems need to be yours. These are the core systems design skills you're building.

**Milestone:** A player walks around a zone generated from IR data, collides with walls, and sees NPC entities patrolling with expression-driven behaviors. The entire game state is driven by data, not hardcoded logic.

---

### Phase 6: RPG Game Systems

**New skills:**

- Turn-based combat system design
- UI rendering without a framework (menus, text boxes, HUD elements on canvas)
- Dialogue tree traversal and state management
- Inventory data structures
- Game state machines (exploring → combat → dialogue → exploring)
- Formula evaluation for game mechanics (damage, accuracy, XP)
- Animation sequencing

**Why it matters beyond this project:**

- Building UI without React teaches you what React actually does for you
- State machine composition (game states containing entity states) is advanced architecture
- Formula-driven systems (where the formulas are data) is a powerful pattern for any configurable system

**Estimated time:** 4-6 weeks (this is the biggest phase)

**Pre-reading:**

1. Study how classic RPGs structure their combat (watch YouTube breakdowns of Final Fantasy or Pokémon battle systems)
2. Read about dialogue tree data structures (they're directed graphs)
3. Study one open-source RPG's inventory system design

**The build sequence:**

1. Game state manager: exploring, combat, dialogue, pause, game_over states with transitions
2. Combat trigger: player walks into enemy → game state transitions to combat
3. Basic combat UI: render a combat screen on canvas (enemy display, player stats, action menu)
4. Action selection: navigate a menu with arrow keys, select with confirm key
5. Turn resolution: player selects attack → evaluate damage formula from IR → apply → enemy turn → evaluate AI attack → apply
6. Combat end: enemy dies (XP, loot) or player dies (game over)
7. Dialogue system: player interacts with NPC → game state transitions to dialogue → render dialogue box → show options → handle selection → set flags
8. $GENERATE markers: when dialogue text is "$GENERATE: ...", queue a Tier 3 call (for now, stub with placeholder text)
9. Inventory system: items stored on player entity, equipment slots, use/equip/drop actions
10. Inventory UI: navigable inventory screen rendered on canvas
11. HUD: persistent health bar, minimap, current quest indicator
12. Progression: XP accumulation, level up using formula from IR, stat increases
13. Flag-gated interactions: locked doors check `has_flag()`, NPCs have conditional dialogue branches
14. Quest tracking: active quests, objectives, completion
15. Integration test: hand-write a complete mini-RPG using your IR format and play through it end-to-end

**Where AI helps:** UI layout math. "I need to render a 4-option combat menu centered at the bottom of the screen with a cursor indicator, here's my canvas size, what are the coordinates?" Also good for playtesting feedback — describe what happens when you play through a scenario and discuss what feels wrong.

**Where AI does NOT help:** The combat system design, the state machine architecture, the dialogue traversal logic. These are the systems you need to own completely because they're the core of what makes the engine work.

**Milestone:** You can play through a complete hand-written RPG: walk around a town, talk to NPCs, accept a quest, enter a dungeon, fight enemies in turn-based combat, find items, use keys to unlock doors, defeat a boss, complete the quest. All driven by IR data.

---

### Phase 7: AI Integration

**New skills:**

- Prompt engineering for structured output
- API integration (Anthropic, Google, Groq SDKs)
- Streaming responses and partial parsing
- Error handling at system boundaries
- Vercel AI SDK (streamObject, schema-constrained generation)
- WebSocket architecture (server pushing AI responses to client)
- Context management and token budgeting
- A/B testing prompt variations

**Why it matters beyond this project:**

- AI integration is the most in-demand skill in the industry right now
- Prompt engineering for structured output is useful across any AI-powered product
- Streaming architecture and WebSocket patterns are broadly applicable
- Understanding token economics informs any product that uses AI

**Estimated time:** 3-4 weeks

**Pre-reading:**

1. Anthropic's prompt engineering guide (docs.anthropic.com)
2. Vercel AI SDK documentation — especially streamObject and structured output
3. Read about WebSocket protocol basics
4. Study few-shot prompting techniques

**The build sequence:**

1. Set up the Bun server with WebSocket support
2. Implement the prompt assembler: system prompt + context + instruction → API request
3. Write the Tier 1 system prompt with IR spec, function reference, and 2 example world genomes
4. Test Tier 1: send a player prompt, get back a world genome, validate with your Zod pipeline
5. Store Tier 1 output in database, decompose into tables
6. Write the Tier 2 system prompt (zone-specific subset)
7. Implement Tier 2 context builder reading from database
8. Test Tier 2: given a world, generate a zone, validate, store
9. Wire Tier 2 into zone transitions: player hits a zone boundary → loading screen → Tier 2 call → zone loads
10. Write the Tier 3 system prompt (patch-specific, minimal)
11. Implement Tier 3 context builder with vector search
12. Implement the trigger system: room entry, NPC interaction, combat events, idle timer → Tier 3 call
13. Implement the patch applicator: validate patch → apply to live game state
14. Wire Tier 3 into live gameplay
15. Implement the DM notes persistence: Tier 3 output dm_context → store in dm_memory → feed back on next call
16. Implement the response pipeline with error recovery and self-correction for Tier 1/2
17. End-to-end test: type a prompt → world generates → play through AI-generated content

**Where AI helps:** Prompt engineering is inherently iterative. Discuss prompt designs, analyze AI output failures, brainstorm better few-shot examples. This is the one phase where AI is a genuine collaborator because you're literally designing the interface between your engine and AI models.

**Where AI does NOT help:** The architecture decisions about WebSocket communication, the trigger system design, the patch applicator logic. You built all the systems this connects to — you understand them better than AI does.

**Milestone:** A player types a prompt, an AI generates a complete RPG world, the player explores AI-generated zones with AI-generated NPCs, combat, items, and quests. The DM patches the world during play. It works.

---

## How To Use AI Without Losing The Learning

### The Rules

1. **Never ask AI to write a system from scratch.** Design it yourself, implement it yourself, debug it with AI's help.

2. **Rubber ducking is the primary use.** Explain your problem to AI. Often just articulating it reveals the answer before AI responds.

3. **Ask "why" not "how."** Instead of "how do I implement collision detection," ask "why does my collision detection allow overlap at high speeds?" The first gives you code to copy. The second teaches you about tunneling and continuous collision detection.

4. **Use AI to review, not write.** "Here's my Pratt parser implementation. What edge cases am I missing?" is a great use. "Write me a Pratt parser" is not.

5. **AI can scaffold boilerplate.** Setting up a turborepo workspace, configuring TypeScript, writing test harness setup — this is plumbing that doesn't teach you much. Let AI help here.

6. **AI explains concepts.** "Explain how Pratt parser binding power determines precedence" is great. You're getting a tailored explanation, not copy-paste code.

7. **AI generates test cases.** "Give me 20 expression strings that should exercise my parser's edge cases" — AI is great at this and it doesn't rob you of anything.

8. **When stuck for more than 30 minutes on a bug, describe it to AI.** Don't paste your code and say "fix it." Describe what you expect, what's happening, and what you've tried. Let AI point you in a direction, not give you the answer.

### The Red Lines

- **Don't let AI write your lexer, parser, or evaluator.** This is the whole point of Phase 1.
- **Don't let AI design your database schema.** Think through the relationships yourself.
- **Don't let AI write your game loop or ECS.** You need to understand every frame.
- **Don't let AI write your combat system.** This is core game logic you must own.
- **Don't ask AI to "make it work."** Always understand WHY the fix works.

### The Green Lights

- **Config files, build setup, tooling configuration** — AI can scaffold this
- **Test data generation** — AI is excellent at generating varied test fixtures
- **Math verification** — "Is this the right formula for AABB overlap detection?"
- **API documentation lookup** — "What's the Canvas 2D API for drawing a clipped region?"
- **Concept explanations** — "Explain ECS cache coherence and why TypedArrays help"
- **Code review** — "Review my state machine evaluator for potential issues"
- **Debugging assistance** — After you've tried to solve it yourself

---

## Weekly Structure (Suggested)

If you're doing this alongside work, maybe 10-15 hours per week:

**Monday-Tuesday:** Read/study the pre-reading for current phase. Understand the concepts before coding.

**Wednesday-Thursday:** Build. This is where you write code, struggle, debug, learn.

**Friday:** Test, refactor, document what you learned. Write notes on what clicked and what's still fuzzy.

**Weekend (optional):** Play-test what you've built. Identify what feels wrong. Plan next week's work.

### Tracking Progress

Keep a simple dev journal — even just a text file:

```
## Week 3 - Pratt Parser
- Spent 2 hours stuck on right-associativity. Turns out left binding power = right binding power makes it left-associative, need right BP = left BP - 1 for right-associative.
- Function call parsing was easier than expected — it's just an identifier followed by a paren group.
- Still fuzzy on how member access chains (a.b.c) interact with function calls (a.b.c()).
- TODO: Add error recovery to the parser so it doesn't crash on first bad token.
```

This journal becomes invaluable. Six months from now when you're debugging a production issue with the expression language, your notes tell you exactly why you made each design decision.

---

## Realistic Timeline

| Phase                        | Duration  | Cumulative |
| ---------------------------- | --------- | ---------- |
| Phase 1: Expression Language | 2-4 weeks | Month 1    |
| Phase 2: IR Schema           | 1-2 weeks | Month 1-2  |
| Phase 3: Database            | 2-3 weeks | Month 2-3  |
| Phase 4: Tile Renderer       | 3-4 weeks | Month 3-4  |
| Phase 5: ECS & Systems       | 3-4 weeks | Month 4-5  |
| Phase 6: RPG Systems         | 4-6 weeks | Month 5-7  |
| Phase 7: AI Integration      | 3-4 weeks | Month 7-8  |

**Total: 6-10 months.** This is real. Don't try to compress it. The learning is the point.

If you ship Phase 6 and never get to Phase 7, you've still built an RPG engine from scratch with a custom scripting language, a database layer, a tile renderer, an ECS, and a turn-based combat system. That's an extraordinary portfolio piece and a massive skill expansion.

The AI integration (Phase 7) is the cherry on top. Everything before it is the meal.

---

## What You'll Be Able To Do After This

**Technical skills gained:**

- Design and implement a programming language (expression parser + interpreter)
- Design database schemas and write efficient queries
- Build real-time rendering systems
- Architect game engines with ECS patterns
- Design and implement state machines
- Build multi-tier AI integration with context management
- Manage a complex monorepo with multiple packages

**Thinking skills gained:**

- Systems design: how components communicate through contracts
- Performance reasoning: why something is slow and how to fix it
- Failure mode thinking: what happens when this breaks and how to degrade gracefully
- Abstraction design: finding the right level between too specific and too general

**Career positioning:**
You go from "senior React developer" to "senior software engineer who happens to be excellent at React." That's a fundamentally different value proposition in a market where AI can write React components but can't design systems, debug interpreters, or architect game engines.
