| [:ru:](README_ru.md) | :uk: |
| --- | --- |

# x80 - i8080-PRO<i>perly</i>
#### The RISC-Technology ([Rebus](http://en.wikipedia.org/wiki/Rebus)ed Instruction Set Computer)
The story of tries to build a processor that has intuitive clear instruction set.
- [x] Read the [wikia](http://x80.wikia.com/)
  - [x] i8080 particulary support by commands (compactness, methods, simplicity)
  - [x] i8086 particulary support by assembly (flexibility, pointers, registers)
  - [ ] i8080/i8086/z80 directly binary support is not provided by this project
- [x] [Try :arrow_forward:](http://htmlpreview.github.com/?https://github.com/Alikberov/x80/blob/master/emulator.html?speed=10&reset=1000&cycle=1024&debug=FFFF) online demo code

#### Main plan
- [x] CPU-182:
  - [x] 1-byte relative jumps (Jcnd $-128 .. $+127)
  - [x] 8-bits APR (BH CH DH AL BL CL DL)
  - [x] 16-bits pairs (BX CX DX)
  - [x] 16-bits pointers (IP SP BP SI DI)
  - [x] 16-bits of page RAM [0..65535]
- [x] CPU-18x:
  - [ ] 08x - Null-bits ALU (experience factor)
  - [x] 18x - 8-bits ALU
  - [ ] 28x - 16-bits ALU
  - [ ] 38x - 32-bits ALU
  - [ ] 48x - 64-bits ALU
- [x] CPU-x82:
  - [ ] x80 - Null-modified realize (experience model)
  - [ ] x81 - First edition model
  - [x] x82 - Second edition model
  - [ ] x83 - Thirth edition model
- [ ] CPU-284:
  - [x] 2-bytes relative jumps (Jcnd $-32768 .. $+32767)
  - [x] 16-bits APR (BH CH DH AL BL CL DL)
  - [x] 32-bits pairs (BX CX DX)
  - [x] 24-bits pointers (IP SP BP SI DI)
  - [x] 24-bits of page RAM [0..16777215]
- [ ] CPU-386:
  - [x] 3-bytes relative jumps (Jcnd $-8388608 .. $+8388607)
  - [x] 32-bits APR (BH CH DH AL BL CL DL)
  - [x] 64-bits pairs (BX CX DX)
  - [x] 32-bits pointers (IP SP BP SI DI)
  - [x] 32-bits of page RAM [up to 4 Gb]

No more any x86-crutches, like EAX or RIP. Register names has no changes, but changes binarity.
<hr />
Do You still thinking the i8080-architecture hopeless obsolete? And Z80 - the felicitous clone of i8080?
Modern technologies allow to rebuild all i8080-software for new architecture with minimal difficulty.
Let's try it!
<hr />
### Details
```
00: HLT - It's alright, since NULL - is Final (of text file and any document) by logic;
80: NOP - Arithmetical -(-128) - is overflow and still returns the -128 anyway;
FF: RET - Other Final, since 0xFF is rare code and finally of ordinary numbers.
```
:repeat_one: Representation for i8080-rudiment:
```
   ┌─►RET         Example:
Bx FF:Jcond $+1─┐ 1.RZ
┌──┘┌►Rcond     │ 2.RNZ
└───┴───────────┘ 3.RPO
```
:repeat: Representation for x86-rudiment:
```
Bx FE:Jcond $+0─┐ 1.REPZ
↑ ┌►REPcond     │ 2.REPNZ
└─┴─────────────┘ 3.REPPO
```
:repeat_one: Representation for internal functions as hollow calls:
```
   ┌─►RET         Example:
Bx FF:Ccond $+1─┐ 1.DAA
┌──┘┌► «Other»  │ 2.CLI/STI
└───┴───────────┘ 3.CPUID
```
:repeat: Representation for complex functions as overflow calls:
```
Bx FE:Ccond $+0─┐ 1.WAIT
↑ ┌► «Complex»  │ 2.MUL/DIV
└─┴─────────────┘ 3.MOVS/CMPS
```
<hr />
Many of [:page_facing_up: Instructions Codes](http://htmlpreview.github.com/?http://github.com/Alikberov/x80/blob/master/emulator.html?instructions) aligned for maximal intuitive clear remembering:
```
Ax: Assign to registers / ALU-operations;
Bx: Branches by conditions;
Cx: Increments;
Dx: Decrements;
Fx: Functions.
```

:heavy_check_mark: Suitable solvings (for remembering):
```
AA: ALU - Adding          (ADD);
AC: ALU - Conjunction     (AND);
AD: ALU - Disjunction     (OR);
AE: ALU - Exclusive Or    (XOR);
... . . . . . . . . . . . . . .
BA: Branch if Altered sign (minus)
BB: Branch if Bigger (not minus)
BC: Branch if Carry
BD: Branch is "Discarry"
BE: Branch if Equal (Zero)
BF: Branch if Fictive (not equal)
... . . . . . . . . . . . . . .
CB: inCrement BX-pointer;
CC: inCrement CX-pointer;
DC: Decrement CX-pointer;
DD: Decrement DX-pointer;
... . . . . . . . . . . . . . .
CF: Carry Flag complement (CMC);
DF: Digits Flip over      (NOT);
EF: Envelope for Function (LOOP);
F0..F7: Function #n       (INT n);
FF: Function's Finish     (RET);
```
:grey_question: Experimental solves:
```
   ┌──┬──┬──┬──┬──┬──┬──┬──┐
AH:│??│??│TF│WF│PF│SF│CF│ZF│
   └──┴──┴──┴──┴──┴──┴──┴──┘
Flags Register (AH - ALU Heap):
Bit 0:Zero  (Null is Zero);
Bit 1:Carry (1 bit to carrier);
Bit 2:Signed(n - 2n = -n);
Bit 4:Word  (1<<4 = 16 bits mode);
Bit 5:Trace (1<<5 = Thirty two);
```
:grey_question: System сode particularity:
```
00: Prefix SS (not HLT);
```
### Shifted ALU
Instruction set allow to do reverse ALU-operations with accumulator (SUB,SBB,CMP):
```
   5B   |SUB AL,BL   ; AL = AL - BL
   AB 7F|SUB AL,0x7F ; AL = AL - 0x7F
33 5B   |SUB DH,BL   ; DH = DH - BL
33 AB 7F|SUB DH,0x7F ; DH = DH - 0x7F
44 5B   |SUB BL      ; AL = BL - AL
44 AB 7F|SUB 0x7F    ; AL = 0x7F - AL
.. .. ..
   5F   |CMP AL,BL   ; Test(AL - BL)
   AF 7F|CMP AL,0x7F ; Test(AL - 0x7F)
33 5F   |CMP DH,BL   ; Test(DH - BL)
33 AF 7F|CMP DH,0x7F ; Test(DH - 0x7F)
44 5F   |CMP BL      ; Test(BL - AL)
44 AF 7F|CMP 0x7F    ; Test(0x7F - AL)
```
Since other reverse operations is nonsense (ADD,ADC,AND,OR,XOR), it working like comparators:
```
   5C   |AND AL,BL   ; AL = AL & BL
   AC 7F|AND AL,0x7F ; AL = AL & 0x7F
33 5C   |AND DH,BL   ; DH = DH & BL
33 AC 7F|AND DH,0x7F ; DH = DH & 0x7F
44 5C   |AND BL      ; Test(BL & AL)
44 AC 7F|AND 0x7F    ; Test(0x7F & AL)
```
<hr />
## System Tricks
### In/Out ports
All relative access by index pointers, where effective vector are escaped the 64K-bound, will processes like the in/out-ports.
```
        MOV  AL,[SP-1] ; Equal IN AL,255 if SP is 0x0000
        MOV  [SP+1],AL ; Equal OUT 1,AL if SP is 0xFFFF
```
### Processors System Control
Processor haven't special instructions to control for inner states of unit.
All PUSH/POP instructions provides access to internal register, when stack pointer aligned to bound of memory.
When the SP sets in NULL or 0FFFFh, the PUSH/POP-instructions working without RAM, but with internal services registers of processor unit (debug register, task-context register, i8080-/z80-emulation).
As a rule, supervisors process must to control this actions from applications and stop it any or making self.
```
	PUSH DX ; If SP is 0x0000, DX are writing to service registers
	PUSH AX ; Skip current service register
	POP  DX ; If SP is 0xFFFF, DX be readed from service registers
	POP  AX ; Skip current service register
	PUSH CX ; If SP is 0x0001, CX has indexing the page of service
```
Particulary, this is different way for access to in/out ports.
In case, when services page included the hexadecimal capital ("A".."F"), it are disabled for application code as system page.
