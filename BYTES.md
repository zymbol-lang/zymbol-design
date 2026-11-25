# Bytes

How a Zymbol program holds bytes and changes them — what other languages call
*manipulating memory*. It is built from three earlier decisions, so there is little
new here. A byte is a character (D-22). Base literals were already characters.
The base conversions were given the width of a byte (D-23), the casts between a
character and its code were removed (D-24), and the bars became three doors, one
per result (D-25). This document puts
them in one place, with how each open question was decided.

**Status.** §1–§6 describe the language as it is on 2026-11-25. Every example
was run in all four engines (zytw, zyvm, zyjs, zyjsvm) and they agree. §7 gives
the measured cost. §8 lists what was open and how each was decided.

---

## 1. There are no addresses

Zymbol has values. It has no memory cells: no pointer, no address, no
`peek`/`poke`, and no two names that reach the same storage. This follows from
two premises, so it is a rule and not a missing feature:

- **COL-6**: assignment copies, and there is no aliasing.
- **MEM-8**: memory is released automatically and invisibly, or destroyed by hand.

So a program that "manipulates memory" in Zymbol holds a **byte buffer** as a
value and changes it with the operations it already uses on text. It is Python's
`bytearray` without the sharing. Changing that would mean changing a premise,
and only the author does that (§8, B-6).

## 2. A byte is a Char from 0x00 to 0xFF

A base literal is a `Char`, not an `Int`. `0x41` is `'A'`. Each base has its own
spelling of the same byte:

| Base | Lowest | Highest | `'A'` |
|---|---|---|---|
| hexadecimal | `0x00` | `0xFF` | `0x41` |
| decimal | `0d0` (or `0d00`, `0d000`) | `0d255` | `0d65` |
| octal | `0o000` | `0o377` | `0o101` |
| binary | `0b00000000` | `0b11111111` | `0b01000001` |

Leading zeros are allowed in every base, and they do not change the value.
Converting back to text uses the widths in the table (D-23). Hexadecimal, octal
and binary never use fewer digits than one byte takes, so the 256 bytes line up
in a column. The decimal is written the way it is read: `0d65`.

A buffer is therefore a `String` whose characters are all between `0x00` and
`0xFF`. It is the form ZyBF's tape already took (GLB-109). `io::read_bytes` and
`io::write_bytes` read such a String from a file and write it back (D-22).

## 3. Three doors, one per result

A number has no base: a base is a way of writing it. So the four prefixes give a
**text** — the value written in that base — and the two other results have doors
of their own (D-25, v0.0.12): the bare bars give the **character**, and `#|…|`
gives the **number**. Each door takes a Char, an Int, or a text written in any
base, with a sign and any of the four prefixes.

```text
                      ┌─ 0x|v| 0d|v| 0o|v| 0b|v| ─▶  text   "0x41" "0d65" "0o101" "0b01000001"
   Char 'A'           │
   Int 65        ─────┼─ |v| ──────────────────────▶  Char   'A'
   text "0x41"        │
                      └─ #|v| ─────────────────────▶  Int    65
```

```zymbol
caracter = 'A'
numero = 65
elemento = 0d|caracter|
>> elemento " " 0d|"0b01000001"| " " 0d|0o101| " " 0x|numero| ¶   // → 0d65 0d65 0d65 0x41
>> |elemento| " " |numero| " " |"0d{numero}"| " " |0d65| ¶         // → A A A A
>> #|caracter| " " #|elemento| " " #|"0x41"| ¶                     // → 65 65 65
```

| From | To | Write |
|---|---|---|
| Char, Int or text in a base | text in a base | `0x\|v\|` `0d\|v\|` `0o\|v\|` `0b\|v\|` — a text with no prefix is read in that base |
| Int or text in a base | Char | `\|v\|` — a `##Range` error past `0x10FFFF`, on a surrogate, or below 0; a Float is `##Type` |
| Char or text in a base | Int | `#\|v\|` — a text that does not read is handed back unchanged |
| digits in a source file | Char | the literal: `0x41` |

**A character is its code.** `#|'7'|` is 55, the code of the character `7`. The
value of a digit is the text's reading, `#|"{c}"|`, in any of the 69 scripts.

**Removed in v0.0.12** (D-24): `##'n`, the cast from a code to its character
(GLB-109, v0.0.10), and `##!` on a `Char` (v0.0.8). Two operators did what the
doors do, they looked nothing like the base forms beside them — `##!letra_A`
gave `65` where `0x|letra_A|` gave `"0x41"` — and the applications that used
them left findings behind (ERROR-BF-002, ZyEdit's digit sums, ZyBank's ASCII
table). Both are refused before a program runs, with a help line that names
`|n|` and `#|c|`. `##!` keeps truncating a Float. For a day of v0.0.12, `0d|…|`
was the door to the number (D-24), which made the decimal the one base whose
result was not text; D-25 undid it.

## 4. A buffer

Every operation on a buffer is an operation on text.

| What | Write | Note |
|---|---|---|
| create | `0x00 $* n`, or `"" 0x00 0x00 0x00 0x00` | a Char repeats as the text of one character (D-27) |
| length | `b$#` | |
| read a byte | `b[i]` (a Char), `#\|b[i]\|` (its number) | 1-based |
| change a byte | `b[i]$~ 0xFF` | the result is discarded, so the edit lands in `b` (COL-1) |
| a changed copy | `c = b[i]$~ 0x7F` | the result is used, so `b` keeps its value |
| a slice | `b$[2..3]`, `b$[2:2]` | |
| search | `b$? 0xFF`, `b$?? 0x00` | contains, every position |
| join | `a b`, `a$++ b` | |
| file | `io::read_bytes(p)`, `io::write_bytes(p, b)` | a Char past `0xFF` is a `##Range` error and leaves the file as it was |

```zymbol
b = 0x00 $* 4
b[2]$~ 0xFF
c = b[3]$~ 0x7F
>> 0x|b[2]| " " 0x|b[3]| " " 0x|c[3]| ¶                 // → 0xFF 0x00 0x7F
>> b$[2..3]$# " " (b$? 0xFF) " " (b$?? 0x00) ¶          // → 2 #1 [1, 3, 4]
```

`$~` does not check the range: `b[1]$~ 0x100` is accepted, and the String then
holds a character that is not a byte. The check is made when the buffer is
written to a file, by `io::write_bytes`.

## 5. Wider than a byte: `std/bits`

A word of 1 to 6 bytes is written and read in either byte order by `std/bits`
(D-28). `to_le(n, k)` and `to_be(n, k)` give a String of `k` bytes, the form a
buffer takes, so a word goes into a buffer by joining or slicing it; `from_le(s)`
and `from_be(s)` read one back; `signed(n, w)` reads a `w`-bit word in two's
complement.

```zymbol
<# std/bits => bits
w = bits::to_le(4660, 2)
>> 0x|w[1]| " " 0x|w[2]| " " bits::from_le(w) ¶            // → 0x34 0x12 4660
cabecera = "ZY" bits::to_be(1, 2) bits::to_le(65536, 4)
>> cabecera$# " " bits::from_le(cabecera$[5..8]) ¶          // → 8 65536
>> bits::signed(#|0xFF|, 8) " " bits::signed(#|0x7F|, 8) ¶  // → -1 127
```

The integer is ±(2^53 − 1), and an overflow is an error, not a wrap. So an
unsigned word of up to six bytes fits exactly, and a 64-bit word does not: six
is the widest word `std/bits` writes or reads, and a wider one is a `##Range`
error. Until D-28 a program wrote `put16` and `get16` itself, with `/` and `%`.

## 6. Bits: `std/bits`

There is no bit operator: a single `&` is not a character of the language (only
`&&` is), and `|` closes the bars. Bits are functions of `std/bits` on a
non-negative integer up to 2^53 − 1, a Char taken as its code (D-28):

```zymbol
<# std/bits => bits
>> bits::and(0xA5, 0x0F) " " bits::or(0xA0, 0x05) " " bits::xor(0xFF, 0x0F) ¶   // → 5 165 240
>> bits::shl(1, 4) " " bits::shr(0xF0, 4) " " bits::not(0x0F, 8) ¶              // → 16 15 240
>> bits::and(bits::shr(0xA5, 7), 1) ¶                                           // → 1
```

`not` takes a width, because an integer of Zymbol has none: `not(0x0F, 8)` is
`0xF0`, and `not(0x0F, 16)` is `0xFFF0`. A `shl` past the range is an overflow,
as `*` past it is.

## 7. Cost, measured 2026-11-24 and 2026-11-25

Reading a byte is linear over the whole buffer: the engines keep the position of
the characters they read, so each `b[i]` does not count from the start.

**Writing one byte in place is linear in all four engines** since 2026-11-25
(B-2, ZYJS-076): when nothing else holds the text and the new character takes
the room the old one did — ASCII for ASCII, a byte from `0x80` to `0xFF` for
another — it is written where the text is, and the engine keeps the positions it
knew. The Rust engines write the bytes of the String; zyjs writes the array of
characters its variable owns; the browser VM, whose registers hold JavaScript
strings that cannot be written into, keeps the text as a tree of pieces whose
root is the text, and rebuilds one piece and the joins above it.
The program below writes `n` bytes, each from `0x80` to `0xFF`:

```zymbol
b = ""
@ _i:1..n { b = b 0x80 }
@ i:1..n { b[i]$~ 0xFF }
```

| n | zytw | zyvm | zyjs | zyjsvm |
|---:|---:|---:|---:|---:|
| 16 000 | 25 ms | 46 ms | 235 ms | 204 ms |
| 256 000 | — | — | 1 311 ms | 363 ms |

The browser figures include reading every byte back, and Node's start. Before
2026-11-25 the four engines copied the text for each byte: at 16 000, zytw
710 ms, zyvm 600 ms (2.59 GB allocated), zyjs 4.8 s and zyjsvm 5.8 s. A write that changes the width — `0x41`
for `0xFF` — still copies the text in every engine: a byte moves, and every
character after it with it. `zyquality/cost/` holds the linear case
(`growth/byte-write-in-place`), with the browser's debt declared.

## 8. What was open, and how it was decided

- ~~**B-1 — reading an Int from text in a base.**~~ **Closed by D-25** on
  2026-11-24: `#|…|` reads the four prefixes, so `#|"0x1F0000"|` is 2031616 and
  `#|"0b0101"|` is 5, and what `0x|n|` writes reads back as the Int.
- ~~**B-2 — writing a byte in place is quadratic.**~~ **Decided 2026-11-25,
  option (a):** the Rust engines write a character of the same width where the
  text is, when nothing else holds it (§7). ~~The browser keeps the copy; that is
  ZYJS-076, a declared debt in the cost gate.~~ The browser engines were fixed
  the same day, at the author's request (ZYJS-076, and ZYJS-077 for appending).
  What remains: a write that changes the width copies in every engine.
- ~~**B-3 — `Char $* n`.**~~ **Decided 2026-11-25 (D-27):** a Char repeats as
  the text of one character, in every engine; zyjs was the one that refused it.
  `0x00 $* n` is the way to create a buffer (§4).
- ~~**B-4 — bit operations.**~~ **Decided 2026-11-25 (D-28):** the module
  `std/bits` (§6), by the rubric: a named operation with arguments, and no
  existing mark derives it.
- ~~**B-5 — words of several bytes.**~~ **Decided 2026-11-25 (D-28):** in the
  same module, `to_le`/`to_be`/`from_le`/`from_be` and `signed` (§5), the byte
  order in the name rather than in a text option.
- ~~**B-6 — real memory.**~~ **Decided 2026-11-25: no.** Addresses, a shared
  buffer, or memory a program frees by hand would contradict COL-6 and MEM-8
  (§1), and the author keeps both. A buffer is a value.
