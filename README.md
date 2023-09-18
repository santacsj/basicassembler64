# BAS64 - BASIC Assembler 64

BASIC Assembler 64, or BAS64 in short, is a macro assembler for the Commodore 64 written entirely in BASIC V2.

BAS64 supports all legal and illegal opcodes except for JAM. It is entirely disk based, only lookup tables are stored in memory. The package also contains a line oriented SEQ file editor to edit source code, but any other SEQ file editor can be used, of course e.g. Kwik Write! by Fairlight!.

## The Reader

The Reader does text processing. It reads the source file and prepares the assembler file for the Assembler. It does macro processing, comment, whitespace and empty line eliminiation in unquoted text.

Macro definitions are _not_ stored in memory and only read from disk at expansion time. Meaning, macro definition length does not affect memory usage, only the number of macros does as references to definitions _are_ stored in memory. The limit of number of macros is 32 by default, it's easy to change.

### Usage
`LOAD "BAS64.RDR??",8:RUN`, where ?? is the latest version number

### Input
`.S`
: source file, a SEQ file contaning text in Reader syntax, e.g. `RASTERBAR.S`

### Syntax

`;`
: a comment, everything after `;` is consider a comment, no block comments

`!incl <filename, w/o extension>`
: include another .S file, single level only

`!defm <macroname>`
: start macro definition, should be on its own line, single level only

`?n`
: inside macro definition, where n is [0-9], reference argument number 0 to 9

`!endm`
: end macro definition, should be on its own line

`!<macroname> <a0>.<a1>.<a2>.<a3>.<a4>.<a5>.<a6>.<a7>.<a8>.<a9>`
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

!defm repeat
lda ?0
ldx #?1
?2
jsr @chrout
dex
bne ?2
!endm
```
MAIN.S:
```
; main.s
;
*=828
!incl repeat

@main
!repeat @txt.3.
rts

@txt
.ii "A "
```
MAIN.A:
```
*=828
@chrout=$ffd2
@main
lda@txt
ldx#3
@m0repeat02
jsr@chrout
dex
bne@m0repeat02
rts
@txt
.ii"A "
.
```

## The Assembler

The Assembler assembles an already prepared assembler file to a .PRG file on disk. It is a 2 pass symbolic assembler. It supports 77 assembly language instructions (legal and illegal, except for JAM), 13 addressing modes, 4 pseudo instructions, rudimentary expressions and number input in decimal or hexadecimal.

Only the symbol, mnemonic and opcode tables are stored in memory to preserve as much memory as possible for the symbols. Assembled machine code is written immediately to disk.

### Usage
`LOAD "BAS64.ASS??",8:RUN`, where ?? is the latest version number

### Input
`.A`
: assembler file, a SEQ file contaning text in Assembler syntax

### Syntax

`n(nnnn)`
: a decimal number

`$n(nnn)`
: a hexadecimal number

`n`
: a number, decimal or hexadecimal

`*=n`
: assign PC start value, an .A file should start with this

`.`
: end of an assembler file, all .A files should terminate with this

EMPTY.S:
```
*=$033c
.
```
between the PC start value and end of assembler file marker go the lines of instructions, which can be lines of symbol instructions or assembler instructions, in their own separate lines.

**Symbol Instructions**

`@symbol=n`
: create symbol named @symbol with value of n

`@symbol`
: create symbol named @symbol with the current value of the PC

**Assembler Expressions**

In assembler instructions a rudimentary expression language is supported.

`@*`
: the current value of the PC, address of the assembly instruction being compiled, read only

`@*+n`, `@*-n`, `@symbol+n`, `@symbol-n`, `n+n` and `n-n`
: offset by n

`>`, `<`
: hi/lo byte, can preceed any of the above or just a number n, has the lowest precedence

`expr`
: an expression is any combination of the above 3 or just a number n

**Assembler Instructions**

Assembler instructions can be pseudo instructions or assembly language instructions.

*Pseudo Instructions*

`.bl n`
: a block of n number of zero bytes

`.ii ""`
: place petscii code of chars inside the quotation marks to disk

`.sc ""`
: place screen display code of chars inside the quotation marks to disk

`.by expr. ... expr.`
: place lo byte of the values of dot separated expressions to disk

*Assembly Language Instructions*

Mnemonics of supported instructions can be found in the BASIC program's DATA lines. The following 13 addressing modes are supported, examples show the actual syntax: 
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
- The comma and other separator characters that the BASIC INPUT# instruction recognises should not be used, not in the source code, not in quoted string. Use the .BY pseudo intruction to store it in memory, if needed in texts.


## The Editor - BASIC Ed aka BED

BED is a line oriented SEQ text file editor, similar to UNIX's ed.

Unlike screen editors, it uses commands to edit the text, one line at a time. To see the changes, a print command is used, similar to BASIC LIST. Unlike the BASIC editor, line numbers are not part of the text, they are only used to refer to specific lines, no line number management is required.

### Commands

`l <device>`
: load SEQ file from device, device will be 8 if not provided

`s <device>`
: save to a SEQ on device, device will be 8 if not provided

`p <line number>`
: print 18 lines from line number, or last 18 lines if not provided (if available), at the top and bottom of the screen `@:` shows the line number of the first/last line shown

`a <line number>`
: append new line at line number, last line if not provided

`e <line number>`
: edit line at line number, last line if not provided, trims lines to max 35 chars

`$`
: append petscii char to end of existing line

`+`
: append string to end of existing line

`m n+/-(n)`
: mark lines, n+ from n to eof, n- from sof to n, n+5 from n, mark 5 lines, inclusive, unmarkes marked lines if not provided

`d <line number>`
: delete line at line number, last line if not provided, delete marked lines if marked

`c <line number>`
: copy marked text to line number (above, if exists), if no line number, append to end of text

`n`
: new, clear memory

`?`
: print status, free BASIC bytes, number of lines, does GC so might take a while
