LAB5 Binary Bomb Report  

Bomb Number: 50 

## Phase 1

First thing I did was look at `sys.phase_1`. The code was calling `strings_not_equal` to compare my input to a hardcoded string. The string was 
just sitting right there in the r2 output next to the leaq line:
"I am for medical liability at the federal level."
I just typed the string exactly as it appeared.


Answer: I am for medical liability at the federal level.

## Phase 2

I went to `sys.phase_2`. I could see it called read_six_numbers so I knew
I needed to enter 6 numbers. The first two were hardcoded checks. The first number
had to be 0, and the second had to be 1. Then there was a loop that kept checking
each next number. I traced through the loop instructions and figured out the
pattern was:

next = 2 * (current + previous)

So I just calculated each number from there:  

start with 0, 1  
2*(0+1) = 2  
2*(1+2) = 6  
2*(2+6) = 16  
2*(6+16) = 44  

Answer: 0 1 2 6 16 44

## Phase 3

I looked at `sys.phase_3`. The first thing it does is check that your input 
is exactly 10 characters long - if it's not, boom. 

Then I noticed it had a list of numbers stored on the stack: 
[9, 5, 10, 1, 6, 2, 3, 9, 6, 2]. It was using these to XOR against my input 
and checking the result against a hidden table. XOR basically just flips bits 
around, but it also works backwards, too. So if you 
know what the result is supposed to be and what number it was XORed with, you 
can figure out the original input.

I ran `px 10 @ obj.t.0` to peek at the table and it contained: E. Hubbard.  
So I just did the XOR backwards on each character to figure out what I needed 
to type:  

E XOR 9 = L  
. XOR 5 = +  
space XOR 10 = *  
H XOR 1 = I  
u XOR 6 = s  
b XOR 2 = `  
b XOR 3 = a  
a XOR 9 = h  
r XOR 6 = t  
d XOR 2 = f  

Answer: L+*Is`ahtf

## Phase 4

I went into `sym.phase_4`. I could see that the format string was 
`%d %c %d` so the input is a number, a character, then another number. The 
first number gets used as an index into a jump table. It's like a switch statement where 
0-7 leads to a different case. I ran `pxw 32 @ 0x5fc817c572d8` to read the jump table and traced case 0 to find 
that it expected the character `x` and the third number `771`. Since any valid 
case works, I just went with that one.

Answer: 0 x 771

## Phase 5

I went into `sys.phase_5` and it reads 2 numbers. The first number has to pass the check
(first - 8) <= 6 so it has to be somewhere between 8 and 14. Then it passes
that number into func5 and the result has to equal 0x16 which is 22. The second
number also just has to be 22.

I tried numbers 8-14 until I got 22 back:  
func5(12) calls func5(6) then adds 12  
func5(6) calls func5(3) then adds 6  
func5(3) calls func5(1) then adds 3, gets 4  
func5(6) = 4 + 6 = 10  
func5(12) = 10 + 12 = 22 

Answer: 12 22

---

## Phase 6

I went into `sys.phase_6`. It reads 6 unique numbers between 1-6. Those numbers are
used to reorder a linked list of 6 nodes. After reordering, it walks the list
and checks that each node's value is less than or equal to the next one, so
the list needs to be in ascending order.

I read each node's value like:  
node1 = 0x174 = 372  
node2 = 0x375 = 885  
node3 = 0x266 = 614  
node4 = 0x1bf = 447  
node5 = 0x2ee = 750  
node6 = 0x6b = 107  

Sorted ascending: 107, 372, 447, 614, 750, 885  
The nodes in order: 6, 1, 4, 3, 5, 2

Answer: 6 1 4 3 5 2
