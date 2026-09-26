# Candidate premises

> **What this is.** The register of rules Zymbol appears to have and
> `PREMISES.md` does not declare. It is the mirror of `MODEL.md` § 8: that
> document names layers **B** (measured, not declared) and **D** (open, and
> blocking something); this one holds them, each with an id, a folder of probes
> and the answer all three engines give — so that deciding one is reading rather
> than re-deriving.
>
> **What this is not.** Not `PREMISES.md`: nothing here is normative. Not a test
> suite: the probes have no expected output and `zyq suite` does not run them. A
> candidate that "fails" is not a regression, because there is nothing yet for it
> to fail against.
>
> **The editorial rule, mirrored from `MODEL.md`.** *A reason is not a rule.*
> An explanation of why a decision was taken does not become a property of the
> language by being written down, and **does not earn a folder of probes by being
> interesting**. § 2 is the filter at the door, and it exists because this
> register already admitted one entry that failed it.
>
> **The rule this register serves.** *A premise is changed — or created — by the
> author, in a commit that changes nothing else* (`PREMISES.md` § 7). This file
> exists so that the author's decision is the only missing step, and so that
> «nobody wrote it down» never becomes the reason something was lost
> (`PREMISES.md` § 6).

---

## 1. The states

| state | what it means |
|---|---|
| `drafted` | stated here. No probes yet, or none are possible |
| `measured` | the probes exist and the three engines' answers are recorded below |
| `decided` | **the author has stated the rule.** It may differ from what the engines do; that distance is then the work, not a contradiction |
| `promoted` | it is in `PREMISES.md` with a permanent id. The entry here becomes a pointer and is not edited again |
| `rejected` | this is not a rule of the language. The entry stays, with the date and the reason |

**`measured` is not `decided`.** A recorded behaviour is evidence about the
engines; a premise is a statement about the language. Turning the first into the
second without the author is the move `PREMISES.md` § 6 forbids and the one
`HOW_TO_CHANGE_ZYMBOL.md` § 2 lists first: *document the behaviour as if it were
the design.* So every answer below is written as **what the engines answer**,
never as what the rule is.

## 2. What earns an id here

Four questions, in order. A candidate that fails any of them does not get a
folder.

1. **Is it a rule?** Something a *program* could violate. If it constrains a
   proposal rather than a program — as SYM-1…SYM-7 do — it can still be a premise,
   but it is `drafted` and held by review, and that is said at the door rather
   than discovered at promotion.
2. **Can a probe show it positively?** If the only probe available is *"this
   spelling is refused"*, the rule is about the absence of a feature, and **no
   program demonstrates the absence of a feature** (`AGT-3`, about `eval`). A
   probe would only test whichever spelling its author guessed.
3. **Is it already implied by a declared premise?** Then it needs a **cell**, not
   an id. § 6.
4. **Is it a reason?** Justifications for a design choice belong in
   `MODEL.md` § 3 and never here.

**The entry that failed this.** `C-OBJ-1` — *"a module has no receiver, no
inheritance, no dispatch"* — was admitted on 2026-09-21 with three probes trying
`v.length()`, `self` and `Punto(1, 2)`. It fails **2** and **4**: not adopting an
object model is a design choice with a reason (`MODEL.md` § 3.5 gives it in a
paragraph), and its probes could only ever show that three guessed spellings are
refused. Rejected the same week, and recorded rather than erased so that nobody
proposes it again.

## 3. How to run one

The runner is the project's own, and it passes no verdict:

```bash
cd zyquality
./zyq show ../zymbol-design/candidates/C-TYP-3/p1-cambio-de-tipo.zy

# a whole candidate
for f in ../zymbol-design/candidates/C-TYP-3/*.zy; do ./zyq show "$f"; done
```

`zyq show` prints what **each** engine said — exit code, stdout, stderr. That is
why it is the right tool: a candidate has no expected output, and giving it one
would be deciding it. The probes sit outside `zyq suite` for the same reason.

Each fact lives once: the probes are `.zy` files, their recorded answers are in
this file, and there is no third copy.

---

## 4. The register

| id | would become | statement | model | state |
|---|---|---|---|---|
| [C-TYP-1](#c-typ-1) | `TYP-1` | A value carries a type; a name does not | § 5.1 | `measured` |
| [C-TYP-2](#c-typ-2) | `TYP-2` | What inference reaches, it refuses | § 5.1–5.2 | `measured` — **splits in two** |
| [C-TYP-3](#c-typ-3) | `TYP-3` | A type change on a name is a warning, not an error | § 5.3 | `measured` — **decision needed first** |
| [C-TYP-4](#c-typ-4) | `TYP-4` | Nothing is coerced | § 5.1 | `measured` |
| [C-FUN-1](#c-fun-1) | `FUN-1` | Arity is fixed | § 6 | `measured` |
| [C-FUN-2](#c-fun-2) | `FUN-2` | Nothing builds a function out of functions | § 6 | `measured` — **rule or gap?** |
| [C-FUN-3](#c-fun-3) | `FUN-3` | A named function used as a value captures nothing | § 6 | `measured` |
| [C-GRW-1](#c-grw-1) | `GRW-1` | Which door a new capability comes through | — | `drafted` — no probe is possible |
| ~~C-OBJ-1~~ | — | ~~a module has no receiver, inheritance or dispatch~~ | § 3.5 | **`rejected`** 2026-09-22 — a reason, not a rule (§ 2) |

*model* points at the section of `MODEL.md` that discusses the same thing.
Statements are worded to match it, so the two never drift into two readings.

All probe answers were taken on **2026-09-21**, `zymbol 0.0.9`, through
`zyq show`, in **all three engines**.

---

## 5. The candidates

### C-TYP-1

**Statement** — A value carries a type; a name does not. What a parameter must be
is decided by how the **body** uses it.

**Probes** — `candidates/C-TYP-1/` (2). **What they show is positive**: that a
type is a property of the value. The companion claim — that no annotation syntax
exists — is an absence and is held by the grammar and by `SYM-5`, not by a probe
(§ 2, question 2).

**What the engines answer** — three engines agreeing. `suma(a, b) { <~ a + b }`
called with `(1, 2)` gives `3`, with no type written anywhere. A name takes an
`Int` and then a `String`: the program runs and prints `1` then `dos`, with
`warning: type mismatch` — which is C-TYP-3's subject, not this one's.

**What it needs from the author** — the least contentious of the four, and the one
that fixes the vocabulary for the other three: after it, *"the type of `x`"* is
not a phrase the language has.

---

### C-TYP-2

**Statement** — What inference reaches, it refuses. A parameter's type comes from
how the body uses it; an array is checked element by element against the first.

**Probes** — `candidates/C-TYP-2/` (4).

**What the engines answer** — and **this is where the candidate splits**:

| probe | zytw | zyvm | zyjs |
|---|---|---|---|
| `f(a,b){<~a+b}` · `f(1,"x")` | `error: argument 2 has type String, but function 'f' expects Number` — **static** | same, **static** | `Runtime error: + is arithmetic only` — **at run time**, naming the operator |
| `[1, "dos"]` | `error: array element 2 has type String, but expected Int` — **static** | same | **same, and static** |
| `#[1, "dos"]` | `[1, dos]` | `[1, dos]` | `[1, dos]` |
| `aplica(g,v){<~g(v)}` · `aplica(doble,"hola")` | `Runtime error: arithmetic requires numeric operands` | same | same |

**The split is per rule, not per engine.** Array homogeneity is refused
**statically in all three engines**, so a premise may say so. Argument type is
refused statically in the Rust engines only, so a premise about it may say
*refused* and nothing stronger today.

This corrects the shorter reading — *"`zyjs` has no type system"* — that
`REFERENCE.md` states: it has some, and which part it has is what decides how
strong the premise may be.

**What it needs from the author** — two decisions:

1. Is this one premise or two (*refused* and *refused statically*)?
2. For the argument-type half: is the Rust behaviour **the rule**, making `zyjs` a
   gap to close — or is *refused* the rule and the static half an implementation
   liberty?

The fourth probe is the boundary of the domain: the type of a parameter *of a
parameter* is not reached, and nothing static sees it.

---

### C-TYP-3

**Statement** — A type change on a name is a **warning**, not an error.

**Probes** — `candidates/C-TYP-3/` (2).

**What the engines answer** — three engines agreeing.

```
x = 1
x = "dos"        → warning: type mismatch: 'x' was Int but assigned String
>> x             → dos                       the program runs

x = 1
x = "dos"        → warning: type mismatch: 'x' was Int but assigned String
>> x + 1         → Runtime error: + is arithmetic only
```

**The analyser saw it, said so, and let the program run into the failure it had
just predicted.**

**What it needs from the author — and this blocks the whole domain.** It is the
one place in the language where something the engine can prove is *reported*
rather than *stopped*; everything in `MODEL.md` § 2.2 refuses. Three exits:

1. **Deliberate**, and the reason is recorded — the premise then states the
   warning as the boundary of the discipline and says why it is there.
2. **It should be an error** — the premise says so, the cells go red, and the red
   is declared debt, exactly as MEM-2's four were.
3. **An unexamined default** — then it is not a premise yet, and saying that is
   also an answer.

No premise about typing can be written without answering this: the warning is
where the static half stops.

---

### C-TYP-4

**Statement** — Nothing is coerced. `==` never converts; `&&`, `||` and the unary
operators take only their own type.

**Probes** — `candidates/C-TYP-4/` (4).

**What the engines answer** — three engines agreeing. `"5" == 5` is `#0`.
`7 && 3`, `0 && #1` and `!7` each produce **a static warning and then a run-time
error**: `warning: logical operation on non-boolean type: Int`, then
`Runtime error: logical AND requires boolean operands, got Int`.

So this has the same shape as C-TYP-3 — the analyser knows and does not stop —
and the two are probably one decision.

**What it needs from the author** — whether the premise states *refused* (true
today in three engines) or *refused statically* (false today in every engine).
This was decided as a language rule on 2026-08-30; what it never had is an id.

---

### C-FUN-1

**Statement** — Arity is fixed: no default parameters, no variadic ones, no named
arguments. A call with the wrong count is a static error.

**Probes** — `candidates/C-FUN-1/` (4).

**What the engines answer** — three engines agreeing. Too few and too many are
both `error: function 'f' expects 2 argument(s), but N were provided`. A default
value in the signature is `error: expected ')' after parameters`; a named argument
at the call is `error: expected ')' after function arguments` — both **parse**
errors, so neither form is in the grammar.

**What it needs from the author** — whether the `std/` exception belongs in the
premise or is a defect. `net::get` takes a URL with or without a second argument,
which a user function cannot do.

---

### C-FUN-2

**Statement** — Nothing builds a function out of functions: no composition, no
currying, no partial application. A function is applied, passed, returned or
closed over.

**Probes** — `candidates/C-FUN-2/` (3).

**What the engines answer** — three engines agreeing. `doble . mas1` is
`Runtime error: the dot reaches a dictionary key, and this is Function`, so the
mark a composition operator would want is **already taken** by dictionary access.
`suma(5)` on a two-parameter function is the arity error, not a partial
application. What the language offers instead runs and gives `8`: write the
lambda, `(x -> doble(mas1(x)))`.

**What it needs from the author — is this a rule or a gap?** As a rule it says a
boundary is never assembled out of view; as a gap it says nobody has needed it
yet. This register cannot decide it, and the `.` collision is a fact the decision
should have in front of it, since `SYM-5` governs what would have to happen for
the mark to change.

---

### C-FUN-3

**Statement** — A named function used as a value captures nothing: `MEM-2` leaves
it no outer name to capture.

**Probes** — `candidates/C-FUN-3/` (3).

**What the engines answer** — three engines agreeing. `f = dup` then `f(5)` gives
`10` and `[1,2,3]$> f` gives `[2, 4, 6]`. The form older documentation called
"captures the scope at assignment" — `base = 10` with `adder(n) { <~ n + base }` —
is `error: 'base' is read from outside this function`. The contrast runs: the same
body as a **lambda**, `f = (n -> n + base)`, gives `15`.

**What it needs from the author** — whether this is a premise of its own or a
**consequence of MEM-2** that needs no id. It may well be the second, and saying
so is worth recording: three documents stated the capture rule and all three were
describing an engine that no longer exists.

---

### C-GRW-1

**Statement** — Which door a new capability comes through: a mark, or a `std/`
module.

**State** — `drafted`. **No probe is possible**: this constrains a *proposal*, not
a program, exactly as SYM-1…SYM-7 do (§ 2, question 1).

**What exists today** — two statements, neither with a home, both already reported
by `PREMISES.md` § 6:

1. **«Zymbol no añade funciones que no sean primitivas del lenguaje»** —
   `serpiente/HALLAZGOS_ES.md:372`, refusing a native random number. One version
   later Zofía obtained `std/random`. The two are not in conflict — one refused a
   *language primitive*, the other added a *library module* — but the distinction
   that reconciles them is written in no document.
2. **The symbol-vs-module rubric** (v0.0.7, confirmed by the author): a flow of
   the running process itself — anonymous, ambient, no named arguments — becomes a
   mark; a named operation on an addressed resource that takes arguments becomes a
   `std/` module. Refined in v0.0.8 by `std/term`: an operation on the *content* of
   a string is a mark; one that depends on how a *terminal renders* it is a module.

`SYMBOLS.md` § 17 has eight rules for the mark side and none for the module side.

**What it needs from the author** — unlike the rest of the register, not a
decision but a **statement**. The rubric exists and was confirmed; `PREMISES.md`
§ 6 declines to attribute it because what is on record is a quotation and a
reconstruction.

---

## 6. What is *not* a candidate

A rule that is already declared and that nothing measures needs a **cell**, not an
id (§ 2, question 3).

| what | where it stands |
|---|---|
| **MEM-6's write direction** — a lambda writes only the names it declares, while a block writes through to its container | **Normative since 2026-09-21.** No cell holds it: an engine whose lambda wrote through would pass all 12 cells MEM-6 names. The missing cell has the shape of `write-inside-a-function-does-not-escape`, itself found by reading the premises against the cells rather than by a failure |

## 7. Before promoting one

In the order `HOW_TO_CHANGE_ZYMBOL.md` § 1 requires — **decide → implement →
hold** — and the first step is the one this file cannot take.

1. **Answer the question in the entry**, not the one the probes answer. The probes
   say what the engines do; the entry says what has to be chosen.
2. **Decide the strength**: *refused* or *refused statically*; three engines or
   two. A premise holding in two engines is a premise with declared debt — allowed,
   as MEM-2 lived with four red cells — but it must say so.
3. **Write it in `PREMISES.md`** with a permanent id, an `Original` if the author
   stated it or a `Source` if it was distilled, the `Normative` text, the
   `Measured` line with its date and engines, and `Held by`.
4. **Declare the cells** in `ZyDDT`, or write *"held by review"* and why no cell is
   possible — the honest answer for C-GRW-1.
5. **Run `zyddt premises`**, which crosses ids against cells in both directions.
6. **Commit `PREMISES.md` alone.** § 7: *a premise is changed by the author, in a
   commit that changes nothing else.* This entry becomes a pointer in a separate
   commit.
