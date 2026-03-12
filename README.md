Here's the rewritten version in a more casual student voice:
markdown# LAB5 Binary Bomb Report
Bomb Number: 50
Username: nichoch

---

## Phase 1

I seeked to phase_1 using `s sym.phase_1` and ran `pdf` to see the assembly.
It was pretty straightforward - I could see it was calling `strings_not_equal`
and comparing my input to a string that was just sitting right there in the
r2 annotation on the leaq line. The string was 
"I am for medical liability at the federal level."
If my input didn't match it would hit the jne and jump straight to explode_bomb,
so I just typed the string exactly as it appeared.

Answer: I am for medical liability at the federal level.

---

## Phase 2

Seeked to phase_2 and ran pdf. I could see it called read_six_numbers so I knew
I needed to enter 6 numbers. The first two were hardcoded checks - first number
had to be 0 and second had to be 1. Then there was a loop that kept checking
each next number. I traced through the loop instructions and figured out the
pattern was:

next = 2 * (current + previous)

So I just calculated each number from there:
- start with 0, 1
- 2*(0+1) = 2
- 2*(1+2) = 6
- 2*(2+6) = 16
- 2*(6+16) = 44

Answer: 0 1 2 6 16 44

---

## Phase 3

Ran pdf on phase_3. First thing I noticed was it called string_length and
compared to 0xa which is 10, so my input had to be exactly 10 characters.
Then I saw it loading a bunch of values onto the stack - those turned out
to be XOR values: [9, 5, 10, 1, 6, 2, 3, 9, 6, 2].

There was a loop that XORed each character of my input with those values
and compared against a lookup table. I ran `px 10 @ obj.t.0` to see what
was in the table and got: E. Hubbard

To get the answer I just reversed the XOR for each character
(if a XOR b = c, then c XOR b = a):

- E (0x45) XOR 9  = L
- . (0x2E) XOR 5  = +
- space (0x20) XOR 10 = *
- H (0x48) XOR 1  = I
- u (0x75) XOR 6  = s
- b (0x62) XOR 2  = `
- b (0x62) XOR 3  = a
- a (0x61) XOR 9  = h
- r (0x72) XOR 6  = t
- d (0x64) XOR 2  = f

Answer: L+*Is`ahtf

---

## Phase 4

Ran pdf on phase_4. I could see the format string was %d %c %d so the input
is a number, a character, then another number. The first number gets used as
an index into a jump table (basically a switch statement). I ran
`pxw 32 @ 0x5fc817c572d8` to read the jump table and then traced each case
to find what character and third number each one expected.

Each case set a character in %eax and compared the third number to a hardcoded
value. I mapped them all out:

0 -> x, 771
1 -> w, 726
2 -> g, 144
3 -> f, 135
4 -> z, 707
5 -> u, 232
6 -> t, 925
7 -> u, 224

Any of those combinations work so I just picked the first one.

Answer: 0 x 771

---

## Phase 5

Ran pdf on phase_5. It reads 2 numbers. The first number has to pass the check
(first - 8) <= 6 so it has to be somewhere between 8 and 14. Then it passes
that number into func5 and the result has to equal 0x16 which is 22. The second
number also just has to be 22.

I looked at func5 and it was recursive. Basically it does:
- if n <= 1, return n
- otherwise return func5(n/2) + n

I just tried numbers 8-14 until I got 22 back:
- func5(12) calls func5(6) then adds 12
- func5(6) calls func5(3) then adds 6  
- func5(3) calls func5(1) then adds 3, gets 4
- func5(6) = 4 + 6 = 10
- func5(12) = 10 + 12 = 22, that's it

Answer: 12 22

---

## Phase 6

Ran pdf on phase_6. It reads 6 unique numbers between 1-6. Those numbers are
used to reorder a linked list of 6 nodes. After reordering, it walks the list
and checks that each node's value is less than or equal to the next one, so
the list needs to be in ascending order.

I read each node's value with pxw:
- node1 = 0x174 = 372
- node2 = 0x375 = 885
- node3 = 0x266 = 614
- node4 = 0x1bf = 447
- node5 = 0x2ee = 750
- node6 = 0x6b = 107

Sorted ascending: 107, 372, 447, 614, 750, 885
That's nodes in order: 6, 1, 4, 3, 5, 2

Answer: 6 1 4 3 5 2
