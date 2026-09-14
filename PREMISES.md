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
Original    the author's words, verbatim — when the author stated it directly
Source      the document it was distilled from — when it came from prose
Normative   the rule, in the form an engine can be measured against
Measured    what the engines actually do, with the date it was measured
Held by     the cells that turn red if it stops holding — or "nothing yet"
```

`Original` and `Source` are alternatives, and which one an entry carries says
where it came from: a premise the author stated is quoted, a premise distilled
from a document cites it. Distilling is not a downgrade — a 600-line document
cannot be crossed against a cell, and an id can.

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

**Measured** (2026-09-13, after implementation) — **Holds, in all three
engines.** Reading a file variable from inside a named function is a static
error, and a lambda written inside that function cannot reach the file either,
because it inherits the boundary (MEM-6).

**Impact, measured before deciding.** Across 1251 files — 666 corpus, the
playground examples and all nine applications — reaching out of a NAMED FUNCTION
happened in **four files, all four corpus files that test this very rule**, and
in **none of the nine applications**. Reaching out of a LAMBDA happened 68 times,
in code that is written and taught. That asymmetry is what made the two halves
separable, and it is why the first sweep's 240 hits were not the answer: 168 of
them were module functions reading their own module's state, which is MEM-4 and
not a crossing at all.

**History** — it held until **2026-08-24**. `GUIDE.md` § 10b documented the
isolation as deliberate; commit `fbccc8e` retired both, to resolve ZyBank's
`ERROR-ZYB-002`, which had found that the same body behaved differently
depending on how it was reached (a direct call ran isolated; the same function
taken as a value captured). Of the two ways to make that coherent — isolate both
paths, or capture in both — the second was taken, with the argument *"one rule
instead of two"*.

**Held by** — `isolation/file-var-direct`, `isolation/file-var-as-value`,
`isolation/file-var-nested`,
`isolation/lambda-inside-a-function-cannot-reach-the-file`,
`isolation/block-var-*`, `isolation/caller-local-*`,
`isolation/parameter-is-the-door`. All green since 2026-09-13.

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

**Held by** — `modularity/module-does-not-see-its-importer` and
`modularity/two-modules-may-hold-one-name`. Possible since 2026-09-13, when a
cell learned to declare sibling files: a module is a file, and until then this
premise could not be asked at all.

---

### MEM-4 — A module's state is written from inside the module

**Original** — «Las variables de un modulo pueden ser modificadas dentro del
modulo desde una funcion. Este seria el caso mas cercano a una variable global
al modulo.»

**Normative** — A module's variables are modified by that module's own
functions, and only by them. They are not exportable: only constants and
functions leave. This is the closest thing the language has to a global
variable, and the fence is what keeps it from being one.

**Measured** (2026-09-12, **corrected 2026-09-13**) — the state persists across
calls and is carried by the module's own functions, in all three engines. **The
fence does not hold in two of them.**

`#> { n }` where `n` is a variable is `E005: Item 'n' not found in module` under
`zymbol check`, and `zytw` refused it at run time — while **`zyvm` and `zyjs`
printed the value**. Recorded as `GLB-009` and **fixed the same day**: the VM's
export table accepted an `Assignment` as the source of an exported constant, and
the browser engine handed out whatever the name held. All three refuse it now,
with the same wording; what still differs is when — the VM decides while
compiling and the other two while running.

The 2026-09-12 entry said the fence *was* enforced. It was verified with
`zymbol check` and not by running — the same mistake `DM-05` made, and the one
`HOW_TO_CHANGE_ZYMBOL.md` § 2 lists as way out number three. A static error two
engines ignore at run time is not a fence.

**Held by** — `modularity/module-state-is-not-exportable` (**red**: `GLB-009`),
`modularity/module-functions-own-the-state` (green: the legitimate half).

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

A `Held by` names cells as globs over `axis/cell`; `zyddt premises` requires
each one to match at least one cell that declares this same id back.

---

### MEM-6 — There are two levels of environment

**Original** — «tenemos niveles de scope uno ligero y uno fuerte […] `fun () { x
= 1 <~ x }` entorno fuerte, este es un contenedor aislado así como lo sería un
módulo. Imagina si cuando tengamos 4 módulos importados ningún módulo pueda
tener la variable de otro de los 3, sería insostenible en el tiempo. Lo mismo veo
para las funciones. […] `? #1 { x = 1 }` ligero, este no permite que esta
variable exista en su entorno fuerte.»

**Normative** — Two levels, and only two.

A **strong environment** is an isolated container: a module, a named function,
and a lambda. No name enters except as a parameter; no name leaves except as a
return value or through `<~`. What it holds is its own, whatever it is called.

A **light environment** is every block — `?`, `_?`, `_`, `@`, `??` arms, `!?`.
It lives inside a strong one: it reads and modifies that container's names, and
what is born in it dies with it. It opens no new namespace, so assigning a name
its strong environment already holds is a modification of that one name, never a
shadowing copy. For a value to survive the block, the language already has a
mark: `°`.

**A lambda is LIGHT** — decided 2026-09-13, and it replaces the 2026-09-12
reading that made it strong.

**Original** — «podríamos definir las lambdas como scope ligero pero explícito,
ya que el parámetro que se declara es el que se modificará y el que toma de
fuera estará contenido en su scope de alcance previo nunca global».

A lambda opens no boundary of its own. Its parameters are explicit and are the
only thing it declares — the same contract a `@` iterator has — and what it
reads from outside is whatever its **enclosing** scope holds, never anything
global to it.

The consequence is that **its isolation is inherited rather than excepted**:

| written at | reads | because |
|---|---|---|
| file level | the file's variables | it **is** that scope, exactly as a `?` block is |
| inside a function | that function's parameters and locals | that is the scope it sits in |
| inside a function | **not** the file | the function cannot reach the file either, so neither can the lambda |

This is what makes MEM-2 need no exception for it, and it is why it is the right
reading rather than merely the cheap one: the alternative — a lambda strong like
a named function — would make `n$> (x -> x * factor)` **inexpressible**, because
`$>` takes a lambda of exactly one parameter and the language has no other
channel for context. A rule that removes a capability rather than relocating it
is not a rule, it is a loss.

**Measured** (2026-09-13, after implementation) — holds, in all three engines,
**and the cost of enforcing it was zero**: across 1251 files the only one that
errors is `errors/semantic/funcion_lee_el_archivo.zy`, which exists to error.
The 68 lambda warnings the strong reading produced are gone, because they were
never crossings.

`ERROR-ZYB-002`'s incoherence is closed either way: both ways of reaching a
named function behave alike, isolated. That finding was about one body reached
two ways, which was never the same question as whether a lambda may read the
scope it is written in.

**Measured** (2026-09-12, three engines) — **Holds for the light environment and
for the module; does not hold for the function or the lambda.** A block's name
read outside it is a static error in all three engines; a block modifying its
container's name works; sibling blocks with the same name are independent; `°`
anchors above a loop; two modules each holding `x` do not collide (7 / 999), nor
do a module and its importer (5 / 999). The function and the lambda both read
the file's top-level names — see MEM-2.

**Held by** — `isolation/lambda-at-file-level-reads-the-file`,
`isolation/lambda-sees-its-enclosing-function`,
`isolation/light-block-does-not-leak`,
`isolation/light-block-modifies-its-strong`,
`isolation/light-siblings-are-independent`,
`isolation/hot-definition-anchors-above`, and the cells it shares with MEM-2:
`isolation/block-var-*`, `isolation/caller-local-*`.

---

### MEM-7 — One name, one thing, inside one strong environment

**Original** — «scope de variable sin permitir la reutilización de nombres»
(2026-09-11), precisado el 2026-09-12: la prohibición es **dentro** de un
entorno fuerte, nunca entre entornos.

**Normative** — Within one strong environment a name designates one thing. Two
parameters, a variable and a function, or two definitions of one function may
not share a name in the same container. Between different strong environments
the same name is free — if it were not, four imported modules would be
unsustainable, which is the argument MEM-6 rests on.

A light environment introduces no exception: it has no namespace of its own, so
a block cannot hold a second thing under a name its strong environment already
uses.

**Measured** (2026-09-12, after implementation) — **Holds, in all three
engines.** All four forms are static errors, with the same wording everywhere and
a help line that states the rule rather than only refusing.

**Impact of enforcing it: zero.** Before turning the check on, the four forms
were searched for across the whole workspace — 666 corpus files, the 41 refusal
forms, the playground examples and all nine applications. **Not one file used
any of them.** Nothing had to be migrated, which is itself the finding: these
were never shapes anyone wrote on purpose. A model generating code is another
matter, which is why `f(a, a)` mattered enough to refuse.

The state it replaced, kept because it is why the rule is an error and not a
warning:

| form | engines |
|---|---|
| `f(a, a) { <~ a }` then `f(1, 2)` | **`zytw` 2, `zyvm` 1, `zyjs` 2** — accepted, and the answer depends on the engine |
| `f(f) { <~ f }` | accepted |
| `dato = 5` and `dato() { … }` | accepted |
| `f()` defined twice | accepted, the last one wins |

`f(a, a)` is a program whose value is decided by the engine that runs it. No
corpus file writes it, which is why nobody had asked.

**Decided 2026-09-12** — a static error. It did not wait for MEM-2: it was
separable and cheap. **Implemented the same day** in
`crates/zymbol-semantic/src/type_check.rs` (`check_name_collisions`, serving
both Rust engines) and in `web/src/zymbol/zymbol.js` (`checkNameCollisions`).
The four cells went green; `zyq consensus` stayed at 660 agreeing and 0
diverging, and every golden and refusal form held.

**Held by** — `isolation/two-parameters-alike`,
`isolation/parameter-named-as-its-function`,
`isolation/variable-and-function-alike`, `isolation/function-defined-twice`.
All four **red** as of 2026-09-12: they are the declared debt of a decision
taken, not a defect nobody noticed.

---

## 3. Collections

Distilled from `COLLECTIONS.md` on 2026-09-12, which says of itself that it
holds "the rules that govern them, and **why each rule was decided the way it
was** … so the rules do not have to be re-argued from the code". That is the job
of a premise; what it lacked was ids, so nothing could point at it.

### COL-1 — The rule of the result

**Source** — `COLLECTIONS.md` § 1.

**Normative** — One rule, governing the whole editing family across all three
collections. A `$` edit whose result is **used** — assignment, argument, `>>`,
condition, chaining — **builds**, and the original is untouched. A `$` edit whose
result is **discarded** — the `$` is the whole statement — **modifies in place**.

**Measured** (2026-09-12) — holds.

**Held by** — **nothing yet.** The rule has two halves and a cell must ask both
of one edit; no axis crosses them.

---

### COL-2 — `=` never writes into a collection

**Source** — `COLLECTIONS.md` § 2.

**Normative** — `arr[i] = v` does not exist, and neither does `m[i][j] = v` nor
`d["k"] = v`. `=` gives a value to a **name**; `$~` changes part of a
collection. Giving one sign to both is what makes assignment ambiguous about
what it touches.

**Measured** (2026-09-12) — **holds, statically**: `error: indexed assignment
does not exist: 'a[…] =' is not a form of Zymbol`, and the same for a
dictionary.

**Held by** — **nothing yet.**

---

### COL-3 — `[…]` is homogeneous, `#[…]` declares the mix

**Source** — `COLLECTIONS.md` § 3.

**Normative** — `[…]` is checked element by element against the first; `#[…]`
declares a mix and is not checked. **They are the same type**, and every
operator behaves the same on both — `#` is the meta mark, not a second type.

**Measured** (2026-09-12) — **holds, statically**: `[1, "dos"]` is `error: array
element 2 has type String, but expected Int`; `#[1, "dos"]` passes.

**Held by** — **nothing yet.**

---

### COL-4 — The tuple is positional, immutable, and unordered

**Source** — `COLLECTIONS.md` § 4.

**Normative** — A tuple cannot be modified — the check is on the receiver, so
`t[1]$~ v` and `t$+ v` both refuse. Equality compares element by element at
every depth; **ordering does not exist**, because a positional heterogeneous
value has no defensible one.

**Measured** (2026-09-12, both engines) — holds: `cannot modify tuple 't':
tuples are immutable`, and `(1,2) < (3,4)` is `cannot compare values with
operator 'Lt'`.

**Held by** — **nothing yet.**

---

### COL-5 — A dictionary is addressed by key, and only by key

**Source** — `COLLECTIONS.md` § 5.

**Normative** — Every positional form is refused — `d[2]`, `d[-1]`, `d[2]$~ v`,
`d$-[2]`, `d$[1:2]` — and the whole family, not only the read: adding a key
changes what sits at each position, so a program that depended on one would stop
being correct without changing. An absent key is an **error** (`##Key`), never a
silent empty value.

**Measured** (2026-09-12, both engines) — holds: `a dictionary is addressed by
key, not by position`, and `no key 'z' in dictionary — available: x`.

**Held by** — **nothing yet.**

---

### COL-6 — Assignment copies; there is no aliasing

**Source** — `COLLECTIONS.md` § 6.

**Normative** — Binding a collection to a second name gives that name a value,
not a view: writing through one never shows through the other. How the copy is
paid for is not part of the semantics — `Rc` and copy-on-write are invisible by
requirement.

**Measured** (2026-09-12, both engines) — holds: `f = e` then `e[1]$~ 99` leaves
`f` as `[1, 2, 3]`.

**Held by** — **nothing yet**, for the semantics. Its *cost* is held by
`zyquality/cost/`, which is a different claim about the same mechanism.

---

### COL-7 — Nesting is navigated with `>`, and `a[i][j]` does not exist

**Source** — `COLLECTIONS.md` § 5 ("A path may mix the two spellings — but not
two brackets"), and the v0.0.9 decision that retired chained reading.

**Normative** — One bracket group addresses one element however deep it lies:
`m[i>j]`. The chained spelling is refused by name, with the replacement in the
message.

**Measured** (2026-09-12) — holds: `chained index does not exist: 'm[…][…]' is
not a form of Zymbol`.

**Held by** — `chained-index/*` (38 cells) and `addressing/*` (23 cells), both
green. **They already existed**: two axes declared on 2026-09-07 as "the two
halves of one rule" were verifying a design premise all along, and nothing said
so. That is the whole argument for ids — the coverage was real and unattributable.

---

## 4. The sign system

Distilled from `SYMBOLS.md` § 17 on 2026-09-12, which already states eight rules
with "what it forbids, why, and how to check a proposal against it" — everything
a premise needs except an id.

**Most of this domain cannot have cells, and that is a property of the domain
rather than a gap.** MEM and COL constrain what a *program* may do, so a program
can be written that violates them. SYM-1, 2, 3, 5, 6 and 7 constrain what a
*proposal* may be: they are checked when someone argues for a new mark, by a
reader, against the inventory. Only SYM-4 and SYM-8 leave a trace a program can
carry. Saying so here is the point — a domain that quietly reports six premises
with "no cell yet" reads as debt, and this is not debt.

### SYM-1 — Derive, do not invent

**Source** — `SYMBOLS.md` § 17.1. A new operator must be explainable as a
composition of marks already in the inventory; the check is the interlinear
gloss.

**Held by** — review, not execution. `zyddt` cannot ask a program whether an
operator was derived.

### SYM-2 — One abstract meaning per base mark

**Source** — § 17.2. A new use of a mark must fit that mark's contract: `~`
means modification, so a new `~X` must transform something.

**Held by** — review.

### SYM-3 — Context constraints are inherited, not restated

**Source** — § 17.3. If a domain carries a restriction, every new member carries
it: a new `@`-statement acting on the time context is invalid outside a loop,
and that is not a decision to be re-made.

**Held by** — review. Its consequences do have cells: the loop-context rules of
v0.0.9 are what this premise produces.

### SYM-4 — No natural-language words in the grammar

**Source** — § 17.4. Not in English, not in any language. Control flow, types,
operators and declarations are marks. The scope is the *grammar*, not the
lexicon: identifiers are free in any script.

**Measured** (2026-09-12) — holds. `if x > 1 { }` and `f() { return 1 }` are
parse errors, and `!=` is refused by name: `'!=' is not a valid Zymbol
operator`. Worth noting the asymmetry: `!=` gets a diagnostic that states the
rule, the words get `unexpected token: LBrace`, which refuses correctly and
explains nothing.

**Held by** — `refusal/not-equal-is-not-a-symbol`.

### SYM-5 — No new base mark without a documented abstract character

**Source** — § 17.5. The meaning is defined in `SYMBOLS.md` *before* it is
implemented, because "a mark that ships before it is described acquires its
meaning from whatever the first few uses happen to be, and that meaning is then
very hard to correct". The description is the design; the implementation follows
it.

**Held by** — review. This is the premise that ZyAudit's `IDEA-001` (raw
strings) was refused under.

### SYM-6 — No mark may carry two unrelated meanings

**Source** — § 17.6. If two readings cannot be stated as one contract they are
homographs, and a homograph is a defect to be paid down, not a feature to be
documented and forgotten. § 13 lists six of standing debt; `\` / `\\` is the one
worth retiring.

**Held by** — review. The debt is enumerated in `SYMBOLS.md` § 13, which is the
closest thing to a cell this premise can have.

### SYM-7 — Prefer iconic over conventional

**Source** — § 17.7. Between two derivable forms, the one whose shape depicts
its meaning: `<>` rather than `!=`, `><` rather than a lettered form.

**Held by** — review.

### SYM-8 — Modality goes last

**Source** — § 17.8. A modal `?` or `!` is the rightmost mark of an operator; an
argument or label never follows it. This is why labelled break is `@:outer!` and
not `@!outer`.

**Measured** (2026-09-12) — holds: `@:outer!` parses, `@!outer` does not — it is
read as a bare break followed by an expression, and fails with `undefined
variable 'outer'`. Correct refusal, and again a message that does not state the
rule it is enforcing.

**Held by** — **nothing yet**; unlike its six siblings this one *can* have a
cell, and should.

---

### MEM-8 — Memory is released automatically and invisibly, or destroyed by hand

**Original** — «auto limpieza de memoria o borrado manual de memoria»
(2026-09-11), declared as a premise 2026-09-13.

**Normative** — Two mechanisms, and the difference between them is
observability.

**Automatic release is invisible.** A value is released after its last use, and
that it was released changes nothing a program can observe: a correct program
behaves identically with the mechanism and without it. Anything the analysis
cannot decide is not released — hot definitions, constants, `_` names, the free
variables of a function used as a value, and every module-level binding — so the
conservative direction is always the silent one.

**Explicit destruction is the exception, and it is observable on purpose.**
`\ x` ends a name's life where it is written, and using the name afterwards is
an **error**. That is the whole point of having it: a programmer who writes `\`
is making a statement about lifetime, and a statement nobody can be wrong about
is not a statement.

**Measured** (2026-09-13) — the automatic half holds, and `zyquality/cost/`
measures it: two aggregates used one after the other peak at the cost of one
(0.51 in the tree-walker, 0.55 in the VM), while the same two alive at once peak
at two. The 634 goldens are the other half of that evidence — releasing early has
never changed an answer.

The explicit half **holds too, since 2026-09-13** — and fixing it corrected the
premise's own reading of what «error» means here.

It must be a **run-time** error, and that is not a concession to how the engines
happen to work. A `\` inside a branch that never runs destroys nothing, so a
static check without flow analysis cannot tell a destruction that HAPPENED from
one that was merely written down. `zyjs` was refusing `? #0 { \ x }` followed by
`>> x`, which both Rust engines print. The late answer is the correct one.

`GLB-008`, closed. It took three fixes, and the third was invisible to the first
measurement because that one read only the first line of the output: **`\` did
nothing at all in the VM** for a file variable, since dropping the register
binding left the value reachable in `global_vars`.

**Held by** — `lifetime/*`; the automatic half by `zyquality/cost/autofree/*`,
which is a claim about cost rather than about behaviour and cannot be a cell.

---

## 5. Generated code

Distilled on 2026-09-12 from `interpreter/AGENTIC.md`, which measured these
against `zymbol 0.0.9` and stated them as observations. Five of its eleven
sections turned out to enunciate rules **no document declares**, which is why
they are here; the other six were re-stating MEM, COL and SYM in prose, and that
document now cites them by id instead.

The domain exists because code written by a model fails differently: it does not
forget a rule, it never had it, and fills the gap with the most probable shape
from another language. What matters is therefore not that correct code be easy
to write but that **incorrect code be impossible to write silently**.

### AGT-1 — Importing executes nothing

**Source** — `AGENTIC.md` § 1.

**Normative** — A module body admits only imports, the export block,
literal-initialised bindings and function definitions. Anything that computes is
refused **before a statement runs**. There is no install-time and no import-time
code, in any form.

**Measured** (2026-09-11, `zymbol check`) — holds: `E013: executable statement
not allowed in module body`, reported through the import with `note: reached
from`.

**Held by** — `modularity/import-executes-nothing`. Possible since 2026-09-13:
a cell can declare sibling files, so a module body that computes can finally be
written down and refused. Until then this — the property that removes the whole
class of install-time and import-time code — was defended by nothing.

### AGT-2 — A module declares its surface

**Source** — `AGENTIC.md` § 2.

**Normative** — Omitting the export block is an error, not an implicit "export
everything". An empty `#> { }` says it exports nothing, and saying so is
required.

**Measured** (2026-09-11) — holds: `E014: module 'm' does not declare what it
exports`.

**Held by** — `modularity/module-declares-its-surface`,
`modularity/undeclared-item-does-not-leave`.

### AGT-3 — The reachable program is knowable before running it

**Source** — `AGENTIC.md` § 3.

**Normative** — Four things together: a `<#` path is a literal, imports precede
every statement, `</ … />` takes its path literally, and **there is no `eval`**.
Therefore reading the entry file and following `<#` has read everything that can
run — a property no mainstream scripting language offers, and the one that makes
reviewing generated code finite work.

**Measured** (2026-09-11) — holds: a variable as an import path is `imports must
come before any statement`; `p = "./x.zy"` then `</ p />` is `file not found:
p`, because `p` *is* the path; no form executes a string as code.

**Held by** — **nothing yet.** The `eval` half cannot have a cell at all: there
is no program that tests the absence of a feature. The other three can.

### AGT-4 — The effect surface is lexical and finite

**Source** — `AGENTIC.md` § 4.

**Normative** — Every way a program reaches outside itself has its own mark or
its own import, and the list is closed: `std/io`, `std/net`, `std/db`,
`std/time`, `std/random`, `<\ … \>`, `</ … />`, `<<`, `><`. Auditing what a
program may do is therefore a search over a closed list and not a
flow-sensitive analysis — and with AGT-3, the audit is complete, because no gate
can be reached through a name computed at runtime.

**Measured** (2026-09-11) — holds as an audit. **It is not enforced**: nothing
restricts a program from using any of them (`AGENTIC.md` G1, open). The premise
as written claims auditability, which is what holds; enforceability is a
separate decision nobody has taken.

**Held by** — **nothing yet.**

### AGT-5 — A package is source, never a binary

**Source** — `AGENTIC.md` § 11.

**Normative** — A `.zyp` is a ZIP of `.zy` files and its manifest: readable,
diffable, auditable. Running one extracts to an ephemeral directory and **never
`chdir`s**, so code is disposable while the data a script writes lands in the
user's real working directory.

**Measured** — holds by construction; `zymbol-package` never compiles or
executes Zymbol code.

**Held by** — **nothing yet.**

---

## 6. Stated but not yet declared

Things the author has said, which are **not** premises until restated here as
one. They are listed so that "nobody wrote it down" never becomes the reason
something was lost.

| what was said | measured 2026-09-12 |
|---|---|
| ~~«scope de variable sin permitir la reutilización de nombres»~~ | **declared 2026-09-12 as MEM-7**, once the boundary was precise: the prohibition is inside a strong environment, not between them. Two of the four forms I had listed as defects are the model working — an inner assignment mutating the outer name (one `x`, no shadowing) and a parameter reusing a file-level name (different containers) |
| ~~«auto limpieza de memoria o borrado manual de memoria»~~ | **declared 2026-09-13 as MEM-8.** Promoting it found the divergence its own absence had hidden: the three engines refuse a use after `\`, at different times and with different words |

### The rule that only exists in a gap log

> **«Zymbol no añade funciones que no sean primitivas del lenguaje»**
> — `serpiente/HALLAZGOS_ES.md:372`, 2026, on refusing a native random number.

It is a design decision, stated as one, and it lives **in the findings log of a
snake game and nowhere else**. One version later Zofía obtained `std/random`,
and `std/math` with it.

The two are not in conflict — one refused a *language primitive*, the other
added a *library module* — but the distinction that reconciles them is written
in no document. There is a rubric for it (v0.0.7, "when a capability becomes a
mark and when it becomes a `std/` module"); it exists as a decision and has no
home. `SYMBOLS.md` § 17 governs what may become a **mark** and says nothing
about what may become a **module**, so the question of which door a capability
comes through has eight rules on one side and none on the other.

This is the clearest single argument for `LDV.md` living in this repository:
**a method that were only validation would not leave design rules in its logs
that exist nowhere else.** It does.

Not declared as a premise because the author has not stated it as one — what is
above is a quotation and a reconstruction, and reconstructing a rule and then
attributing it is exactly what § 6 forbids.

### Open questions this domain raises

- ~~Is a lambda a self-contained space, like a function?~~ **Decided
  2026-09-12: it is strong.** See MEM-6.
- **Does MEM-2 apply to reading, to writing, or to both?** Today the write side
  is isolated and the read side is not; the premise as written covers both
  ("vistas o modificadas"), which is the reading this document takes.

---

## 7. The rule that protects this file

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
