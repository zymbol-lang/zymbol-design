# Zymbol — the computational model

> **Rank: source.** Like `PREMISES.md`, this document is not derived from any
> engine. It states what kind of language this is and what organises it — the
> half no other document holds: `SYMBOLS.md` governs the notation,
> `COLLECTIONS.md` and `MEMORY_MODEL.md` govern two domains, and `PREMISES.md`
> holds the rules an engine can be measured against.
>
> **It declares nothing.** No statement here is a premise. § 8 lists what this
> document states that no premise declares, and `CANDIDATES.md` is where those
> wait for the author's decision.
>
> **Three layers, kept apart.** Every claim below is **normative** (a premise,
> cited by id), **measured** (a fact about the engines, dated, with the engines
> named), or **interpretation** (why those facts are one thing). § 3 separates
> them structurally: every boundary is *Rule / Evidence / Why it is the same
> rule*.
>
> **The editorial rule of this document.** *A reason is not a rule.* An
> explanation of why a decision was taken does not become a property of the
> language by being written down, and does not acquire a battery of tests by
> being interesting. Where a section exists only to justify a choice, it is a
> paragraph and it stays one. This rule is here because the previous draft broke
> it: a one-line reason for not adopting an object model grew into an
> enumeration, then into formal claims, then into probes trying to demonstrate
> the absence of a feature.
>
> **Language.** English. The author's words are quoted in Spanish only where the
> quotation carries a distinction a summary would lose.

---

## 1. What Zymbol is

**A modular imperative language organised by one rule about boundaries.**

The ordinary part first, because it is true and it is not the point: Zymbol has
statements, assignment, conditionals, loops, functions and modules. A programmer
from any imperative language recognises every construct behind the marks — *"a
symbolic vocabulary, not a new paradigm"* (`interpreter/README.md`).

What is not ordinary is what a construct may reach. Every paradigm label fits
partially and the partial fits do not overlap (§ 7). That is not indecision:
those labels answer *how is computation expressed*, and Zymbol answers it
ordinarily. The language was designed on a different axis, which is § 2.

---

## 2. The rule of the boundary

### 2.1 The rule

> **Nothing crosses a boundary invisibly. What crosses is written where it
> crosses — at both ends when the crossing has two — or it cannot change
> anything.**

A *boundary* is any container the language recognises: the file, a block, a
function, a lambda, a module, the life of a value, the program itself.

The second clause is not an exception list; it is what lets the first survive
contact with the language:

- **Marked.** A parameter is named at the call and at the signature. One that
  comes back modified carries `<~` at both, and each half is an error without the
  other. A module says what leaves with `#>`. Every effect has its own mark or
  its own import.
- **Or unable to change.** A constant is readable at any depth and never
  writable. A lambda reads its enclosing scope as a snapshot: it cannot write
  back, and a later change outside does not reach it. Binding a collection to a
  second name gives a value, not a view.

Two ways through a wall, and no third.

### 2.2 Why this is a rule and not a label

A principle that explains everything explains nothing. This one is falsifiable:
a construct that let something cross invisibly **and** change something would
break it. The obvious candidates are all refused, each on its own recorded
grounds and each decided at a different time:

| a crossing that would be invisible and able to change something | what Zymbol does | held by |
|---|---|---|
| a function reading the file it is written in | `error: 'g' is read from outside this function` | MEM-2 |
| `<~` at the signature but not at the call | static error naming the missing half — and the reverse | MEM-5 |
| two names viewing one collection | does not exist: assignment copies | COL-6 |
| `eval`, or an import path computed at run time | no form takes a string and runs it; an import path is a literal | AGT-3 |
| code that runs when a module is imported | `E013: executable statement not allowed in module body` | AGT-1 |
| a module exporting everything by omission | `E014: module does not declare what it exports` | AGT-2 |
| `arr[i] = v` — one sign for "bind a name" and "write into a structure" | `error: indexed assignment does not exist` | COL-2 |
| a value silently becoming another type | no coercion anywhere; `"5" == 5` is `#0` | § 5 |

That they are all the *same* refusal is this document's claim, and it is
interpretation. The right-hand column is fact.

### 2.3 What the rule does not govern

**The sign system.** SYM-1…SYM-8 are about what a *notation* may become, not
about what a program may reach, and `SYMBOLS.md` § 17 governs them completely.
Zymbol has two design axes and this document covers one. Stretching this rule
until it covered the other would be the failure § 2.2 exists to prevent.

### 2.4 What the axis is for

The author, 2026-09-21, kept verbatim because it is the sentence that says what
the axis buys:

> «En conjunto, las cinco apuntan a una misma idea: reducir el contexto implícito
> y el espacio de errores que puede manejar el programador —y, especialmente, una
> IA— al modificar código.»

Distilled: the design optimises for **how much of a program must be read to know
what a fragment does**, and spends expressiveness to buy it (§ 4). `PREMISES.md`
§ 5 reached the same place for one domain — *"incorrect code impossible to write
silently"* — and § 2.1 is what makes *silently* precise.

---

## 3. The five boundaries

The author named five properties of the design (2026-09-21): restricted global
memory, strong and light scope, auto-free, explicit function context, and modules
with no mandatory object model.

They are one rule stated at the five places something can cross: **the global
space, the name, time, the call, and the component.**

### 3.1 The global space — what may be shared, and from where

**Rule.** One global thing, and it cannot vary: a file-level constant (`:=`) is
readable inside any function at any call depth and never writable; reassignment
is a static error (MEM-1). A module's variables are written **only by that
module's own functions** and never exported — only constants and functions leave
(MEM-4).

**Evidence** (2026-09-12, three engines). `LIMITE := 99` is read inside a function
body; `PI = 3` after `PI := 3.14` is `error: cannot reassign constant 'PI'`. A
module's state persists across calls, carried by its own functions.

**Why it is the same rule.** A constant crosses every boundary and is marked at
none — the purest case of the second clause. It is allowed through because it
cannot change. Module state is the mirror: it can change, so it does not cross at
all. What is removed is not shared state but **shared state nobody declared**.

### 3.2 The name — where it exists, and who may write it

**Rule.** Two levels, and only two (MEM-6). A **strong environment** — a module, a
named function — is an isolated container: no name enters except as a parameter,
none leaves except as a return value or through `<~`. A **light environment** —
every block (`?`, `_?`, `_`, `@`, `??` arms, `!?`) and the lambda — opens no
namespace of its own: it reads its container's names and what is born in it dies
with it. Assigning a name the strong environment already holds modifies that one
name; it is never a shadowing copy, and a value that must survive the block is
marked `°`. Within one strong environment a name designates one thing; between
environments the same name is free (MEM-7).

The two light members differ in one direction:

| | reads its container | writes its container | names born in it |
|---|---|---|---|
| a block | yes | **yes** | die with it |
| a lambda | yes | **no** — it writes only the names it declares | die with it |

**Evidence** (reading half 2026-09-13, writing half 2026-09-21; three engines
agreeing):

```zymbol
c = 0
r = [1,2,3]$> (x -> { c = c + x  <~ x })
>> c ¶                                      // → 0   a lambda's write stays inside

c = 0
? #1 { c = c + 5 }
@ i:1..2 { c = c + 1 }
>> c ¶                                      // → 7   a block's write goes through
```

For reading: a file-level lambda reads the file (`factor = 3` gives `[3, 6, 9]`);
a lambda inside a function reads that function's parameters (`[5, 10]`); the same
lambda reading a file variable is `error: 'tope' is read from outside this
function`; a name born inside a lambda is gone after it. All four forms of MEM-7
are static errors, and enforcing MEM-7 changed **no file** across 666 corpus
files, the playground examples and nine applications.

**Why it is the same rule.** Shadowing is a crossing with no mark: an inner name
that silently hides an outer one changes which value a later line reads, and
nothing says so. Refusing it costs a word.

The lambda is the case the model had to be corrected for, and it ends up as
evidence for the model rather than against it. It reads outward with no mark,
which looks like a violation until the second clause applies: the read is a
**snapshot at creation**. Measured 2026-09-21, three engines: `a = 5`, lambda
created, `a = 99`, the lambda still answers `5`. What crossed inward cannot
change the outside, and the outside cannot reach in.

> «podríamos definir las lambdas como scope ligero pero **explícito**, ya que el
> parámetro que se declara es el que se modificará y el que toma de fuera estará
> contenido en su scope de alcance previo nunca global» (2026-09-13, MEM-6)

**Light on reading and lifetime, explicit on writing.** The lambda was listed as
*strong* when MEM-6 was first written (2026-09-12) — an error of precision,
corrected in the engines and cells on 2026-09-13 and in the premise's normative
text on 2026-09-21. The correction matters here for one reason: **the corrected
reading is the one § 2.1 predicts, and the wrong one was the one that had to be
argued for.**

### 3.3 Time — how long a value lives

**Rule.** Two mechanisms, differing in observability (MEM-8). **Automatic release
is invisible**: a value is released after its last use and a correct program
behaves identically without the mechanism; anything the analysis cannot decide is
not released. **Explicit destruction is observable on purpose**: `\ x` ends a
name's life where it is written, and using it afterwards is a run-time error.

**Evidence.** `zyquality/cost/` measures the automatic half as a ratio against a
control: two aggregates used one after the other peak at the cost of one (0.51
tree-walker, 0.55 VM); the same two alive at once peak at two. The 634 goldens
are the other half — releasing early has never changed an answer. The explicit
half must be a **run-time** error: a `\` inside a branch that never runs destroys
nothing, so a static check without flow analysis cannot tell a destruction that
happened from one merely written down.

**Why it is the same rule** — and this is where the unification is weakest, so it
is stated plainly. Explicit destruction is an ordinary application: a lifetime
that ends is a boundary, `\` is its mark. **Automatic release is not a crossing
at all** — it is the absence of a context other languages create.

What the two together do establish is a constraint the rest of the model relies
on: **the implementation may be invisible only where it is unobservable.** That is
why auto-free must change nothing a program can observe, and it is the same
requirement COL-6 places on how a copy is paid for — `Rc` and copy-on-write are
invisible *by requirement*.

### 3.4 The call — what a function may reach

**Rule.** A function is a self-contained space: a value crosses into it as a
parameter and never by being in view (MEM-2). A parameter that may come back
modified is marked `<~` at the signature **and** at every call site, each half an
error without the other (MEM-5); `p~` marks a working copy the caller never sees.
Beyond the call: every way a program reaches outside itself has its own mark or
its own import and the list is closed (AGT-4), and no gate is reachable through a
name computed at run time (AGT-3).

**Evidence** (2026-09-12, 2026-09-21):

```text
g = 100
t() { <~ g + 1 }

error: 'g' is read from outside this function
  = help: a function is a self-contained space: a value crosses into it as a
    parameter, never by being in view — pass 'g' as one
```

Both halves of MEM-5 are static errors, each naming the other. A `<~` slot given
an expression rather than a variable is refused: there is nowhere to write back.
Enforcing MEM-2 cost four files across 1251, all four tests of this rule.

**Why it is the same rule.** The home case, and the one the diagnostic states out
loud. Its consequence pays for the axis: **the cost of reviewing a call is the
call.** A language could isolate a function and still make you read it; `<~` at
both ends removes that second read.

### 3.5 The component — modules, and why not objects

**Rule.** A module is not a class (MEM-3). It has its own environment,
self-contained in both directions: the module does not see its importer's scope
and the importer does not see the module's variables; identity is the file path.
Importing executes nothing — a module body admits only imports, the export block,
literal-initialised bindings and function definitions (AGT-1). Omitting the export
block is an error, not an implicit "export everything"; `#> { }` says it exports
nothing and saying so is required (AGT-2).

**Evidence** (2026-09-11, 2026-09-12). A module body naming its importer's
variable is `undefined variable`, reported through the import with `note: reached
from`. A module body that computes is `E013`. A module with no export block is
`E014`. Two modules each holding `x` do not collide.

**Why it is the same rule.** Most modern languages answer this boundary with an
object model. **Zymbol does not adopt one** — not because objects are wrong, but
because an object model introduces a structure this design does not need, and the
design is minimal and restrictive about context on purpose. Concretely that means
no classes or instances, no inheritance, no dispatch, no receiver and no `self`;
`mod::f(x)` is a call to a named function in a named environment, not a message
to a value. That sentence describes what the decision means — it is not a set of
propositions to be demonstrated.

The conceptual problem being avoided is the one Armstrong named:

> *"The problem with object-oriented languages is they've got all this implicit
> environment that they carry around with them. You wanted a banana but what you
> got was a gorilla holding the banana and the entire jungle."*
> — Joe Armstrong, co-author of Erlang, in *Coders at Work* (2009)

Implicit environment is what crosses; the gorilla is how much. So Zymbol
separates state from behaviour with modules and stops there: data travels as
collections, behaviour as functions, and the language never fuses the pair into a
third thing. AGT-1 and AGT-2 are the same boundary applied to *when* and *what*.

---

## 4. What the axis costs

Each item reads as an omission to someone who does not know the axis, which is
why it is written down.

| given up | bought | held by |
|---|---|---|
| a function reading the file it is written in | the call site is the complete input surface | MEM-2 |
| shadowing | one name, one definition, inside a strong environment | MEM-6, MEM-7 |
| `eval`, computed import paths, import-time code | the reachable program is knowable by reading | AGT-1, AGT-3 |
| an object model, with everything it makes easy | no implicit environment travels with a value | MEM-3 |
| aliasing | a write through one name is never a surprise through another | COL-6 |
| `arr[i] = v` | `=` means exactly one thing | COL-2 |
| currying, composition, partial application | a function is applied or passed, never assembled out of view | § 6 |

**Each cost is a capability whose value depended on being invisible.** Aliasing is
useful only because the second name sees the change; shadowing only because the
inner name wins silently. The language gives up what the rule forbids, and
nothing else.

---

## 5. The typing discipline

**Not dynamic typing, not static typing, and not gradual typing** in the technical
sense — that term names a system with annotations and explicit boundaries between
typed and untyped regions, and Zymbol has neither.

**One premise declares part of this**: TYP-2 (2026-10-06) — what inference
reaches about a parameter is refused before the program runs, and inference
reaches only what every path through the body requires. The rest is measured or
interpreted, never normative; the candidates left are `C-TYP-1`, `C-TYP-3` and
`C-TYP-4` in `CANDIDATES.md`.

### 5.1 The four rules, measured

Measured 2026-09-21 against `zymbol 0.0.9`, with the engine named where they
differ.

**1. A value carries a type; a name does not.** No annotation syntax anywhere —
not on a variable, not on a parameter, not on a return.

**2. What inference reaches, it refuses.** A parameter's type comes from how the
**body** uses it:

```zymbol
f(a, b) { <~ a + b }
>> f(1, "x")      // error: argument 2 has type String, but function 'f' expects Number
```

Same for array homogeneity — `[1, "dos"]` is `error: array element 2 has type
String, but expected Int`, while `#[1, "dos"]` passes because the mix is declared
(COL-3) — and for arity, which is fixed.

**3. What inference does not reach is decided at run time, and a type change on a
name is a warning.** `x = 1` followed by `x = "dos"` prints `warning: type
mismatch: 'x' was Int but assigned String` and runs.

**4. Nothing is coerced.** `"5" == 5` is `#0`. `&&` and `||` take only `Bool`:
`7 && 3` and `0 && #1` are both refused, because a `0` is a number and not a
false; `!7` and `-"a"` likewise (decided 2026-08-30). Measured precisely: these
produce a **static warning followed by a run-time error**, not a static error.

The type is inspectable at run time — `v#?` gives `(type symbol, count, value)` —
and run-time failures carry a kind that is itself a type symbol: `##Type`,
`##Index`, `##Key`, `##Div`, `##Range`.

### 5.2 Refused, and refused statically, are different rules

Measured 2026-09-21, and **the split is per rule, not per engine**:

| | zytw | zyvm | zyjs |
|---|---|---|---|
| `f(1, "x")` against `f(a,b){<~a+b}` | static | static | **static** since 2026-10-05 (ZYJS-048); at run time before, naming the operator |
| `[1, "dos"]` | static | static | **static** |

All three refuse both. Array homogeneity is refused **statically in all three
engines**; argument type was refused statically in the Rust engines only, until
2026-10-05, when `zyjs` got the Rust analyser's parameter inference (ZYJS-048) and
the author decided the static refusal is the rule (TYP-2).

**A property of the Rust engines is not a property of Zymbol** — and the converse
trap is just as real: `REFERENCE.md` says the browser engine *"has no type
system"*, and the second row shows it has some. Any premise distilled from § 5.1
has to choose which rule it states, rule by rule.

### 5.3 The one place the language sees and does not stop

Rule 4 is § 2.1 in the type domain: a value silently becoming another type is a
change nothing in the source shows.

**Rule 3 is the exception.** A name whose type changes is exactly the situation
the rule describes — the analyser has *seen* it and says so — and the language
declines to refuse it. Measured: `x = 1`, `x = "dos"`, `>> x + 1` emits the
warning and then dies with `Runtime error: + is arithmetic only`. The analyser
predicted the failure and let the program run into it.

Whether that is deliberate is recorded nowhere, and until it is, no premise about
a name whose type changes can be written: the warning is where the static half
stops. Registered as `C-TYP-3`. TYP-2, about what a call may pass, did not have to
wait for it.

---

## 6. What a function is, and what it is not

Functions are first-class and deliberately small — the author's characterisation,
distilled: basic, and neither complex nor complete. Measured 2026-09-21:

| | |
|---|---|
| a named function is a value | `f = dup` then `f(5)` and `[1,2,3]$> f` both work. It captures **nothing**: MEM-2 leaves it no outer name to capture |
| a lambda reads its enclosing scope, by snapshot | `mk(n) { <~ (x -> x + n) }` then `mk(10)(5)` is `15`; a variable changed afterwards does not reach it |
| a lambda does **not** reach the file from inside a function | it inherits the boundary rather than excepting it (MEM-6) |
| a named function goes into a higher-order slot **by name** | `nums$> double` works — bare, since `(` opens a lambda: `nums$> (double)` is `error: expected '->' in lambda expression` |
| arity is fixed | no defaults, no variadic, no named arguments. Some `std/` functions are variadic; user functions are not |
| nothing builds a function out of functions | no currying, no partial application, no composition operator — and `.` is already dictionary access |
| there are output parameters | `<~`, marked at both ends (MEM-5) — a procedural feature, not a functional one |

**Why *functional* is the wrong label in both directions.** The language has map,
filter and reduce, and neither the calculus underneath them nor the purity above.
A function can be named, passed, returned and closed over its enclosing scope, and
that is the whole list. `COL-1` is the closest it comes to a functional rule, and
it is a rule about **both** modes at once: a `$` edit whose result is used builds,
one whose result is discarded modifies in place.

Whether the absence of composition is a **rule** or a **gap** is not decided;
registered as `C-FUN-2`.

---

## 7. The labels

Descriptive, and last.

| label | what it shares | where it stops |
|---|---|---|
| **COBOL** | nothing structural. Its real trait is English-as-syntax, which SYM-4 forbids in any language; verbosity is the symptom | — |
| **Lisp** | functions as values | no homoiconicity, no macros, no `eval` (AGT-3) |
| **Simula / OOP** | modules hold state and behaviour near each other | not adopted, and § 3.5 says why |
| **Pascal / Modula-2** | the closest structural relative: modules, explicit interfaces, in/out parameters | Pascal declares every type; Zymbol declares none |
| **APL** | one mark per construct | APL's glyphs are an array calculus; Zymbol's cover ordinary constructs |
| **"modular-functional"** | modules are the unit; the collection operators are functional in shape | the functions are not a calculus (§ 6), and mutation is a first-class mode |

Lineage, where there is one: module structure from the Modula-2 tradition,
collection operators from the functional tradition, notation from mathematics
rather than APL, and a memory model borrowed from nothing.

---

## 8. Formal status

**This document creates nothing in layer A.**

**A — declared and measurable.** MEM-1…MEM-8, COL-1…COL-7, SYM-1…SYM-8,
AGT-1…AGT-5 in `PREMISES.md`, crossed against their cells by `zyddt premises`.
Every **Rule** paragraph in § 3 cites these and states nothing beyond them.

**B — measured, not declared.** Registered in `CANDIDATES.md`, each with probes
and the answer all three engines give: `C-TYP-1`…`C-TYP-4` (§ 5) and
`C-FUN-1`…`C-FUN-3` (§ 6). Named here, not restated — a fact kept in two places
drifts.

One measured fact is **not** a candidate because it is already normative: a
lambda's write does not reach its container while a block's does (§ 3.2). MEM-6
states it; no cell holds it. That is a missing cell, not a missing premise.

**C — interpretation, and it stays here.** The rule of § 2.1; that the eight
refusals of § 2.2 are one refusal; that the five properties are five applications
of it; that each cost in § 4 is a capability whose value depended on invisibility;
that Zymbol has two design axes and this document covers one. None is measurable
against an engine, and moving any of them to `PREMISES.md` would create a rule
nothing can contradict.

**D — open, and blocking something.**

| open | what it blocks | registered as |
|---|---|---|
| is the type-change warning deliberate? (§ 5.3) | every premise about typing | `C-TYP-3` |
| *refused*, or *refused statically*? (§ 5.2) | the same, rule by rule | `C-TYP-2` |
| is the absence of composition a rule or a gap? (§ 6) | whether `FUN-2` can exist | `C-FUN-2` |
| the lambda's write direction has no cell | an engine whose lambda wrote through would pass all 12 cells MEM-6 names | `CANDIDATES.md` § 5 |
| which door a new capability comes through | the v0.0.7 symbol-vs-module rubric is homeless; `SYMBOLS.md` § 17 has eight rules for the mark side and none for the module side | `C-GRW-1` |

Two limits of the evidence, so they are not mistaken for strength: the type-change
warning is measured **only in the tree-walker**, and the closure of § 2.2 is held
by review rather than execution — no program demonstrates the absence of a
feature.

---

## 9. How these limits are found

`LDV.md` § 1 separates *verification* — was the product built right, which is what
every automated suite asks — from *validation*: was the right product built. § 2
states what green means for the second: **the language changes.**

```text
LDV finds the gap  →  the author decides  →  a premise gets an id  →  a cell holds it
└─────────── this document states the gap as a model ───────────┘
```

Every entry in § 8's layer D arrived that way rather than from a failing test, and
three were found by **reading the premises against the cells** — which is how
`write-inside-a-function-does-not-escape` was found on 2026-09-13 and the lambda's
write direction on 2026-09-21. A suite cannot report a question nobody asked it.

The rule this repository protects is in `PREMISES.md` § 7: **a premise is changed
by the author, in a commit that changes nothing else** — which is why this
document declares none.

---

## 10. Related

| document | what it holds |
|---|---|
| `PREMISES.md` | the measurable half — MEM, COL, SYM, AGT, by id |
| `CANDIDATES.md` | layer B and D: id, probes, what the engines answer, the decision each waits for |
| `MEMORY_MODEL.md` | § 3.1–3.4 audited against the implementations |
| `COLLECTIONS.md` | what a value is, and why each collection rule was decided as it was |
| `SYMBOLS.md` · `SIMBOLOS_ES.md` | the other axis: the notation and the rules governing its growth |
| `LDV.md` · `WHAT_LDV_BUILT.md` | how the gaps are found, and what nine applications put into the language |
| `HOW_TO_CHANGE_ZYMBOL.md` | what to do before changing any of it |
