# BAS64 - BASIC Assembler 64

BASIC Assembler 64, or BAS64 in short, is a macro assembler for the Commodore 64 written entirely in BASIC V2.

BAS64 supports all legal and illegal opcodes except for JAM. It is entirely disk based, only lookup tables are stored in memory. The package also contains a line oriented SEQ file editor to edit source code, but any other SEQ file editor can be used, of course e.g. Kwik Write! by Fairlight!.

## The Reader

The Reader does text processing. It reads the source file and prepares the assembler file for the Assembler. It does macro processing, comment, whitespace and empty line eliminiation in unquoted text.

Macro definitions are _not_ stored in memory and only read from disk at expansion time. Meaning, macro definition length does not affect memory usage, only the number of macros does as references to definitions _are_ stored in memory. The limit of number of macros is 32 by default, it's easy to change.

### Usage
`LOAD "BAS64.RDR*",8:RUN`

### Input
`.S`
: source file, a SEQ file contaning text in Reader syntax, e.g. `RASTERBAR.S`

### Syntax

`;`
: a comment, everything after `;` is consider a comment

`!incl. <filename, w/o extension>`
: include another .S file, single level only

`!defm. <macroname>`
: start macro definition, should be on its own line, single level only

`?a`
: inside macro definition, where `a` is [0-9], reference argument number 0 to 9

`!endm`
: end macro definition, should be on its own line

`!<macroname>. <a0>.<a1>.<a2>.<a3>.<a4>.<a5>.<a6>.<a7>.<a8>.<a9>`
: expand macro `macroname` with list of actual args separated by `.`, all args are optional, in case no actual argument provided the Reader generates a unique symbol for it, single level only, embedding is not supported

### Output
`.A`
: assembler file, a SEQ file with all macros expanded and all comments/whitespaces eliminiated from unquoted text, `@0:` is used to overwrite file with the same name

### Example
REPEAT.S:
```
; repeat.s
;
@chrout=$ffd2 ;kernal chrout addr

!defm. repeat
[
lda ?0
ldx #?1
?2
jsr @chrout
dex
bne ?2
]
!endm
```
MAIN.S:
```
; main.s
;
*=828
!incl. repeat

@main
!repeat. @txt. 3.
rts

@txt
.ii "A "
```
MAIN.A:
```
*=828
@chrout=$ffd2
@main
[
lda@txt
ldx#3
@m0a2e0
jsr@chrout
dex
bne@m0a2e0
]
rts
@txt
.ii"A "
.
```

## The Assembler

The Assembler assembles an already prepared assembler file to a .PRG file on disk. It is a 2 pass symbolic assembler. It supports 77 assembly language instructions (legal and illegal, except for JAM), 13 addressing modes, 4 pseudo instructions, rudimentary expressions and number input in decimal or hexadecimal.

Only the symbol, mnemonic and opcode tables are stored in memory to preserve as much memory as possible for the symbols. Assembled machine code is written immediately to disk.

### Usage
`LOAD "BAS64.ASS*",8:RUN`

### Input
`.A`
: assembler file, a SEQ file contaning text in Assembler syntax

### Syntax

**Literals**

`n(nnnn)`
: a decimal number e.g. `42`

`$n(nnn)`
: a hexadecimal number e.g. `$ffe`

`"a(aaa)"`
: a string

_Note: from here on where `n` is written it means both a decimal or hexadecimal number can be used and `s` means a string._

**The Program Counter**

`*=n`
: assign PC start value, **all .A files should start with this**

`.`
: end of an assembler file, **all .A files should terminate with this**

Between the PC start value and the end of assembler file marker go the lines of instructions on their own separate lines. Instructions can be symbol instructions or assembler instructions.

**Symbol Instructions**

`@symbol=n`
: create symbol with value of n

`@symbol`
: create symbol with value of current PC value

`[` and `]`
: open/close a lexical closure, every symbol declared within a lexical closure is local to that closure and cannot be referenced from the outside or from another closure, should go on its own line

**Expressions**

`n`
: evaluates to the literal value

`@symbol`
: evaluates to the value of the symbol

`@*`
: evaluates to the the current value of the PC, the address of the assembly instruction being compiled, **read only**

`+` or `-`
: any of the above three can be offseted by a literal `n`, e.g. `40+2`, `@address+1` or `@*-2`

`>`, `<`
: hi/lo byte, can preceed any of the above, has the lowest precedence e.g. `>@address+2`

_Note: from here on where `expr` is written it means a valid expression according to the above rules._

**Assembler Instructions**

Assembler instructions can be pseudo instructions or assembly language instructions.

*Pseudo Instructions*

`.bl n`
: a block of n number of zero bytes

`.ii s`
: place petscii code of chars inside the quotation marks to disk

`.sc s`
: place screen display code of chars inside the quotation marks to disk

`.by expr. expr.`
: place lo byte of the values of dot separated expressions to disk

*Assembly Language Instructions*

Supported mnemonics/opcodes by addressing modes (1-13, see table below):
```
ADC   ,   , 69,   , 65, 75,   , 61, 71, 6D, 7D, 79,   
AHX   ,   ,   ,   ,   ,   ,   ,   , 93,   ,   , 9F,   
ALR   ,   , 4B,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
ANA   ,   ,  B,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
ANB   ,   , 2B,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
ANC   ,   ,  B,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
AND   ,   , 29,   , 25, 35,   , 21, 31, 2D, 3D, 39,   
ARR   ,   , 6B,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
ASL  A,  A,   ,   ,  6, 16,   ,   ,   ,  E, 1E,   ,   
AXS   ,   , CB,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
BCC   ,   ,   , 90,   ,   ,   ,   ,   ,   ,   ,   ,   
BCS   ,   ,   , B0,   ,   ,   ,   ,   ,   ,   ,   ,   
BEQ   ,   ,   , F0,   ,   ,   ,   ,   ,   ,   ,   ,   
BIT   ,   ,   ,   , 24,   ,   ,   ,   , 2C,   ,   ,   
BMI   ,   ,   , 30,   ,   ,   ,   ,   ,   ,   ,   ,   
BNE   ,   ,   , D0,   ,   ,   ,   ,   ,   ,   ,   ,   
BPL   ,   ,   , 10,   ,   ,   ,   ,   ,   ,   ,   ,   
BRK  0,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
BVC   ,   ,   , 50,   ,   ,   ,   ,   ,   ,   ,   ,   
BVS   ,   ,   , 70,   ,   ,   ,   ,   ,   ,   ,   ,   
CLC 18,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
CLD D8,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
CLI 58,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
CLV B8,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
CMP   ,   , C9,   , C5, D5,   , C1, D1, CD, DD, D9,   
CPX   ,   , E0,   , E4,   ,   ,   ,   , EC,   ,   ,   
CPY   ,   , C0,   , C4,   ,   ,   ,   , CC,   ,   ,   
DCP   ,   ,   ,   , C7, D7,   , C3, D3, CF, DF, DB,   
DEC   ,   ,   ,   , C6, D6,   ,   ,   , CE, DE,   ,   
DEX CA,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
DEY 88,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
EOR   ,   , 49,   , 45, 55,   , 41, 51, 4D, 5D, 59,   
INC   ,   ,   ,   , E6, F6,   ,   ,   , EE, FE,   ,   
INX E8,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
INY C8,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
ISC   ,   ,   ,   , E7, F7,   , E3, F3, EF, FF, FB,   
JMP   ,   ,   ,   ,   ,   ,   ,   ,   , 4C,   ,   , 6C
JSR   ,   ,   ,   ,   ,   ,   ,   ,   , 20,   ,   ,   
LAS   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   , BB,   
LAX   ,   , AB,   , A7,   , B7, A3, B3, AF,   , BF,   
LDA   ,   , A9,   , A5, B5,   , A1, B1, AD, BD, B9,   
LDX   ,   , A2,   , A6,   , B6,   ,   , AE,   , BE,   
LDY   ,   , A0,   , A4, B4,   ,   ,   , AC, BC,   ,   
LSR 4A, 4A,   ,   , 46, 56,   ,   ,   , 4E, 5E,   ,   
NOP EA,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
ORA   ,   ,  9,   ,  5, 15,   ,  1, 11,  D, 1D, 19,   
PHA 48,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
PHP  8,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
PLA 68,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
PLP 28,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
RLA   ,   ,   ,   , 27, 37,   , 23, 33, 2F, 3F, 3B,   
ROL 2A, 2A,   ,   , 26, 36,   ,   ,   , 2E, 3E,   ,   
ROR 6A, 6A,   ,   , 66, 76,   ,   ,   , 6E, 7E,   ,   
RRA   ,   ,   ,   , 67, 77,   , 63, 73, 6F, 7F, 7B,   
RTI 40,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
RTS 60,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
SAX   ,   ,   ,   , 87,   , 97, 83,   , 8F,   ,   ,   
SBC   ,   , E9,   , E5, F5,   , E1, F1, ED, FD, F9,   
SEC 38,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
SED F8,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
SEI 78,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
SHX   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   , 9E,   
SHY   ,   ,   ,   ,   ,   ,   ,   ,   ,   , 9C,   ,   
SLO   ,   ,   ,   ,  7, 17,   ,  3, 13,  F, 1F, 1B,   
SRE   ,   ,   ,   , 47, 57,   , 43, 53, 4F, 5F, 5B,   
STA   ,   ,   ,   , 85, 95,   , 81, 91, 8D, 9D, 99,   
STX   ,   ,   ,   , 86,   , 96,   ,   , 8E,   ,   ,   
STY   ,   ,   ,   , 84, 94,   ,   ,   , 8C,   ,   ,   
TAS   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   , 9B,   
TAX AA,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
TAY A8,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
TSX BA,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
TXA 8A,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
TXS 9A,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
TYA 98,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
USB   ,   , EB,   ,   ,   ,   ,   ,   ,   ,   ,   ,   
XAA   ,   , AB,   ,   ,   ,   ,   ,   ,   ,   ,   ,    
```

The following 13 addressing modes are supported, examples show the actual syntax: 
```
1 	clc 		implied
2 	lsr a 		accumulator
3 	lda #expr 	immediate
4 	beq expr        relative

5 	lda *expr 	zero-page absolute
6 	lda *expr.x 	zero-page indexed x
7 	ldx *expr.y 	zero-page indexed y
8 	lda *(expr.x)   zero-page indexed indirect
9 	lda *(expr).y   zero-page indirect indexed

10 	lda expr 	absolute
11 	lda expr.x	absolute indexed x
12 	lda expr.y	absolute indexed y
13 	jmp (expr)	indirect
```
Note: for LSR,ASL,ROR and ROL instructions both accumulator and implied modes are supported and compile to the same opcode.

### Error Codes

The Assembler will stop on errors and print the affected line number and one of the following error codes:
```
1 	No *, start of file not found
2 	Unknown mnemonic
3 	Unknown addressing mode
4 	No such addressing mode for mnemonic
5 	Symbol does not exists
6 	Symbol already exists
7 	Branch out of range
```

### Output
`PRG`
: a PRG file with the name of the .A file, extension excluded, `@0:` is used to overwrite file with the same name

### Hints

- Since no * assignment is supported, the .BL pseudo instruction can be used to leave out bytes between instructions
- The comma and other separator characters that the BASIC INPUT# instruction recognises should not be used, not in the source code, not in quoted strings. Use the .BY pseudo instruction to store it in memory directly

## The Editor - BASIC Ed aka BED

BED is a line oriented SEQ text file editor, similar to UNIX's ed.

Unlike screen editors, it uses commands to edit the text, one line at a time. To see the changes, a print command is used, similar to BASIC LIST. Unlike the BASIC editor, line numbers are not part of the text, they are only used to refer to specific lines, no line number management is required.

### Usage
`LOAD "BAS64.BED*",8:RUN`

### Commands

`l <device>`
: load SEQ file from device, device will be 8 if not provided, load not allowed when there is text in memory, use `n`

`s <device>`
: save to a SEQ file on device, device will be 8 if not provided

`p <line number>`
: print 18 lines from line number, or last 18 lines if not provided, at the top and bottom of the screen `@:` shows the line numbers of the first/last line shown

`a <line number>`
: append new line at line number, last line if not provided

`e <line number>`
: edit line at line number, last line if not provided, trims lines to 35 chars

`$ <line number>`
: append char by petscii code to the end of an existing line

`+ <line number>`
: append string to the end of an existing line

`m n+/-(n)`
: mark lines, n+ from n to eof, n- from sof to n, n+5 mark n and 4 lines below it, n-5 marks n and 4 lines above it, unmarks marked lines if not provided

`d <line number>`
: delete line at line number, last line if not provided, delete marked lines if marked

`c <line number>`
: copy marked lines to line number, append to the end of the text if not provided

`n`
: new, clear all text from memory

`?`
: print free BASIC bytes and number of lines, does GC so might take a while

### Hints

- Last command is not cleared, so pressing Return on an empty command line repeats the last command. It can be used to delete multiple lines, or append at the end of the text while inputting the source code.
