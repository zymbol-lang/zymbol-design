# What LDV built

> **What this document is.** The evidence behind `LDV.md` § 2's claim that green
> means *"the **language** changes"* — read back out of the applications' own
> findings logs, project by project, and written down so the claim stops being
> an assertion about the method and becomes a record of it.
>
> **Method, and its limit.** A sweep of the nine logs on **2026-09-12** —
> ~6 100 lines across `ZethyCLI/GAPS.md`, `ZyAudit`, `serpiente`, `Zofía`,
> `GO`, `Chaturanga`, `ZyBank`, `GoL` and `klingon_galaxy`. It reads their
> **index tables and resolution rows**, not every finding in full: a claim below
> is as good as the row it came from, and a row that misdescribed its own fix
> would be reproduced here. What is *not* in it: eight of the nine logs are
> indexed by `LDV.md` § 5, which remains the authority on where each lives.

---

## 1. What each project put into the language

Not bugs it found — features and rules that did not exist before it.

| project | version | what entered the language |
|---|---|---|
| **ZethyCLI** | v0.0.3 | modules, `<\ cmd \>` shell execution, multi-turn module state, string building — and a deliberately documented absence that became **`std/net`** in v0.0.7 |
| **ZyAudit** | v0.0.4 | arithmetic bounds in `$[a..b]`, a parenthesised expression as an item of `$++`, the suppression rule for the `ambiguous lifetime` warning |
| **Serpiente** | v0.0.5 | the **`$*`** operator; **`>>~` re-designed** to sparse slots `(row, col, BKS, fg, bg)` with a bold/italic/underline bitmask; `TuiGuard` with `Drop` so a labelled break cannot skip the terminal restore; the LSP recognising `<<\|` as a definition |
| **Zofía** | v0.0.6 | **`std/math`** (20+ functions, `PI`, `E`), **`std/random`** (xoshiro256++), the float formats `#.N\|x\|` and `#!N\|x\|`, and the canonical deep update **`arr[i>j]$~ val`** |
| **囲碁 (GO)** | v0.0.8 | copy-on-write for aggregates (`HLZ-012`/`HLZ-014` — the value model both Rust engines use today), output parameters of a module function under the VM, negative constants in a module body |
| **Chaturanga** | v0.0.9 | the loop-specifier rule — *a thing is a count or a condition, and anything else is refused*: **no truthiness** — and the descending-range warning |
| **ZyBank** | v0.0.9 | `ERROR-ZYB-002`, from which came the function-capture rule — and therefore `MEM-2` |
| **GoL** | — | ten findings, **all open**: four of them are language gaps, not bugs |
| **Hov veS** | v0.0.5 | multi-module orchestration and 3-language i18n at application level |

`$*` reached the lexer, the parser, the tree-walker, the VM, the compiler, the
formatter **and** the semantic analyser. A gap log entry from a snake game is
seven crates wide.

## 2. The chain, and the proof it was already running

`zymbol-design`'s README states the chain this repository holds:

```text
LDV finds the gap  →  the author decides  →  a premise gets an id  →  a cell holds it
```

It was running years before it was written down, and the sweep found the proof.
**`COL-7`** — *nesting is navigated with `>`, and `a[i][j]` does not exist* — was
distilled from `COLLECTIONS.md` on 2026-09-12. Its origin is `GAP-Z006` in
**Zofía**, a neural-network library written in Spanish, which needed a canonical
deep update for tensors and did not have one. The premise, the 61 cells that hold
it, and the document it was distilled from all descend from one application's
inability to write a line.

Nobody had connected those four things. That is what ids are for.

## 3. What was refused, and the rules that only live in a log

The refusals are worth more than the acceptances, because a refusal states a
rule and an acceptance only exercises one.

| refused | project | reason given |
|---|---|---|
| raw strings for shell interpolation | ZyAudit `IDEA-001` | changing `{var}` is a high-impact breaking change; the alternatives conflict with the `$` operators "or introduce new vocabulary without sufficient justification" |
| `$@` as a functional map | Zofía `IDEA-Z003` | "`$>` already does exactly the same" |
| terminal resize detection | Serpiente `GAP-004` | not a language gap: `>>?` already answers at any time |
| a native random number | Serpiente `GAP-002` | **"Zymbol does not add functions that are not primitives of the language"** |

The first three are `SYMBOLS.md` § 17 being applied — *derive, do not invent*;
*no new base mark without a documented abstract character*. Those eight rules are
now `SYM-1`…`SYM-8` in `PREMISES.md`, six of them checked by review rather than
by execution, which is a property of what they constrain: a proposal, not a
program.

The fourth is not in § 17, or anywhere else. It is a design decision, stated as
one, living in the findings log of a snake game — and one version later Zofía
obtained `std/random`. The two reconcile (one refused a *primitive*, the other
added a *module*) through a distinction no document makes: § 17 governs what may
become a **mark** and says nothing about what may become a **module**. That
question has eight rules on one side and none on the other, and it is recorded
in `PREMISES.md` § 5 as stated-but-not-declared, because reconstructing a rule
and then attributing it is what that document's § 6 forbids.

**This is the sharpest argument for `LDV.md` belonging to this repository.** A
method that were only validation would not leave, in its logs, design rules that
exist nowhere else. It does.

## 4. What is open

`GoL` is the newest project and the only one whose log is entirely open. Four of
its ten are language gaps:

| | |
|---|---|
| `GAP-GOL-003` | a program cannot construct an error value |
| `GAP-GOL-009` | an interactive Zymbol program cannot be tested from Zymbol |
| `GAP-GOL-010` | a program cannot capture what its own code prints |
| `GAP-GOL-011` | `<\ … \>` discards the exit status of what it ran |

Each is what LDV produces: a real domain forced the question. The answer is the
author's, and until it is given they stay open, which is the correct state for a
question nobody has answered.

## 5. Redoing this sweep

```bash
# the logs
ls */HALLAZGOS*.md ZethyCLI/GAPS.md klingon_galaxy/hallazgos_es.md

# their index rows — what entered, what was refused, and with which reason
grep -E "^\| *\[?\*{0,2}(GAP|IDEA|HLZ|ERROR)" */HALLAZGOS*.md
```

The rows carry a resolution column naming the version and, usually, the crates
touched. A project whose rows stop naming what changed has stopped feeding this
document, and that is the signal to look for.
