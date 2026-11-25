# Characters

What a character is in Zymbol, how it is written, how it becomes a number or a
text, and where bytes begin. [`BYTES.md`](BYTES.md) builds on this one: a byte
is a character whose code fits between `0x00` and `0xFF`.

**Status.** §1–§5 describe the language as it is on 2026-11-25. Every example
was run in all four engines (zytw, zyvm, zyjs, zyjsvm) and they agree. §6 lists
what is open, and nothing in it is decided.

---

## 1. Three things that are easy to confuse

| | What it is | `A` | `こ` | `🌕` |
|---|---|---|---|---|
| **the character** | what is seen; a `Char` holds one | `A` | `こ` | `🌕` |
| **its code** | one number per character, Unicode's | 65 | 12371 | 127765 |
| **its bytes** | how a file stores it, in UTF-8 | `41` | `E3 81 93` | `F0 9F 8C 95` |

- A **character** is one Unicode code point, and a `Char` holds exactly one.
- Its **code** is a number from 0 to `0x10FFFF` (1 114 111). The 2 048 codes
  from `0xD800` to `0xDFFF` (surrogates) name no character. The code is what the
  four bases write — §2.
- Its **bytes** are how the character is stored. A file holds a text as UTF-8,
  where a character takes 1 to 4 bytes, and those bytes are other numbers: none
  of the four bytes of `🌕` is 127765. A program sees them only when it asks for
  them — §5.

The confusion is natural, because Zymbol writes a byte as the character with
the same number ([`BYTES.md`](BYTES.md) § 1). In a buffer read with
`io::read_bytes`, the byte `0x41` is the Char `0x41`, `A`, and the byte `0xFF`
is the Char `0xFF`, `ÿ`. For `A` this also matches its UTF-8, a single byte
`41`, and so it goes for every code from `0x00` to `0x7F`. From `0x80` on it
does not: the text `"ÿ"` written to a file is two bytes, `C3 BF`, and read back
with `io::read_bytes` it is two Chars, `0xC3` and `0xBF`. A literal such as
`0x1F315` writes a **code**; whether that code is also a byte depends only on
whether it is `0xFF` or less.

## 2. Writing a character

**Between quotes**, exactly one character: `'A'`, `'こ'`, `'🌕'`. `'ab'` is an
error before the program runs. A Char takes these escapes: `\n` `\t` `\r` `\'`
`\\` and `\0` (the character of code 0). Any other `\` is an error.

**By its code, in any of four bases.** The four spellings are one number, so
they are one character:

```zymbol
>> 0x1F315 " " 0d127765 " " 0o371425 " " 0b11111001100010101 ¶   // → 🌕 🌕 🌕 🌕
>> (0x41 == 'A') " " (0d65 == 'A') " " (0o101 == 0b01000001) ¶   // → #1 #1 #1
```

| Prefix | Base | Digits | `A` | `こ` | `🌕` |
|---|---|---|---|---|---|
| `0x` | hexadecimal | `0`–`9`, `a`–`f`, `A`–`F` | `0x41` | `0x3053` | `0x1F315` |
| `0d` | decimal | `0`–`9` | `0d65` | `0d12371` | `0d127765` |
| `0o` | octal | `0`–`7` | `0o101` | `0o30123` | `0o371425` |
| `0b` | binary | `0`, `1` | `0b1000001` | `0b11000001010011` | `0b11111001100010101` |

- Leading zeros are allowed and change nothing: `0d065`, `0o0101`.
- The prefix is lower case. `0X41` is the number `0` followed by a name `X41`.
- The digits are ASCII. `0d६५` is an error, even though `६५` is a number
  elsewhere in the language.
- A code past `0x10FFFF`, or a surrogate, is refused before the program runs:
  `invalid Unicode code point: 0x110000`.

**Without a prefix it is a number, not a character.** `3061` is the Int three
thousand sixty-one. `ち` is `0x3061`, which in decimal is `0d12385`. And
`0306B` is the number `0306` followed by a name `B`.

**In a text**, the characters are written as they are, `"こんにちは"`, or a
text is built by joining them: `"" 0x3053 0x3093 0x306B 0x3061 0x306F` is the
same text. A String takes these escapes: `\n` `\t` `\r` `\"` `\\` `\{` `\}`.
`{name}` puts a value inside: `"{c}"` is the text of the character `c` (§6 lists
where the two sets of escapes disagree).

## 3. A character is a number: the three doors

A character is not an Int, even though its code is one. `0x41 == 65` is `#0`,
and `'A' + 1` is a `##Type` error. Three doors cross between a character, its
number and its text in a base (D-25, v0.0.12). Each takes a Char, an Int, or a
text written in any base:

| Door | Gives | `'A'` | `65` | `"0x41"` |
|---|---|---|---|---|
| `0x\|v\|` `0d\|v\|` `0o\|v\|` `0b\|v\|` | the text in that base | `"0x41"` `"0d65"` … | the same | the same |
| `\|v\|` | the character | `'A'` | `'A'` | `'A'` |
| `#\|v\|` | the number | `65` | `65` | `65` |

```zymbol
luna = 0x1F315
>> luna " " #|luna| " " 0x|luna| " " 0b|luna| ¶   // → 🌕 127765 0x1F315 0b11111001100010101
>> |127765| " " |"0x1F315"| " " |"0d127765"| ¶     // → 🌕 🌕 🌕
>> |#|'a'| + 1| ¶                                   // → b
```

To compute with a character, take its number with `#|c|` and come back with
`|n|`. The text in a base never has fewer digits than a byte takes: `0x|'A'|` is
`"0x41"` and `0b|'A'|` is `"0b01000001"` (D-23). A text with no prefix is read in
the base of the door, and in decimal for `|…|` and `#|…|`.

**Characters compare by their code.** `'Z' < 'a'` is `#1`, because 90 < 97, and
`'é' > 'z'` is `#1`.

**A Char and a text of one character are different values.** `s[2]` is a
`Char`; `s$[2..2]` is a `String`. `#|…|` reads a Char as its code and a text as
the number its digits write:

```zymbol
s = "a7€🌕"
>> s$# " " s[2] " " (s[2]#?)[1] " " (s$[2..2]#?)[1] ¶   // → 4 7 ##' ##"
>> #|s[2]| " " #|s$[2..2]| ¶                           // → 55 7
c = s[2]
>> #|"{c}"| ¶                                           // → 7
```

The value of a digit is its text's reading, `#|"{c}"|`, and that works in any of
the 69 scripts the language writes numbers in.

## 4. Code point, grapheme, column

One character is one code point, and that is what every position counts: `$#`
counts code points, and `s[i]` is the `i`-th one. What a reader takes for one
letter may be several, and what a screen draws may take two columns:

```zymbol
<# std/text => tx
<# std/term => t
e = "e" |0x301|
>> e " " e$# " " tx::graphemes(e)$# ¶              // → é 2 1
>> "こんにちは"$# " " t::width("こんにちは") ¶      // → 5 10
```

| To count | Use |
|---|---|
| code points, the characters of Zymbol | `s$#` |
| letters as a reader sees them (grapheme clusters) | `tx::graphemes(s)$#` — `std/text` |
| columns on a terminal | `t::width(s)` — `std/term` |

A name — a variable, a function — is normalized to NFC, so `é` written in one
code point or in two is the same name (D-20). A text keeps the code points it
was written with.

## 5. From text to bytes

A file stores a text in UTF-8. `io::write` turns the characters into bytes and
`io::read` turns them back, and a file that is not valid UTF-8 is an `##IO` soft
error for `io::read`. To see the bytes, `io::read_bytes` gives one Char per byte,
each from `0x00` to `0xFF` (D-22):

```zymbol
<# std/io => io
io::write("luna.txt", "🌕")
b = io::read_bytes("luna.txt")
>> b$# ":" ¶                                       // → 4:
@ c:b { >> " " 0x|c| }                             // →  0xF0 0x9F 0x8C 0x95
>> ¶
>> (io::read("luna.txt") == "🌕") ¶                 // → #1
io::delete("luna.txt")
```

| Text | Characters (`$#`) | Bytes in UTF-8 |
|---|---:|---|
| `A` | 1 | `41` |
| `ÿ` | 1 | `C3 BF` |
| `こんにちは` | 5 | `E3 81 93` `E3 82 93` `E3 81 AB` `E3 81 A1` `E3 81 AF` — 15 |
| `🌕` | 1 | `F0 9F 8C 95` — 4 |

**A character past `0xFF` is no byte.** `io::write_bytes(p, "🌕")` is a
`##Range` error — `'🌕' is U+1F315, out of range for a byte` — and so is
`bits::from_le("🌕")`. Bytes are characters from `0x00` to `0xFF`, and the rest
of their story is [`BYTES.md`](BYTES.md).

**How an engine stores a text is invisible.** The Rust engines keep it in
UTF-8, where `ÿ` takes two bytes. The browser keeps it in JavaScript's UTF-16,
where `🌕` takes two 16-bit units. A program never sees either: `"ÿ"$#` and
`"🌕"$#` are both 1, in every engine. When `BYTES.md` § 6 says a write "takes
the room the old one did", it is that hidden storage it talks about.

## 6. Open — none of this is decided

- **C-1 — the escapes of a Char and of a String disagree.**
  - The problem:
    - `'\0'` is the character of code 0, but `"\0"` is the text `"0"` (code 48).
    - An unknown escape is an error in a Char (`'\q'`), and in a String it is
      kept without a word: `"\q"` is `"q"`.
    - Neither takes a code: `\u{41}` is an error in both. VS Code's grammar
      colours `\u{…}` as an escape all the same.
  - Options: (a) one set for both, with unknown escapes an error everywhere;
    (b) leave them as they are, and document the difference; (c) add `\u{…}` to
    both. Each one changes what some program means, so the corpus decides how
    far.
- **C-2 — a text's UTF-8 bytes need a file.** There is no `encode` or `decode`:
  the only way from `"🌕"` to `F0 9F 8C 95` in memory is to write the text to a
  file and read its bytes back.
  - Options: (a) two functions, for example `tx::utf8(s)` → bytes and
    `tx::from_utf8(b)` → text; (b) leave it to the file round trip.
