# BAS64 - BASIC Assembler 64 macro assembler

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

## The Assenbler

The Assembler assembles an already prepared assembler file to a .PRG file on disk. It is a 2 pass symbolic assembler. It supports 77 assembly language instructions (legal and illegal, except for JAM), 13 addressing modes, 4 pseudo instructions, rudimentary expressions and number input in decimal or hexadecimal.

The preserve as much memory as possible for the symbols, only the symbol, mnemonic and opcode tables are stored in memory. Assembled machine code is written immediately to disk.

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
between the PC start value and End of assembler file marker go the lines of instructions, which can be lines of symbol instructions or assembler instructions.

**Symbol Instructions**

`@symbol=n`
: create symbol named @symbol with value of n

`@symbol`
: create symbol named @symbol with value of the currect PC value

**Assembler Expressions**

In assembler instructions a rudimentary expression language is supported.

`@*`
: value of PC, address of the assembly instruction being compiled, read only

`@*+n`, `@*-n`, `@symbol+n`, `@symbol-n`, `n+n` and `n-n`
: offset by n

`>`, `<`
: hi/lo byte, can preceed any of the above, has the lowest precedence

`expr`
: an expression is any combination of the above

**Assembler Instructions**

Assembler instructions can be pseudo instructions or assembly language instructions.

*Pseudo Instructions*

`.bl n`
: a block of n number of zero bytes, can also be used to leave out space in disk

`.ii ""`
: place petscii code of chars inside the quotation marks to disk

`.sc ""`
: place screen display code of chars inside the quotation marks to disk

`.by expr. ... expr.`
: place lo byte of the values of dot separated expressions to disk

*Assembly Instructions*

List of supported mnemonics can be found near to the end of the BASIC program.

```
1 	clc 		implied
2 	lsa a 		accumulator
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
