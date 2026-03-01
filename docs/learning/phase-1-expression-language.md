# Phase 1: Expression Language — Learning Path

## Overview

Build a custom expression language from scratch: lexer, Pratt parser, and tree-walking evaluator. The language evaluates RPG expressions like `player.atk * 4 - target.def * 2` and `roll(2, 6) + max(0, player.str - 5)` against game state.

**What you're building:** source text → tokens → AST → evaluated value

**What you're learning:** how programming languages actually work under the hood — the same fundamentals behind TypeScript, Babel, ESLint, Prettier, GraphQL, and every config language you use.

---

## Pre-Reading

Read before writing any code. The concepts need to land first, or you'll be debugging without understanding.

### Week 1: Foundations

#### 1. Crafting Interpreters, Chapter 4: Scanning

**URL:** https://craftinginterpreters.com/scanning.html

The scanner (lexer) turns raw source text into a flat list of tokens. This is your foundation — the rest of the pipeline never touches raw characters again.

**Focus on:**

- The character-by-character loop pattern
- `peek()` (look without consuming) vs `consume()` (advance and return)
- How multi-character tokens like `!=` are recognized (one character of lookahead)
- How number literals are scanned (consume digits, optionally a `.` and more digits)
- How identifiers vs keywords are distinguished (scan the name, check a keyword table)
- Error handling: record the error, keep scanning, report all errors at the end

**Key vocabulary:**

- **Lexeme** — the raw substring from source (the characters `!=`)
- **Token** — the lexeme bundled with its type (`BANG_EQUAL`), optional literal value, and line number
- **Maximal munch** — always consume the longest matching lexeme (`!=` not `!` then `=`)

#### 2. Crafting Interpreters, Chapter 5: Representing Code

**URL:** https://craftinginterpreters.com/representing-code.html

Defines the data structures the parser produces and the evaluator consumes.

**Focus on:**

- Why you need a tree, not a flat list (nesting requires hierarchy)
- Each grammar rule maps to a node type: `BinaryExpr`, `UnaryExpr`, `LiteralExpr`, `GroupingExpr`
- The **Visitor pattern** concept — adding operations (evaluate, pretty-print) without modifying node types

**TypeScript note:** You won't use OOP visitors. Discriminated unions with exhaustive `switch` statements are the idiomatic TypeScript approach. But understand the concept — it's the same idea.

**Skim:** The code generation stuff. The takeaway is the node types, not the Java boilerplate.

#### 3. Crafting Interpreters, Chapter 6: Parsing Expressions

**URL:** https://craftinginterpreters.com/parsing-expressions.html

**Read this to feel the pain of recursive descent for expressions.** The grammar stratification approach works:

```
expression -> equality
equality   -> comparison ( ("!=" | "==") comparison )*
comparison -> term ( (">" | ">=" | "<" | "<=") term )*
term       -> factor ( ("-" | "+") factor )*
factor     -> unary ( ("/" | "*") unary )*
unary      -> ("!" | "-") unary | primary
primary    -> NUMBER | STRING | "true" | "false" | "nil" | "(" expression ")"
```

Each precedence level is a separate function. It works, but adding a new operator means restructuring function chains. This is the problem Pratt parsing solves. You need to have felt why it's a problem before the solution clicks.

**Also focus on:**

- "Panic mode" error recovery — when a syntax error is found, discard tokens until a safe synchronization point, then resume. This lets you report multiple errors in one pass.

### Week 1-2: The Pratt Pivot

#### 4. Bob Nystrom: "Pratt Parsers: Expression Parsing Made Easy"

**URL:** https://journal.stuffwithstuff.com/2011/03/19/pratt-parsers-expression-parsing-made-easy/
**Companion code:** https://github.com/munificent/bantam (Java)

**This is the key article.** It replaces the stratified grammar with two dispatch maps and a single loop.

**The two core abstractions:**

- **Prefix parselet** — called when a token appears at the *start* of an expression (no left-hand side). Handles: literals, identifiers, unary operators, grouping `(`.
- **Infix parselet** — called when a token appears *after* a left operand. Handles: binary operators, postfix operators, function calls `(`, ternary `? :`.

**The core loop (pseudocode):**

```
parseExpression(precedence):
  token = consume()
  prefix = prefixParselets[token.type]   // must exist or error
  left = prefix.parse(this, token)

  while precedence < getPrecedence():    // peek at next token's precedence
    token = consume()
    infix = infixParselets[token.type]
    left = infix.parse(this, left, token)

  return left
```

**Associativity:** Left-associative operators pass their own precedence when recursing. Right-associative operators pass `precedence - 1`.

Read in full. Then look at the Bantam repo — skim the Java or find a community TypeScript port.

#### 5. matklad: "Simple but Powerful Pratt Parsing"

**URL:** https://matklad.github.io/2020/04/13/simple-but-powerful-pratt-parsing.html
**Companion code:** https://github.com/matklad/minipratt (Rust)

**Read after Nystrom.** This reframes precedence as **binding power pairs** `(left_bp, right_bp)` — probably the formulation you'll implement.

**The key insight:** Associativity isn't a special-cased flag. It's a consequence of binding power asymmetry:

| Scenario | Binding powers | Why |
|----------|---------------|-----|
| Left-associative `+` | `(5, 6)` | Right BP is higher, so the next `+` at `left_bp=5` loses to `min_bp=6` and stops |
| Right-associative `^` | `(6, 5)` | Right BP is lower, so the next `^` at `left_bp=6` beats `min_bp=5` and continues |
| Prefix `-` | `(-, 9)` | Only right side matters |
| Postfix `!` | `(11, -)` | Only left side matters |

**Critical rule:** Binding powers must never be equal between operators that interact. Space them out — use even numbers for operators, keep odds as gaps for future operators.

### Read When You Need It

#### 6. Crafting Interpreters, Chapter 7: Evaluating Expressions

**URL:** https://craftinginterpreters.com/evaluating-expressions.html

Read when you start building the evaluator (build step 3). Covers the tree-walking pattern, runtime type checking, truthiness rules, and runtime error handling.

#### 7. Crafting Interpreters, Chapter 8: Statements and State

**URL:** https://craftinginterpreters.com/statements-and-state.html

Read when you reach build step 8 (statement sequences). **Important difference from Lox:** your expression language doesn't manage its own variables or scoping. Your "state" is the game context object passed into `evaluate()` — expressions read from it, they don't create variables in it.

#### 8. Eli Bendersky: "Top-Down Operator Precedence Parsing"

**URL:** https://eli.thegreenplace.net/2010/01/02/top-down-operator-precedence-parsing

Skim to learn the original Pratt vocabulary — you'll encounter these terms in other codebases:

| Original term | Modern equivalent |
|---------------|-------------------|
| `nud` (null denotation) | Prefix parselet |
| `led` (left denotation) | Infix parselet |
| `lbp` (left binding power) | Left BP / precedence level |
| `rbp` (right binding power) | The `min_bp` argument to recursive `parseExpression()` |

### Optional Deep Dives

- **Theodore Norvell: "Parsing Expressions by Recursive Descent"** — https://www.engr.mun.ca/~theo/Misc/pratt_parsing.htm — Proves Pratt parsing and precedence climbing are the same algorithm. Read after your implementation works.
- **matklad: "From Pratt to Dijkstra"** — https://matklad.github.io/2020/04/15/from-pratt-to-dijkstra.html — Proves Pratt parsing and shunting-yard are the same algorithm (one recursive, one iterative with explicit stack). Interesting, not essential.
- **Pratt's 1973 paper** — https://tdop.github.io/ — The original. Historical context and bragging rights.

---

## Build Sequence

### Step 1: Lexer

**Goal:** `lex("2 + 3 * 4")` → `[NUMBER(2), PLUS, NUMBER(3), STAR, NUMBER(4), EOF]`

**Token types to start with:**

```
NUMBER  PLUS  MINUS  STAR  SLASH  LPAREN  RPAREN  EOF
```

**Approach:**

- Write the test first, then make it pass
- Character-by-character loop with `peek()` and `advance()`
- Skip whitespace
- Multi-digit numbers and decimals: consume digits, optionally `.` and more digits
- Don't try to handle negative numbers in the lexer — `-5` is `MINUS` then `NUMBER(5)`. The parser handles unary minus.

**Edge cases to test:** whitespace everywhere, multi-digit numbers, decimals (`3.14`), consecutive operators (`2+-3`), empty input, just whitespace.

### Step 2: Parser (Pratt)

**Goal:** Correct precedence. `2 + 3 * 4` → `BinaryExpr(PLUS, Literal(2), BinaryExpr(STAR, Literal(3), Literal(4)))`

**Start small:**

- Just `+` and `*`. Get `2 + 3 * 4` right.
- Binding power table:

  | Operator | Left BP | Right BP |
  |----------|---------|----------|
  | `+` `-`  | 5       | 6        |
  | `*` `/`  | 7       | 8        |

- Add grouping `( )` — the `(` is a prefix parselet that calls `parseExpression(0)` then consumes `)`.
- Add unary `-` — a prefix parselet with right BP of 9.

**Build a pretty-printer early.** Something that turns the AST into `"(+ 2 (* 3 4))"`. This is your primary debugging tool for every step that follows.

**Tests to write:**

- `2 + 3` — basic binary
- `2 + 3 * 4` — precedence: `(+ 2 (* 3 4))`
- `(2 + 3) * 4` — grouping overrides precedence: `(* (+ 2 3) 4)`
- `2 + 3 + 4` — left associativity: `(+ (+ 2 3) 4)`
- `-5` — unary prefix
- `-(2 + 3)` — unary on grouped expression
- `2 * -3` — unary inside binary

### Step 3: Evaluator

**Goal:** `evaluate(parse(lex("2 + 3 * 4")))` → `14`

**Approach:**

- Post-order tree walk: evaluate children first, then apply the operator
- Discriminated union switch on `node.type`
- At this point you have a working calculator

**Tests:** Same expressions as the parser tests, but asserting on the computed value.

### Step 4: Comparison Operators

**Goal:** `<`, `>`, `==`, `!=`, `<=`, `>=`

**Lexer changes:** Two-character tokens need lookahead. When you see `<`, peek at the next character — if it's `=`, consume both and emit `LTE`.

**Binding power (lower than arithmetic):**

| Operator | Left BP | Right BP |
|----------|---------|----------|
| `==` `!=` | 1 | 2 |
| `<` `>` `<=` `>=` | 3 | 4 |
| `+` `-` | 5 | 6 |
| `*` `/` | 7 | 8 |

**Evaluator:** These return booleans. `3 > 2` → `true`, `1 + 2 == 3` → `true`.

### Step 5: Boolean Logic

**Goal:** `and`, `or`, `not`

**Lexer changes:** These are keywords. After scanning an identifier, check it against a keyword table: `{ "and": AND, "or": OR, "not": NOT, "true": TRUE, "false": FALSE }`.

**Binding power:**

| Operator | Left BP | Right BP |
|----------|---------|----------|
| `or` | -3 | -2 |
| `and` | -1 | 0 |
| `==` `!=` | 1 | 2 |
| `<` `>` `<=` `>=` | 3 | 4 |
| `+` `-` | 5 | 6 |
| `*` `/` | 7 | 8 |
| unary `-` `not` | — | 9 |

**Design decision:** What's falsy? Pick a rule and stick with it. Suggestion: only `false` and `null` are falsy (matches Lox, avoids `0` being falsy which causes subtle bugs in damage formulas).

### Step 6: Identifiers and Property Access

**Goal:** `player.hp`, `self.stats.strength`

**This is where it gets RPG-specific.** Your `evaluate()` signature becomes:

```ts
evaluate(node: ASTNode, ctx: GameContext): Value
```

Where `ctx` is something like `{ player, self, target, vars }`.

**Identifier nodes** look up the name in the context. `player` resolves to the player object.

**Dot `.` operator** is an infix parselet with high binding power (say `(13, 14)`). `player.hp` parses as `PropertyAccess(Identifier("player"), "hp")`. Chaining works naturally: `player.stats.str` builds a left-associative chain.

**Fail loudly:** When `player.mana` is referenced and the actor has no `mana` property, throw a `RuntimeError` with context: `"Property 'mana' not found on 'player'"`. Silent nulls in game formulas produce invisible bugs.

### Step 7: Function Calls

**Goal:** `roll(2, 6)`, `max(a, b)`, `floor(player.hp * 0.5)`

**Parser:** The `(` after an expression is an infix parselet — it has a left-hand side (the function name). Parse comma-separated arguments until `)`.

**Evaluator:** Dispatch against a built-in function table, not the game context:

```
roll(count, sides)    — sum of `count` random dice with `sides` faces
max(a, b)             — higher of two values
min(a, b)             — lower of two values
floor(x)              — round down
ceil(x)               — round up
abs(x)                — absolute value
clamp(val, lo, hi)    — constrain to range
```

### Step 8: Statement Sequences

**Goal:** `damage(target, 3); apply_status(target, "poison", 2)`

Semicolons separating expressions. Each is evaluated in order; the last value (or `null`) is returned. This is where chapter 8 of Crafting Interpreters becomes relevant — but you likely don't need variables, scoping, or environments.

### Step 9: Conditionals

**Simplest option:** `if(cond, then, else)` as a built-in function. Already parses from step 7. But needs **lazy evaluation** — you don't want to evaluate both branches, only the one the condition selects.

**Alternative:** Ternary `cond ? then : else` — a mixfix infix parselet. Nystrom's Bantam repo shows the implementation.

### Step 10: Full Test Suite

Test against all IR expression examples from the design doc. Evaluate each against mock game state and assert correct results. This is your milestone: the expression language works for everything the engine will need.

---

## Pitfalls

Things that trip people up on their first Pratt parser:

1. **The same token can be prefix AND infix.** `-` is unary in `-5` and binary in `3 - 5`. `(` is grouping in `(a + b)` and function call in `foo(a)`. You need two dispatch tables.

2. **Getting associativity backwards.** If `a - b - c` parses as `a - (b - c)` instead of `(a - b) - c`, your binding powers are swapped. Left-associative needs right BP > left BP.

3. **Equal binding powers between operators.** If a postfix and infix operator share a left BP, the break condition is ambiguous. Space them out.

4. **EOF must have the lowest binding power.** If it doesn't, the parser loop never terminates.

5. **Prefix parselets must recurse.** Unary `-` must call `parseExpression(rightBP)`, not just consume the next token. Otherwise `-(3 + 4)` breaks.

6. **Only testing simple cases.** Write chain tests: `a + b + c`, `a - b - c`, `a == b == c`. These catch associativity bugs that `a + b` won't.

---

## RPG Expression Examples

These are the kinds of expressions the language needs to handle by the end of Phase 1:

```
# Damage formulas
player.atk * 4 - target.def * 2
roll(2, 6) + player.str - max(0, target.armor - 5)

# Condition checks
player.hp < 20
target.hp <= 0
player.gold >= item.cost

# Boolean logic
player.hp < 20 and not has_status(player, "shielded")
self.faction != player.faction or player.reputation < -50

# Stat calculations
floor(player.intelligence / 4) + player.level
clamp(player.str * 2 - target.def, 1, 999)

# Combat AI decisions
self.hp < self.max_hp * 0.25 and self.has_ability("heal")

# Dialogue conditionals
player.charisma >= 12 or has_flag("rescued_mayor")
```

---

## Resources Summary

| Resource | When to read | Time |
|----------|-------------|------|
| Crafting Interpreters Ch 4 (Scanning) | Before any code | 1-2 hours |
| Crafting Interpreters Ch 5 (Representing Code) | Before any code | 1 hour |
| Crafting Interpreters Ch 6 (Parsing Expressions) | Before any code | 1-2 hours |
| Nystrom: Pratt Parsers blog post | Before writing the parser | 1 hour |
| matklad: Simple but Powerful Pratt Parsing | After Nystrom, before/during parser | 1 hour |
| Crafting Interpreters Ch 7 (Evaluating) | When starting the evaluator | 1-2 hours |
| Crafting Interpreters Ch 8 (Statements) | When reaching step 8 | 1 hour |
| Bendersky: Top-Down Operator Precedence | During/after parser for vocabulary | 30 min skim |
| Bantam repo (github.com/munificent/bantam) | Reference during parser build | Keep open |
| minipratt repo (github.com/matklad/minipratt) | Reference during parser build | Keep open |
