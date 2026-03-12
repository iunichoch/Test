# LAB5 Binary Bomb Report
**Bomb Number:** 50
**Username:** nichoch

---

## Phase 1

I used `s sym.phase_1` then `pdf` to disassemble the function.
The assembly loaded a string into `%rsi` using `leaq` and then
called `strings_not_equal` to compare it against my input.
The string was visible in the r2 annotation on that line:
"I am for medical liability at the federal level."
If the strings did not match, a `jne` instruction would jump
to `explode_bomb`. So the answer is simply that exact string.

**Answer:** `I am for medical liability at the federal level.`

---

## Phase 2

I used `s sym.phase_2` then `pdf` to disassemble the function.
The assembly called `read_six_numbers` meaning the input must
be exactly 6 integers. The first two were checked directly:
index 0 must be `0` and index 1 must be `1`. A loop then
checked each subsequent number using the formula:
```
next = 2 * (current + previous)
```

Working it out from the first two values:
- n1 = 0
- n2 = 1
- n3 = 2*(0+1) = 2
- n4 = 2*(1+2) = 6
- n5 = 2*(2+6) = 16
- n6 = 2*(6+16) = 44

If any number failed the check, the code jumped to `explode_bomb`.

**Answer:** `0 1 2 6 16 44`

---

## Phase 3

I used `s sym.phase_3` then `pdf` to disassemble the function.
The assembly first called `string_length` and compared the
result to `0xa` (10), meaning the input must be exactly 10
characters long. If the length was wrong, it jumped to
`explode_bomb` immediately.

The code then loaded an array of XOR values onto the stack:
`[9, 5, 10, 1, 6, 2, 3, 9, 6, 2]`

A loop XORed each character of my input with the corresponding
array value and compared the result against a lookup table at
`obj.t.0`. I ran `px 10 @ obj.t.0` to read the table which
returned the bytes for: `E. Hubbard`

To find the correct input, I reversed the XOR for each character
using: `input[i] = table[i] XOR array[i]`

- 'E' (0x45) XOR 9  = 0x4C = 'L'
- '.' (0x2E) XOR 5  = 0x2B = '+'
- ' ' (0x20) XOR 10 = 0x2A = '*'
- 'H' (0x48) XOR 1  = 0x49 = 'I'
- 'u' (0x75) XOR 6  = 0x73 = 's'
- 'b' (0x62) XOR 2  = 0x60 = '`'
- 'b' (0x62) XOR 3  = 0x61 = 'a'
- 'a' (0x61) XOR 9  = 0x68 = 'h'
- 'r' (0x72) XOR 6  = 0x74 = 't'
- 'd' (0x64) XOR 2  = 0x66 = 'f'

**Answer:** `L+*Is\`ahtf`

---

## Phase 4

I used `s sym.phase_4` then `pdf` to disassemble the function.
The format string `%d %c %d` revealed the input must be two
integers with a character in between. The first number is used
as an index (0-7) into a jump table — this is a switch statement
in assembly. Values outside 0-7 triggered `explode_bomb` via
an `ja` (jump if above) instruction.

I ran `pxw 32 @ 0x5fc817c572d8` to read the jump table offsets
and traced each case to find its expected character and third
number. Each case loaded a character into `%eax` and compared
the third number against a hardcoded value. At the end, both
the character and third number were checked against `%eax` and
`arg_4h` respectively.

The full table of valid inputs:

| First # | Char | Third # |
|---------|------|---------|
| 0 | x | 771 |
| 1 | w | 726 |
| 2 | g | 144 |
| 3 | f | 135 |
| 4 | z | 707 |
| 5 | u | 232 |
| 6 | t | 925 |
| 7 | u | 224 |

Any valid row from the table works as the answer.

**Answer:** `0 x 771`

---

## Phase 5

I used `s sym.phase_5` then `pdf` to disassemble the function.
The format string showed it reads 2 numbers with sscanf. The
first number must satisfy `(first - 8) <= 6`, meaning it must
be between 8 and 14. The first number is then passed to `func5`
and the result must equal `0x16` (22). The second number must
also equal `0x16` (22).

Looking at the `func5` disassembly from earlier, it is a
recursive function:
```
func5(n):
  if n <= 1: return n
  else: return func5(n/2) + n
```

I tested values between 8 and 14 until the result equaled 22:
- func5(12) = func5(6) + 12
- func5(6)  = func5(3) + 6
- func5(3)  = func5(1) + 3 = 1 + 3 = 4
- func5(6)  = 4 + 6 = 10
- func5(12) = 10 + 12 = 22 ✓

So the first number is 12 and the second number is 22.

**Answer:** `12 22`

---

## Phase 6

I used `s sym.phase_6` then `pdf` to disassemble the function.
The assembly called `read_six_numbers` for 6 unique integers
each between 1 and 6. These numbers are used to reorder the
nodes of a linked list (node1 through node6). After reordering,
the code walks the linked list and checks that each node's value
is greater than or equal to the next node's value — meaning the
list must end up in descending order.

I read each node's value using `pxw 8 @ obj.nodeX`:
- node1 = 0x174 = 372
- node2 = 0x375 = 885
- node3 = 0x266 = 614
- node4 = 0x1bf = 447
- node5 = 0x2ee = 750
- node6 = 0x6b  = 107

Sorting in descending order: 885, 750, 614, 447, 372, 107
Which corresponds to nodes:    2,   5,   3,   4,   1,   6

**Answer:** `2 5 3 4 1 6`
