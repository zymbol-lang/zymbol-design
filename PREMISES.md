# Zymbol — declared premises

> **Rank: source.** This document is not derived from any engine. `GUIDE.md`,
> `REFERENCE.md`, `LLM.md` and `AGENTIC.md` describe what the implementations
> do; this one states what they **must** do. Where a description and a premise
> disagree, the description is what is wrong.
>
> **Why it lives in its own repository.** Not for tidiness. A premise that sits
> in the same tree an agent edits while fixing an engine can be edited as *part
> of* fixing the engine — and that is exactly how the rule this file exists to
> protect was lost once already (§ 4). Here it is not in the working tree at
> all when somebody is working on `interpreter/`.
>
> **Language.** English, like the rest of the language's documentation
> (`CLAUDE.md`). Each premise carries the author's **original wording in
> Spanish**, verbatim, because that text is the author's and a translation of it
> is already an interpretation.

---

## 1. How to read an entry

```
## MEM-n — <one line>
Original    the author's words, verbatim, undated only if never restated
Normative   the rule, in the form an engine can be measured against
Measured    what the engines actually do, with the date it was measured
Held by     the cells that turn red if it stops holding — or "nothing yet"
```

`Measured` may say the premise does not hold. That is not a contradiction: a
premise is what must be true, and the distance between it and `Measured` is the
work. A premise is never weakened to match an engine. It is changed only by the
author, deliberately, and never in the same commit as an engine change (§ 4).

IDs are permanent and are not reused. `MEM` is the memory model; other domains
take their own prefix as they are declared.

---

## 2. The memory model

### MEM-1 — Constants are global, and never vary

**Original** — «Las constantes son globales sino pierden su sentido pero estas
nunca varian en su valor.»

**Normative** — A constant declared with `:=` at file level is readable from
every function in that file. Its value never changes: reassignment is a static
error, not a runtime one.

**Measured** (2026-09-12, zymbol 0.0.9, three engines) — **Holds.** `LIMITE := 99`
is read inside a function body; `PI = 3` after `PI := 3.14` is
`error: cannot reassign constant 'PI'`.

**Held by** — `isolation/const-is-global`.

---

### MEM-2 — A variable is visible only inside its own scope

**Original** — «Las variables solo pueden ser vistas o modificadas en su scope
si es el principal solo en el scope principal.»

**Normative** — A variable is visible and modifiable only within the scope that
declares it. A variable of the main scope is visible only in the main scope; a
function body is a different scope and does not reach it. Values cross that
boundary as parameters (MEM-5), never by being in view.

**Measured** (2026-09-12) — **Does not hold, through exactly one door.** A named
function reads the file's top-level names, by value, at call time, when they are
declared lexically before it. `zymbol check` says nothing. Every other door is
refused statically: a name from a block, a name from another frame, a name
declared after the function, a name from the importing script.

**History** — it held until **2026-08-24**. `GUIDE.md` § 10b documented the
isolation as deliberate; commit `fbccc8e` retired both, to resolve ZyBank's
`ERROR-ZYB-002`, which had found that the same body behaved differently
depending on how it was reached (a direct call ran isolated; the same function
taken as a value captured). Of the two ways to make that coherent — isolate both
paths, or capture in both — the second was taken, with the argument *"one rule
instead of two"*.

**Held by** — `isolation/file-var-{direct,as-value,nested,lambda}` (4 cells,
**red**: this is the distance, not a regression) and
`isolation/{block-var,caller-local}-*` (8 cells, green: the doors that are shut).

---

### MEM-3 — A module owns its environment

**Original** — «Los Modulos (Un modulo no es una clase) tienen su propio entorno
de variables, constantes y estas estan auto contenidas en el.»

**Normative** — A module is **not** a class. It has its own environment of
variables and constants, self-contained in both directions: the module does not
see the importing script's scope, and the importer does not see the module's
variables. Identity is the file path, so one module is one environment however
many aliases name it.

**Measured** (2026-09-12) — **Holds, both directions.** A module body naming the
importer's variable is `undefined variable`, reached through the import and
reported with `note: reached from`.

**Held by** — **nothing yet.** A module is a file and a generated cell is one
file; until the runner can emit a sibling, this is a pin or it is untested.
Today it is untested.

---

### MEM-4 — A module's state is written from inside the module

**Original** — «Las variables de un modulo pueden ser modificadas dentro del
modulo desde una funcion. Este seria el caso mas cercano a una variable global
al modulo.»

**Normative** — A module's variables are modified by that module's own
functions, and only by them. They are not exportable: only constants and
functions leave. This is the closest thing the language has to a global
variable, and the fence is what keeps it from being one.

**Measured** (2026-09-12) — **Holds, and the fence is enforced.** `#> { n }`
where `n` is a variable is `E005: Item 'n' not found in module`. State persists
across calls and is shared by every alias and importer of that file.

**Held by** — **nothing yet**, same reason as MEM-3.

---

### MEM-5 — A crossing is declared at both ends

**Original** — «Los pasos y retornos una variable en funciones las cuales deben
de pasarse por parametro no solo indicando en la funcion si esta se podria
retornar modificada en la funcion sino que tambien tendras la informacion desde
la llamada.»

**Normative** — Values cross a function boundary as parameters. A parameter that
may come back modified is marked `<~` **at the signature and at every call
site**, and each half is an error without the other. The cost of reviewing a
call is the call: a reader never opens the function to find out what it changes.

**Measured** (2026-09-12) — **Holds, both halves.** A marked signature with an
unmarked call site and an unmarked signature with a marked call site are both
static errors, each naming the other half.

**Held by** — `isolation/output-parameter-marked-both-ends`,
`isolation/working-copy-stays-inside`,
`isolation/output-parameter-unmarked-at-call`,
`isolation/output-mark-without-an-output-parameter`.

---

## 3. Stated but not yet declared

Things the author has said, which are **not** premises until restated here as
one. They are listed so that "nobody wrote it down" never becomes the reason
something was lost.

| what was said | measured 2026-09-12 |
|---|---|
| «scope de variable sin permitir la reutilización de nombres» (2026-09-11) | **does not hold**, four ways: an inner assignment mutates the outer name with no diagnostic; a variable `dato` and a function `dato()` coexist silently; a function may be redefined silently; a parameter may reuse a file-level name |
| «auto limpieza de memoria o borrado manual de memoria» (2026-09-11) | **holds**: auto-free at last use is required to be unobservable, `\ x` destroys and a later use aborts — though only at runtime, never in `check` |

### Open questions this domain raises

- **Is a lambda a self-contained space, like a function?** MEM-2 does not say.
  It is the half `ERROR-ZYB-002` never resolved toward isolation, and the answer
  decides four of the eighteen cells in `axes/isolation.toml`.
- **Does MEM-2 apply to reading, to writing, or to both?** Today the write side
  is isolated and the read side is not; the premise as written covers both
  ("vistas o modificadas"), which is the reading this document takes.

---

## 4. The rule that protects this file

**A premise is changed by the author, in a commit that changes nothing else.**

Not a convention — a check. A commit that touches this repository and an engine
repository for the same change is the shape in which a rule gets quietly
rewritten to match what was just implemented. It has happened once, on
2026-08-24, and nothing reported it because the engine, the guide and the
one-page brief all moved together and ended up perfectly consistent with each
other and in disagreement with the author.

The same doctrine already governs `zyddt axis --regen-baseline`
(*"deliberate and separate"*). This is that doctrine applied where it matters
most.

Three things follow:

1. **A premise is never edited as part of fixing something.** If an engine
   cannot meet a premise, the engine is wrong, or the premise is changed on
   purpose, first, alone, and by the author.
2. **A red cell is not a reason to weaken a premise.** MEM-2 has four red cells
   today. They stay red until the author decides, and the decision is recorded
   here with its date.
3. **A description that disagrees with a premise is a defect in the
   description.** `LLM.md` rule 5 disagrees with MEM-2 as of 2026-09-12, and
   that is a bug in `LLM.md`, not evidence about the rule.
