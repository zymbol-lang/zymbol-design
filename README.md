# zymbol-design

The premises Zymbol's implementations must meet, and nothing else.

## What is here

**The method**, which is where everything else comes from:

- **[`LDV.md`](LDV.md)** — Language-Driven Validation. Not a test method: its own
  § 1 separates *verification* (was the product built right) from *validation*
  (was the right product built) and files every automated suite in the project,
  ZyDDT included, under the first. Its § 2 states what green means — *"the
  **language** changes"* — and closes with the sentence this repository is
  organised around: **TDD is how the interpreter's crates are written; LDV is how
  it is discovered what to write.**

  The evidence is in the applications. `Serpiente` lists *TUI primitives* among
  what it produced, not among what it required: the game and the terminal
  directives were written at the same time, and the game is what said which
  directives had to exist. `ZethyCLI` was deliberately over-ambitious for
  v0.0.2, and shell execution, HTTP and multi-turn module state are what came
  back from it.

  So the chain this repository holds is:

  ```text
  LDV finds the gap  →  the author decides  →  a premise gets an id  →  a cell holds it
  └──────────────────────── zymbol-design ────────────────────────┘      └── ZyDDT ──┘
  ```

- **[`WHAT_LDV_BUILT.md`](WHAT_LDV_BUILT.md)** — the evidence, read back out of
  the nine applications' findings logs: what each one put into the language,
  what was refused and under which rule, and the four language gaps still open.
  It is also where the chain above is shown to have been running before anyone
  wrote it down — `COL-7` traces back to a tensor library that could not express
  a deep update.

- **[`HOW_TO_CHANGE_ZYMBOL.md`](HOW_TO_CHANGE_ZYMBOL.md)** — the door. What to
  read before modifying an engine, the six ways out of a hard decision that were
  each taken here at least once, what every harness can and cannot see, and what
  to check before calling a thing done.

**The rules**, which are what an engine is measured against:

- **[`PREMISES.md`](PREMISES.md)** — the declared premises, by ID. The only part
  that is machine-checkable today: `zyddt premises` crosses each id against the
  cells that hold it.

**The reasoning**, moved here from `interpreter/` on 2026-09-12 because a
document that says what the language *must* do belongs with the rules and not
with the implementation that must meet them:

| document | what it is |
|---|---|
| [`COLLECTIONS.md`](COLLECTIONS.md) | the point of record for the three collections — *and why each rule was decided the way it was* |
| [`SYMBOLS.md`](SYMBOLS.md) · [`SIMBOLOS_ES.md`](SIMBOLOS_ES.md) | the sign system: what may be a mark, how marks combine, and every place the rules do not hold |
| [`MEMORY_MODEL.md`](MEMORY_MODEL.md) | the memory model audited against its implementation |
| [`FORMATTER_RULES.md`](FORMATTER_RULES.md) | `zymbol fmt` is a layout tool and never changes meaning — enforced, not promised |
| [`I18N.md`](I18N.md) · [`USERAPPI18N.md`](USERAPPI18N.md) | the two i18n mechanisms, and what an application must do to be multilingual |
| [`DESIGN_STD_DB.md`](DESIGN_STD_DB.md) | the `std/db` design proposal — its header still says *pre-implementation*, which stopped being true in v0.0.7 |

What stayed in `interpreter/` is everything **derived**: `GUIDE.md`,
`REFERENCE.md`, `IMPLEMENTATION.md`, `ARCHITECTURE.md`, `LLM.md`, `AGENTIC.md`.
Three of them carry *"Interpreter version: v0.0.9"* in their own headers, which
is the test: a document that is versioned with the engine describes it.

Only `PREMISES.md` is checked mechanically. The rest is prose that states rules
nothing verifies yet — distilling each domain into premises with ids is the work
this repository exists to do, and `COL` is the one in progress.

## What this repository is for

Every other layer of this project compares an implementation against another
implementation. `zyq consensus` compares engine to engine; `zyq expect` compares
an engine to its own recorded output; `zyddt ask` requires three engines to
answer alike. None of them can see a rule that all three implementations break
together — and none can see a rule that was *changed* to match what the engines
happened to do.

That is not hypothetical. On 2026-08-24 a rule of the memory model was retired,
the guide that documented it was retired in the same commit, and the one-page
brief was rewritten to describe the result. Everything stayed green, because
everything had moved together. The author found out on 2026-09-11, by reading.

This repository holds the half that nothing else holds: **what the language is
supposed to do**, stated so that an engine can be measured against it.

## How it is used

`ZyDDT/axes/isolation.toml` declares which cell asserts which premise, and
`zyddt premises` crosses the two lists: a premise with no cell, a cell claiming
a premise that does not exist, and a `Held by` naming a cell that is gone are
all reported. ZyDDT treats this repository the way it treats any sibling — if it
is not cloned, the check is **skipped and says so**, never silently passed.

## The rule

**A premise is changed by the author, in a commit that changes nothing else.**
`PREMISES.md` § 4 says why, and what follows from it. The short version: if an
engine cannot meet a premise, the engine is wrong — or the premise is changed on
purpose, first, alone, and by the author. A red cell is never a reason to weaken
the rule it is measuring.

## Related

| repository | what it holds |
|---|---|
| `ZyDDT/` | the cells and pins that measure these premises |
| `zyquality/` | the corpus, the goldens and the shared gate |
| `interpreter/` | the two Rust engines, and the documentation derived from them |
| `web/` | the browser engine |
