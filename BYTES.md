# Bytes

How a Zymbol program holds bytes and changes them — what other languages call
*manipulating memory*. A byte is a character whose code fits between `0x00` and
`0xFF`. So this document builds on [`CHARACTERS.md`](CHARACTERS.md), which says
what a character is, how it is written in four bases, how it becomes a number
or a text, and how a text becomes bytes in a file. Read its § 1 first: a
character, its code and its bytes are three different things.

It gathers decisions taken one at a time: a byte is a character (D-22); the
base conversions write the width of a byte (D-23); the casts between a
character and its code were removed (D-24) and the bars became three doors
(D-25); `Char $* n` builds a buffer (D-27); writing a byte in place is linear
(D-26); and `std/bits` holds the bits and the words (D-28).

**Status.** §1–§5 describe the language as it is on 2026-11-25. Every example
was run in all four engines (zytw, zyvm, zyjs, zyjsvm) and they agree. § 6 gives
the measured cost. § 7 lists what was open and how each was decided.

---

## 1. A byte is a Char from 0x00 to 0xFF

Zymbol has no byte type. A byte is the character with that number, written as
any character is written by its code ([`CHARACTERS.md`](CHARACTERS.md) § 2):

| Base | Lowest | Highest | the byte 65 |
|---|---|---|---|
| hexadecimal | `0x00` | `0xFF` | `0x41` |
| decimal | `0d0` | `0d255` | `0d65` |
| octal | `0o000` | `0o377` | `0o101` |
| binary | `0b00000000` | `0b11111111` | `0b01000001` |

The byte `0x41` is the character `A` and the byte `0xFF` is the character `ÿ`.
Written back as text, a byte never takes fewer digits than one byte needs in
that base, so 256 of them line up in a column: `0x|'A'|` is `"0x41"`,
`0o|'A'|` is `"0o101"`, `0b|'A'|` is `"0b01000001"` (D-23).

A **buffer** is therefore a `String` whose characters are all from `0x00` to
`0xFF`. It is the form ZyBF's tape already took (GLB-109). `io::read_bytes(p)`
reads a file into one, a Char per byte, and `io::write_bytes(p, b)` writes one
back (D-22).

**What is not a byte.** A character whose code is past `0xFF` — `こ` is
`0x3053`, `🌕` is `0x1F315` — is a character and not a byte, and writing it as
one is a `##Range` error:

```zymbol
<# std/io => io
io::write_bytes("x.bin", "a🌕")
>> "no llega" ¶
```

stops with `io::write_bytes: '🌕' is U+1F315, out of range for a byte: a byte
is a Char from 0x00 to 0xFF`, and the file is not touched. The **bytes of** `🌕`
are four, `F0 9F 8C 95`, the ones UTF-8 stores it in; to get them, write the
text as text and read it back as bytes ([`CHARACTERS.md`](CHARACTERS.md) § 5).

**A byte is a number through the doors.** `#|b[i]|` is the number of the byte
at `i`, `|n|` is the byte of the number `n`, and `0x|b[i]|` writes it in
hexadecimal ([`CHARACTERS.md`](CHARACTERS.md) § 3):

```zymbol
b = "" |72| |105| 0xFF
>> #|b[1]| " " #|b[2]| " " 0x|b[3]| " " b$# ¶   // → 72 105 0xFF 3
```

## 2. There are no addresses

Zymbol has values. It has no memory cells: no pointer, no address, no
`peek`/`poke`, and no two names that reach the same storage. This follows from
two premises, so it is a rule and not a missing feature:

- **COL-6**: assignment copies, and there is no aliasing.
- **MEM-8**: memory is released automatically and invisibly, or destroyed by hand.

So a program that "manipulates memory" in Zymbol holds a **byte buffer** as a
value and changes it with the operations it already uses on text. It is
Python's `bytearray` without the sharing. The author kept both premises when
the question came up (§ 7, B-6).

## 3. A buffer

Every operation on a buffer is an operation on text.

| What | Write | Note |
|---|---|---|
| create | `0x00 $* n`, or `"" 0x00 0x00 0x00 0x00` | a Char repeats as the text of one character (D-27) |
| length | `b$#` | in bytes, because every character of a buffer is one |
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

## 4. Wider than a byte: words, with `std/bits`

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

A word is not a character either: `bits::to_le(4660, 2)` is two bytes, `0x34`
and `0x12`, and has nothing to do with the character whose code is 4660.

## 5. Bits, with `std/bits`

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

## 6. Cost, measured 2026-11-24 and 2026-11-25

Reading a byte is linear over the whole buffer: the engines keep where the
characters they read start, so each `b[i]` does not count from the beginning.

**Writing one byte in place is linear in all four engines** since 2026-11-25
(B-2, ZYJS-076), when nothing else holds the text and the new byte takes the
room the old one did. That room is the engine's hidden storage, not anything a
program sees ([`CHARACTERS.md`](CHARACTERS.md) § 5): in the Rust engines' UTF-8,
a byte from `0x00` to `0x7F` takes one byte of storage and one from `0x80` to
`0xFF` takes two, so `0x41` for `0x42` and `0x80` for `0xFF` are written where
they are. The Rust engines write the bytes of the String; zyjs writes the array
of characters its variable owns; the browser VM, whose registers hold
JavaScript strings that cannot be written into, keeps the text as a tree of
pieces whose root is the text, and rebuilds one piece and the joins above it.
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
710 ms, zyvm 600 ms (2.59 GB allocated), zyjs 4.8 s and zyjsvm 5.8 s.

A write that changes the room — `0x41` for `0xFF` in the Rust engines — still
copies the text: what follows it moves. `zyquality/cost/` holds the linear case
(`growth/byte-write-in-place`) in all four engines.

## 7. What was open, and how it was decided

- ~~**B-1 — reading an Int from text in a base.**~~ **Closed by D-25** on
  2026-11-24: `#|…|` reads the four prefixes, so `#|"0x1F0000"|` is 2031616 and
  `#|"0b0101"|` is 5, and what `0x|n|` writes reads back as the Int.
- ~~**B-2 — writing a byte in place is quadratic.**~~ **Decided 2026-11-25,
  option (a):** the Rust engines write a byte where the text is, when nothing
  else holds it and it takes the room the old one did (§ 6). ~~The browser keeps
  the copy; that is ZYJS-076, a declared debt in the cost gate.~~ The browser
  engines were fixed the same day, at the author's request (ZYJS-076, and
  ZYJS-077 for appending). What remains: a write that changes the room copies.
- ~~**B-3 — `Char $* n`.**~~ **Decided 2026-11-25 (D-27):** a Char repeats as
  the text of one character, in every engine; zyjs was the one that refused it.
  `0x00 $* n` is the way to create a buffer (§ 3).
- ~~**B-4 — bit operations.**~~ **Decided 2026-11-25 (D-28):** the module
  `std/bits` (§ 5), by the rubric: a named operation with arguments, and no
  existing mark derives it.
- ~~**B-5 — words of several bytes.**~~ **Decided 2026-11-25 (D-28):** in the
  same module, `to_le`/`to_be`/`from_le`/`from_be` and `signed` (§ 4), the byte
  order in the name rather than in a text option.
- ~~**B-6 — real memory.**~~ **Decided 2026-11-25: no.** Addresses, a shared
  buffer, or memory a program frees by hand would contradict COL-6 and MEM-8
  (§ 2), and the author keeps both. A buffer is a value.

What is still open about characters — the escapes, and a text's UTF-8 bytes
without a file — is in [`CHARACTERS.md`](CHARACTERS.md) § 6.
