# Bytes

How a Zymbol program holds bytes and changes them — what other languages call
*manipulating memory*. It is built from three earlier decisions, so there is little
new here. A byte is a character (D-22). Base literals were already characters.
The base conversions were given the width of a byte (D-23), the casts between a
character and its code were removed (D-24), and the bars became three doors, one
per result (D-25). This document puts
them in one place and lists what is still missing.

**Status.** §1–§6 describe the language as it is on 2026-11-24. Every example
was run in all four engines (zytw, zyvm, zyjs, zyjsvm) and they agree. §7 gives
the measured cost. §8 is open: **nothing in §8 is decided**, and each entry there
names its options.

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
| create | `"" 0x00 0x00 0x00 0x00`, or `0x00 $* n` | for `0x00 $* n` the engines disagree (§8, B-3) |
| length | `b$#` | |
| read a byte | `b[i]` (a Char), `#\|b[i]\|` (its number) | 1-based |
| change a byte | `b[i]$~ 0xFF` | the result is discarded, so the edit lands in `b` (COL-1) |
| a changed copy | `c = b[i]$~ 0x7F` | the result is used, so `b` keeps its value |
| a slice | `b$[2..3]`, `b$[2:2]` | |
| search | `b$? 0xFF`, `b$?? 0x00` | contains, every position |
| join | `a b`, `a$++ b` | |
| file | `io::read_bytes(p)`, `io::write_bytes(p, b)` | a Char past `0xFF` is a `##Range` error and leaves the file as it was |

```zymbol
b = "" 0x00 0x00 0x00 0x00
b[2]$~ 0xFF
c = b[3]$~ 0x7F
>> 0x|b[2]| " " 0x|b[3]| " " 0x|c[3]| ¶                 // → 0xFF 0x00 0x7F
>> b$[2..3]$# " " (b$? 0xFF) " " (b$?? 0x00) ¶          // → 2 #1 [1, 3, 4]
```

`$~` does not check the range: `b[1]$~ 0x100` is accepted, and the String then
holds a character that is not a byte. The check is made when the buffer is
written to a file, by `io::write_bytes`.

## 5. Wider than a byte: arithmetic

Two Ints divided with `/` give an Int, truncated toward zero, and `%` gives the
remainder. That is enough to pack and unpack a word in either byte order:

```zymbol
put16(n) { <~ "" |n % 256| |n / 256 % 256| }           // little-endian
get16(s, i) { <~ #|s[i]| + #|s[i + 1]| * 256 }
signed8(v) {
    ? v >= 128 { <~ v - 256 }
    <~ v
}
w = put16(4660)
>> 0x|w[1]| " " 0x|w[2]| " " get16(w, 1) ¶              // → 0x34 0x12 4660
>> signed8(#|0xFF|) " " signed8(#|0x7F|) ¶              // → -1 127
```

The integer is ±(2^53 − 1), and an overflow is an error, not a wrap. So an
unsigned word of up to six bytes fits exactly, and a 64-bit word does not fit.
`-7 / 2` is `-3` and `-7 % 2` is `-1`. Keep byte arithmetic non-negative.

## 6. Bits: arithmetic too

There is no bit operator. A single `&` is not a character of the language (only `&&` is), and `|` closes
the bars of `#|…|` and `0x|…|`. A bit, a shift left and a shift right are all
arithmetic:

```zymbol
bit(n, k) { <~ n / (2 ^ k) % 2 }
>> bit(#|0xA5|, 0) " " bit(#|0xA5|, 1) " " bit(#|0xA5|, 7) ¶   // → 1 0 1
>> 1 * (2 ^ 4) " " #|0xF0| / (2 ^ 4) ¶                            // → 16 15
```

`and`, `or` and `xor` need a loop over the eight bits. The precedent is that no
application has needed more. Serpiente's generator and ZyBF's tape are built
from `%` alone, and no LDV application has asked for bit operators (searched
2026-11-24).

## 7. Cost, measured 2026-11-24

Reading a byte is linear over the whole buffer: the engines keep the position of
the characters they read, so each `b[i]` does not count from the start.
**Writing one byte is quadratic.** A byte from `0x80` to `0xFF` takes two bytes
of UTF-8, so a String is not a byte array on the inside, and `b[i]$~ v` rebuilds
the whole text. The program below writes `n` bytes, each from `0x80` to `0xFF`:

```zymbol
z = 0x80
b = z $* n
@ i:1..n { b[i]$~ 0xFF }
```

| n | zytw | zyvm | zyjsvm | bytes allocated (Rust) |
|---:|---:|---:|---:|---:|
| 4 000 | 68 ms | 66 ms | 425 ms | 162 MB |
| 8 000 | 213 ms | 219 ms | 1 208 ms | 648 MB |
| 16 000 | 699 ms | 785 ms | 5 762 ms | 2.59 GB |

Doubling `n` quadruples the allocation. At this rate a 1 MB buffer cannot be
written byte by byte. zyjs does not appear in the table: it refuses `z $* n`
(B-3). This is a cost of the engines, not a rule of the language. It is §8, B-2.

## 8. Open — none of this is decided

- ~~**B-1 — reading an Int from text in a base.**~~ **Closed by D-25** on
  2026-11-24: `#|…|` reads the four prefixes, so `#|"0x1F0000"|` is 2031616 and
  `#|"0b0101"|` is 5, and what `0x|n|` writes reads back as the Int.
- **B-2 — writing a byte in place is quadratic** (§7). This is work inside the
  engines, not something a program can see.
  - Options:
    - (a) Edit in place when the old and new characters have the same width in
      UTF-8, which covers every write within `0x00`–`0x7F` or within
      `0x80`–`0xFF`.
    - (b) A String whose characters are all ≤ `0xFF` keeps one byte per
      character inside the engine.
    - (c) Record it as a finding with a `cost` case, and leave it for later.
- **B-3 — `Char $* n`.**
  - The problem: three engines accept `0x00 $* 4`. zyjs refuses it with
    `$* requires a string, got Char`. The GUIDE only describes repeating a
    String, so no corpus file holds this case and the gate cannot see the
    difference.
  - Options:
    - (a) A Char is accepted as the text of one character. That gives the idiom
      for creating a buffer.
    - (b) It is a `##Type` error in all four engines.
- **B-4 — bit operations.**
  - Options:
    - (a) Leave them to arithmetic (§6).
    - (b) A module, `std/bits`, with `and`, `or`, `xor`, `not`, `shl` and `shr`
      on non-negative Ints. This follows the rubric: a named operation with
      arguments, and no existing mark derives it.
    - (c) Symbols. That needs a base mark with a documented abstract meaning
      (SYM-5), and `&` and `|` are not available.
- **B-5 — words of several bytes.** The options are §5's arithmetic, or helper
  functions such as `u16le`/`u16be`/`u32le` in the same module as B-4.
- **B-6 — real memory.** Addresses, a shared buffer, or memory a program frees
  by hand would contradict COL-6 and MEM-8 (§1). This entry is recorded so the
  question has an answer here. Only the author can open it, in a commit that
  changes a premise and nothing else.
