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

**Held by** — `isolation/const-*`, `isolation/an-unused-constant`.

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
`isolation/write-inside-a-function-does-not-escape`,
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

**Held by** — `modularity/module-state-is-not-exportable`,
`modularity/module-functions-own-the-state` (the legitimate half). Both green,
`GLB-009` fixed on 2026-09-13.

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
`isolation/output-mark-without-an-output-parameter`,
`isolation/output-argument-must-be-a-variable`.

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

**Original** (the lambda, 2026-09-13) — «podríamos definir las lambdas como
scope ligero pero explícito, ya que el parámetro que se declara es el que se
modificará y el que toma de fuera estará contenido en su scope de alcance previo
nunca global».

**Normative** — Two levels, and only two.

A **strong environment** is an isolated container: a module and a named
function. No name enters except as a parameter; no name leaves except as a
return value or through `<~`. What it holds is its own, whatever it is called.

A **light environment** is every block — `?`, `_?`, `_`, `@`, `??` arms, `!?` —
**and the lambda**. It opens no namespace of its own: it lives inside a strong
environment, reads that container's names, and what is born in it dies with it.
Assigning a name its strong environment already holds is a modification of that
one name, never a shadowing copy. For a value to survive the block, the language
already has a mark: `°`.

Its two members are alike in what they read and in how long their names live.
They differ in **one direction, and that difference is declared rather than
incidental**:

| | reads its container | writes its container | names born in it |
|---|---|---|---|
| a block | yes | **yes** | die with it |
| a lambda | yes | **no** — it writes only the names it declares | die with it |

That row is what «ligero pero **explícito**» names. A lambda reaches out for a
value the way a block does, and the only name it modifies is the one it declared,
so nothing it does travels back out of it unannounced.

**Why the lambda is light and not strong.** It opens no boundary of its own, so
the isolation it has is the one it **inherits** from the strong environment it is
written in:

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

**Measured** (2026-09-21, the write direction, three engines agreeing) — holds,
and it is the half nothing had asked. The contrast is the evidence:

```zymbol
c = 0
r = [1,2,3]$> (x -> { c = c + x  <~ x })
>> c ¶                                     // → 0   the lambda's write stays inside

c = 0
? #1 { c = c + 5 }
@ i:1..2 { c = c + 1 }
>> c ¶                                     // → 7   the block's write goes through
```

and the three reading rows above, also in all three engines: a file-level lambda
reads the file (`[3, 6, 9]`), a lambda inside a function reads that function's
parameters (`[5, 10]`), and the same lambda reading a file variable is
`error: 'tope' is read from outside this function`. A name born inside a lambda
is gone after it (`error: undefined variable 't'`).

`ERROR-ZYB-002`'s incoherence is closed either way: both ways of reaching a
named function behave alike, isolated. That finding was about one body reached
two ways, which was never the same question as whether a lambda may read the
scope it is written in.

**History** — the lambda was listed as a **strong** environment when this premise
was first written, on 2026-09-12. That was an **error of precision, not a rule
that was later reversed**: the author's own reading had always been «ligero pero
explícito», and the entry said so from 2026-09-13.

What was done on 2026-09-13 was to correct the engines and the cells, and to
*append* the corrected reading to this entry. What was not done was to rewrite
the rule it replaced. For nine days the entry therefore stated both readings at
once, and the **`Normative` paragraph — the part an engine is measured against —
carried the wrong one**, while the implementation, `ZyDDT/axes/isolation.toml`
and the prose below it all carried the right one.

A correction that lands everywhere except in the sentence being measured is its
own failure mode, and it is the mirror of the one § 7 exists to catch: there the
rule was quietly moved to match the engines, here the rule was left behind while
everything else moved. Both end with `Normative` disagreeing with the author.
**Corrected 2026-09-21**, on the author's instruction, in a commit that changes
nothing else.

The state before the 2026-09-13 implementation, kept because it is what the
cells were declared against: *"Holds for the light environment and for the module;
does not hold for the function or the lambda"* — a block's name read outside it
was already a static error in all three engines, a block modifying its container's
name worked, sibling blocks with the same name were independent, `°` anchored
above a loop, and two modules each holding `x` did not collide (7 / 999), nor did
a module and its importer (5 / 999). The function and the lambda both read the
file's top-level names.

**Held by** — `isolation/lambda-at-file-level-reads-the-file`,
`isolation/lambda-sees-its-enclosing-function`,
`isolation/light-block-does-not-leak`,
`isolation/light-block-modifies-its-strong`,
`isolation/light-siblings-are-independent`,
`isolation/hot-definition-anchors-above`, `isolation/lambda-write-stays-inside`,
`isolation/*underscore*`, and the cells it shares with MEM-2:
`isolation/block-var-*`, `isolation/caller-local-*`.

**The write direction** is held since 2026-10-06 by
`isolation/lambda-write-stays-inside`, which asks both rows of the table above
and checks itself: a lambda that wrote through, or a block that did not, ends in
a `##Div`. Until then `light-block-modifies-its-strong` held one half and nothing
held the other — an engine in which a lambda wrote through to its container
passed every cell this premise named. It was found, like
`write-inside-a-function-does-not-escape` on 2026-09-13, by reading the premises
against the cells rather than by a failure.

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
`isolation/variable-and-function-alike`, `isolation/function-defined-twice`,
`isolation/loop-iterator-*`. The first four were **red** when declared on
2026-09-12 — the declared debt of a decision taken, not a defect nobody noticed —
and are green since the implementation the same day.

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

**Held by** — `collections/result-used-builds`, `collections/result-discarded-modifies` — the two halves asked together, because either alone is met by an engine that always does one of them.

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

**Held by** — `collections/indexed-assignment-does-not-exist`, `collections/indexed-assignment-on-a-dictionary`.

---

### COL-3 — `[…]` is homogeneous, `#[…]` declares the mix

**Source** — `COLLECTIONS.md` § 3.

**Normative** — `[…]` is checked element by element against the first; `#[…]`
declares a mix and is not checked. **They are the same type**, and every
operator behaves the same on both — `#` is the meta mark, not a second type.

**Measured** (2026-09-12) — **holds, statically**: `[1, "dos"]` is `error: array
element 2 has type String, but expected Int`; `#[1, "dos"]` passes.

**Held by** — `collections/array-is-checked`, `collections/declared-mix-is-not`, `collections/lambdas-*`.

---

### COL-4 — The tuple is positional, immutable, and unordered

**Source** — `COLLECTIONS.md` § 4.

**Normative** — A tuple cannot be modified — the check is on the receiver, so
`t[1]$~ v` and `t$+ v` both refuse. Equality compares element by element at
every depth; **ordering does not exist**, because a positional heterogeneous
value has no defensible one.

**Measured** (2026-09-12, both engines; re-measured 2026-10-06, all three) —
holds: `cannot modify tuple 't': tuples are immutable`, and `(1,2) < (3,4)` is
`cannot compare values with operator '<': Tuple and Tuple`, a `##Type` — the
operator as the program writes it since GLB-028, the kind since GLB-091.

**Held by** — `collections/tuple-is-immutable`, `collections/tuple-has-no-ordering`, `collections/tuple-compares-equal`.

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

**Held by** — `collections/dictionary-refuses-a-position`, `collections/absent-key-is-an-error`.

---

### COL-6 — Assignment copies; there is no aliasing

**Source** — `COLLECTIONS.md` § 6.

**Normative** — Binding a collection to a second name gives that name a value,
not a view: writing through one never shows through the other. How the copy is
paid for is not part of the semantics — `Rc` and copy-on-write are invisible by
requirement.

**Measured** (2026-09-12, both engines) — holds: `f = e` then `e[1]$~ 99` leaves
`f` as `[1, 2, 3]`.

**Held by** — `collections/assignment-copies`. Its COST is held by `zyquality/cost/`, which is a different claim about the same mechanism.

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
rule, the words get `unexpected token: '{'` (re-measured 2026-10-06; it was
`LBrace`), which refuses correctly and explains nothing.

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
read as a bare break followed by an expression, and failed with `undefined
variable 'outer'`: a correct refusal, with a message that did not state the rule.
Re-measured 2026-10-06, all three engines: `'@! outer' is a bare '@!' and a name
that is read and discarded, not a labelled jump` — the message now says what
happened.

**Held by** — `refusal/modality-before-its-label`. Unlike its six siblings this
one leaves a trace a program can carry, so it was the one that could have a cell
— and now has it.

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

**Held by** — `lifetime/*` for the explicit half. The automatic half is held
elsewhere: `zyquality/cost/`, where it is a ratio of peak memory against a
control — a claim about cost, which no cell can assert.

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

**Held by** — `modularity/import-path-is-a-literal`,
`modularity/imports-come-first`,
`modularity/subscript-path-is-taken-literally`.

The `eval` half **cannot have a cell at all**, and the reason is worth keeping:
no program demonstrates the ABSENCE of a feature. A cell that tried would test
whichever spelling its author guessed, and pass for every spelling nobody
thought of. It is held by the language's own shape — there is no form in the
grammar that takes a string and runs it — and by `SYM-5`, which requires a mark
to be described before it exists. Adding `eval` would have to go through that
door first.

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

**Held by** — review, and it cannot be otherwise. The claim is that the list
is CLOSED, and closure is not a property any program exhibits: a cell can show
that `std/net` reaches the network, and no cell can show that nothing else does.
What holds it is the same thing that holds `SYM-5` — a new gate would need a new
mark or a new `std/` module, and both go through a declaration before they
exist. The day the list grows without this premise changing, the defect is in
the review and no test would have caught it.

### AGT-5 — A package is source, never a binary

**Source** — `AGENTIC.md` § 11.

**Normative** — A `.zyp` is a ZIP of `.zy` files and its manifest: readable,
diffable, auditable. Running one extracts to an ephemeral directory and **never
`chdir`s**, so code is disposable while the data a script writes lands in the
user's real working directory.

**Measured** — holds by construction; `zymbol-package` never compiles or
executes Zymbol code.

**Held by** — elsewhere: `web/tests/test_zyp.mjs`, which reads `.zyp` archives as ZIPs of
source and is where the two defects an audit found on this path are pinned. Not
a cell, and not for want of trying: a `.zyp` is an archive rather than a program,
so what has to be asked is about the FILE and not about what it prints —
the same shape as `zyquality/cost/`, which measures a ratio no cell can assert.

---

## 5b. Types

Distilled on 2026-10-06 from `CANDIDATES.md` § C-TYP-2, the first of the four
type candidates to be decided. It is numbered 5b so that §§ 6 and 7 keep the
numbers other documents cite them by. `MODEL.md` § 5.3 said, when TYP-2 was
decided, that no premise about typing could be written until C-TYP-3 was. TYP-2
does not depend on it — it is about what a call may pass, not about what a type
change on a name is — and § 5.3 says so since 2026-10-07.

### TYP-2 — What inference reaches about a parameter is refused before the program runs

**Source** — `CANDIDATES.md` § C-TYP-2, decided by the author on 2026-10-05
(`ZyDDT/HALLAZGOS/zyjs.md`, ZYJS-048): of the candidate's two questions, the
argument-type half is *refused statically*, and the Rust engines' behaviour is
the rule rather than a liberty of one implementation. The array half of the same
candidate was already a premise, COL-3, and holds statically in all three.

**Normative** — A parameter's type is what the body's use of it requires:
arithmetic makes it a number, the left side of `&&` and `||`, and the operand
of `!`, a Bool, an ordering against a literal that literal's type, an index into
a collection the body built a position or a key, and passing it to a function declared before it that
function's parameter type. A use requires something of a parameter only when
every path through the body goes through it: a use in one branch of a `?`, in
the body of a `@`, under `!?`, or in one arm of a `??` requires nothing — a
function may treat each type its own way, and the language stays variant. A
call that passes a value of another type is
**refused statically** — before anything in the program runs, so nothing the
program would have printed or written happens. What inference does not reach is
not refused statically: the program runs, and the operation fails where it
fails.

**Measured** (2026-10-06, all three engines) — **holds, statically**: with
`f(v) { <~ v + 1 }`, `>> f("a") ¶` is `error: argument 1 has type String, but
function 'f' expects Number`, and a `>> "antes" ¶` written before the call does
not print. The boundary is where the candidate put it: the type of a parameter
*of a parameter* (`aplica(g, v) { <~ g(v) }` called with `doble` and `"hola"`)
is not reached, and that program fails at run time in all three.

**Measured** (2026-10-06, all three engines, after GLB-101 and GLB-103) —
**holds**: a function that treats each type in its own branch, `f(v) { t = v#?
? t[1] == "###" { <~ v + 1 } _? t[1] == "##\"" { <~ v "1" } }`, accepts `f(1)`
and `f("a")` and answers `2` and `a1`; so does a guard that returns early, a use
in the body of a `@`, under `!?` or in one arm of a `??`; and `f(a, b) { <~ a &&
b }` accepts `f(#0, 5)`, which answers `#0`. Both branches of a `?` and its `_`,
every arm of a `??`, and a use every path goes through are still refused before
the program runs.

**Held by** — `refusal/argument-type-*`

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

- ~~Is a lambda a self-contained space, like a function?~~ **Decided: it is
  light** — it reads the scope it is written in and writes only the names it
  declares (MEM-6). The answer first written here on 2026-09-12, «strong», was
  the error of precision MEM-6's History records.
- ~~Does MEM-2 apply to reading, to writing, or to both?~~ **Both, since
  2026-09-13**: a file variable read from inside a named function is a static
  error in all three engines, and a write inside a call never escapes it — the
  reading this document took from «vistas o modificadas».

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
2. **A red cell is not a reason to weaken a premise.** MEM-2 had four red cells
   on 2026-09-12. They stayed red until the author decided, the decision was
   recorded here with its date, and they went green with the implementation on
   2026-09-13.
3. **A description that disagrees with a premise is a defect in the
   description.** `LLM.md` rule 5 disagreed with MEM-2 from 2026-09-12 until it
   was corrected on 2026-10-07, and that was a bug in `LLM.md`, not evidence
   about the rule.
