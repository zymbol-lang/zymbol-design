# How to change Zymbol

> **Who this is for.** Whoever is about to modify an engine, a diagnostic, a
> premise or a document — human or model. It is not about writing programs *in*
> Zymbol (`interpreter/AGENTIC.md` and `interpreter/LLM.md` are that). It is
> about changing the language itself.
>
> **Why it exists.** Every safeguard in this project can be satisfied by an
> agent that wants to finish. The ones below were all taken, here, by capable
> work under deadline — not by carelessness. Naming them is cheaper than
> detecting them.

---

## 1. The order

**Decide → implement → hold.** Not the other way round, and the middle step is
not the first one.

A harness cannot enforce what the analyser permits. Declaring `MEM-2` as a cell
before the decision produced four red cells that are not regressions; that was
correct, and it was correct *because the decision had been taken first and the
red was named debt*. Red that nobody declared is a regression. Red that nobody
decided is noise, and noise is how a gate stops meaning anything.

**A premise is changed by the author, in a commit that changes nothing else.**
`PREMISES.md` § 7 says why. The short version: on 2026-08-24 an engine, the
guide that documented its rule, and the one-page brief all moved in one commit
and ended up perfectly consistent with each other and in disagreement with the
author. Nothing could go red, because nothing disagreed.

## 2. The six ways out, each of which was taken here

| | the move | when it happened |
|---|---|---|
| 1 | **Document the behaviour as if it were the design.** | `LLM.md` rule 5 states "a named function sees the file's variables" — a description of the engine, written as a rule. `GUIDE.md` § 10b had said the opposite and was deleted the same day the engine changed |
| 2 | **Take the easy branch of a design decision.** | `ERROR-ZYB-002` had two exits — isolate both paths, or capture in both. The second was taken, argued as *"one rule instead of two"*. That is the argument from convenience, and it was made inside the session that was fixing the bug |
| 3 | **Close a finding by checking what it was not.** | `DM-05` was closed with *"all three warn"*. They do warn, identically — and with an array, a Float or an empty string they take **different branches** under that same warning |
| 4 | **Relabel the failure as an exclusion.** | 43 in `corpus.toml`. Each requires a reason, and the reason is written by the same agent that wants to pass |
| 5 | **Re-record the golden.** | `zyddt axis --regen-baseline` rewrote `wording.baseline` and silently dropped a hand-written block explaining the one entry in it. Nothing reported that |
| 6 | **Lower the question.** | turning an `expect` into an `ask`, deleting a cell, or declaring an axis without `expect` "because it would be false in both directions" |

And a seventh, found on 2026-09-12 while writing this: **classify a document by
how another document cites it.** `LDV.md` was filed as a testing method because
ZyDDT's charter cites two points of its decalogue. Its own § 1 separates
validation from verification and files every automated suite, ZyDDT included,
under verification. Reading the citation instead of the source is move 1 applied
to documents.

## 3. What each harness can and cannot see

| harness | asks |
|---|---|
| `zyq consensus` | do the engines agree with each other |
| `zyq expect` | does output match a recorded golden |
| `zyq reject`, `axes/refusal` | is this refused, whatever the message |
| `zyddt ask` | did all three answer alike |
| `zyddt axis` + `expect` | did they agree **wrongly** — the only one that can |
| `zyddt premises` | does every premise have a cell, and every cell a premise |
| `zyquality/cost` | auto-free, and complexity, as ratios against their own control |
| `cargo test` | each crate against its own contracts |

**Three engines agreeing is not correctness.** `zytw` and `zyvm` share the
lexer, the parser and the semantic analyser; `zyjs` was hand-ported from them. A
wrong answer all three inherited is a perfectly green row — which is why
`expect` and the oracle exist, and why premises are asserted rather than asked.

**What nothing covers today**: capability enforcement, termination, shell
quoting (`AGENTIC.md` G1–G3, decisions nobody has taken), and **the text of the
normative documents themselves** — `zyddt premises` checks that ids and cells
line up, not that a premise still says what it said.

## 4. Before you start

- Read the premise, not the description. If `interpreter/` and `PREMISES.md`
  disagree, `interpreter/` is what is wrong.
- If the change needs a premise to change, **stop**, and say so. That is the
  author's decision and it is a commit of its own.
- Look for the finding's id. `HALLAZGOS/`, `Divergente_ES/`, the applications'
  gap logs. A rule that looks arbitrary usually has a reason that was written
  down once, in a place nobody re-reads.

## 5. Before you call it done

- Run `zyddt premises`, `zyddt axis` and `zyq suite`. A suite that could not run
  exits **2**, never 0: nothing ran is not nothing failed.
- **Check the consequence, not the message.** Which branch executed, which value
  came out, which exit code — a diagnostic that matches proves the diagnostic
  matches (move 3).
- Sweep the type range, not a representative: array, `[]`, Float, empty string,
  `0`. Four of the seven cases of the `?`-condition divergence hide behind an
  `Int` that behaves.
- If you excused something, say where and why in the same breath. An exclusion
  whose reason nobody wrote is indistinguishable from a bug somebody hid.
- If you could not do part of it, say which part. Scaling the work down is the
  author's call.
