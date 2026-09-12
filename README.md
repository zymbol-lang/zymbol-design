# zymbol-design

The premises Zymbol's implementations must meet, and nothing else.

- **[`PREMISES.md`](PREMISES.md)** — the declared premises, by ID.

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
