# Appendix A: Opcode dictionary v0.1

This appendix is the one place the final operation numbers live. Every operation in the shared middle form from Chapter 10 is here, with the number a Riza file actually uses for it, and no operation is here that the chapters did not name first.

The chapters before this one used other numbers, and said so every time. Section 4.2 introduced the encoding with illustrative values: LOAD_CONST as `10`, ADD as `20`, PRINT as `40`. Section 7.1 added CALL as `60` and RETURN as `61`, marked as illustrations in the same way. Section 10.2 added LOAD_DATA as `12`, following the load family. Those values showed the shape of the scheme, so the worked examples could be counted by hand, and the chapters said plainly, each time, that they were not the final codes. Chapter 12 stated where the final codes would come from: the worked examples use the illustrative operation values from sections 4.2 and 7.1, not the final codes, which come from measured frequency and live in Appendix A. This is that appendix, and these are those codes. From here on, the illustrative values are retired. A file written for version 1 of the system uses the numbers below. The worked examples in the chapters keep their illustrative bytes, correct as illustrations of the scheme and superseded as numbers.

The v0.1 in the title is the dictionary's version, the first one published, not a warning that the dictionary is unfinished. It is finished for version 1 of the system. It is versioned because section 4.1 designed the dictionary to be versioned: the ranking follows the counts, the counts move as programs change, and a new ranking is a new version, made in the open, one version at a time.

These numbers are fixed for version 1's files. A file's header carries the version of the dictionary it was written against, in the version byte Chapter 9 defined, and the loader reads that byte and uses that dictionary. A later dictionary can therefore never silently change what an existing file means: the file names its own dictionary, and the old dictionary is kept. Changing a number in this appendix is not an edit. It is a layout-grade decision, made the way Chapter 34 requires numbers to change: in the open, argued with a chapter's weight of argument, with the reason written down.

## How the numbers were assigned

The rule is Chapter 4's rule: the most used operation gets the shortest code, and here, where every operation in the set fits in the one byte tier, shortest means the lowest number first. The most used operation gets number 0, the next gets number 1, and the ranking runs down the table in order of use.

The evidence is the book's own worked examples. Three programs in this book were worked all the way through in middle form, operation by operation, with every byte counted: the hello loop in section 4.4, the branch in section 7.2, and the add program in section 12.1, in the plain compiler's form before the optimizer's passes. Between them they use an operation 44 times. The count below is those 44 uses, program by program, so any reader can recount it from the three sections and check the ranking it produces.

| Operation | Hello loop, 4.4 | Branch, 7.2 | Add program, 12.1 | Total uses | Code, decimal | Code, hex |
| --- | --- | --- | --- | --- | --- | --- |
| LOAD_CONST | 3 | 2 | 4 | 9 | 0 | 00 |
| LOAD | 2 | 1 | 5 | 8 | 1 | 01 |
| STORE | 2 | 1 | 4 | 7 | 2 | 02 |
| PRINT | 1 | 2 | 1 | 4 | 3 | 03 |
| LOAD_STR | 1 | 2 | 0 | 3 | 4 | 04 |
| ADD | 1 | 0 | 2 | 3 | 5 | 05 |
| END | 1 | 1 | 1 | 3 | 6 | 06 |
| LESS_THAN | 1 | 1 | 0 | 2 | 7 | 07 |
| JUMP_IF_FALSE | 1 | 1 | 0 | 2 | 8 | 08 |
| JUMP | 1 | 0 | 0 | 1 | 9 | 09 |
| CALL | 0 | 0 | 1 | 1 | 10 | 0A |
| RETURN | 0 | 0 | 1 | 1 | 11 | 0B |
| LOAD_DATA | 0 | 0 | 0 | 0 | 12 | 0C |
| SUBTRACT | 0 | 0 | 0 | 0 | 13 | 0D |
| MULTIPLY | 0 | 0 | 0 | 0 | 14 | 0E |
| DIVIDE | 0 | 0 | 0 | 0 | 15 | 0F |
| EQUAL | 0 | 0 | 0 | 0 | 16 | 10 |
| Total | 14 | 11 | 19 | 44 | | |

Say plainly what this evidence is, and what it is not. It is the book's own worked examples, counted by hand, the same examples the earlier chapters counted byte by byte so any reader could check them, and any reader can recount the table above from sections 4.4, 7.2, and 12.1 and get the same numbers. It is not a count over a body of real programs, because no such body exists yet. Section 4.1 designed the ranking to come from the compiler's tally, counted as it compiles, over the standard library, the tools, and the programs people actually write and run, and the compiler that keeps that tally, designed in section 11.1, is not built. When real programs exist, a dictionary version 0.2 earns its numbers from that tally, argued in the open the way Chapter 34 requires any number to change. Until then, the examples are the only honest evidence the book has, and this appendix uses them rather than inventing a body of programs it does not have.

Where the counts tie, the order is the order of section 10.2's groups: loads before stores, stores before arithmetic and comparison, then jumps, then calls and returns, then the operations that touch the world, then the end. Inside a group, the order is the order section 10.2 names the operations. That is why LOAD_STR, with 3 uses, stands before ADD, also with 3, and why LESS_THAN stands before JUMP_IF_FALSE at 2 uses each.

One result will surprise a reader who knows the chapters' prose, and it is stated plainly rather than smoothed over. Sections 2.1 and 4.1 name add among the operations that appear again and again, and in these examples PRINT, with 4 uses, outranks ADD, with 3, because two of the three example programs exist in order to print something. The count is the count. This appendix follows the count, notes the reason, and leaves the correction, if real programs correct it, to the compiler's tally in a later version.

Five operations were named by the chapters and used in no worked example. Four of them, SUBTRACT, MULTIPLY, DIVIDE, and EQUAL, were named in passing, each inside one sentence of section 10.2 and nowhere else in the book. They are in this dictionary because Chapter 10 named them as operations of the form, and Chapter 11 counted them in its table of seventeen and gave each one a test. The fifth, LOAD_DATA, was named and explained in section 10.2's loads paragraph. The examples are small, and their smallness is the whole reason the five show no uses: the example files have empty data sections, the example arithmetic only ever adds, and the example tests only ever ask less than. The five take the numbers after the counted twelve, in section 10.2's order, with LOAD_DATA first among them, because the loads are the most used family in the count: 20 of the 44 uses are loads, 9 plus 8 plus 3.

The code space keeps the shape section 4.2 fixed. A byte holds 256 values. Values 0 to 254 are the one byte tier, 255 codes, and this version assigns 17 of them, 0 to 16. Values 17 to 254, 238 codes, are unassigned and reserved: they are where a later version puts an operation promoted from the long tail on measured frequency, or a new operation that has earned a short code by count, without renumbering anything already assigned. Value 255 is the two byte marker, never an operation by itself: 255 followed by a byte from 0 to 254 names one of 255 more operations, none of them assigned in this version, the whole tier reserved. The values 255 then 255 are the long tail marker: two more bytes after them name one of 65,536 rare operations, none assigned in this version, and the long tail is where new operations go first, the way section 4.2 designed. Reserved does not mean free for private use. Section 9.4's checks walk the code before anything runs, and a code value the file's own dictionary version does not define is a file that fails its checks, so an unassigned value in a version 1 file is a broken file, not an extension. And a number, once this dictionary assigns it, keeps its meaning in every later version. If a later version ever removes an operation, its number is retired with it, never handed to a different operation, because a number that changed its meaning would silently change every file that holds it, and silence of that kind is the one thing the version byte exists to prevent.

## The dictionary

The tables below group the operations the way Chapter 10 grouped them and Chapter 11 counted them: loads, stores, arithmetic and comparison, jumps, calls and returns, the operations that touch the world, and the end. Inside each group the entries stand in number order. Each entry gives the operation's name exactly as the chapters wrote it, its final number in decimal and in hex, its length in bytes counting every byte that follows the code, what it takes off the pile, what it leaves on the pile, and one sentence saying what it does.

### Loads

| Operation | Code | Length | Takes from the pile | Leaves on the pile | What it does |
| --- | --- | --- | --- | --- | --- |
| LOAD_CONST | 0 (hex 00) | 2 bytes | Nothing | One value, the number | Puts the small number stored in the byte right after the code on the pile. |
| LOAD | 1 (hex 01) | 2 bytes | Nothing | One value, the name's current value | Puts the value held in the slot of the name named by the byte after the code, a name table number, on the pile. |
| LOAD_STR | 4 (hex 04) | 2 bytes | Nothing | One value, the string | Puts the string stored in the name table entry named by the byte after the code on the pile. |
| LOAD_DATA | 12 (hex 0C) | 5 bytes | Nothing | One value, the static value | Puts the static value that sits at the position named by the four bytes after the code, counted in bytes from the start of the data section, on the pile; the type byte at that position says the value's kind and size. |

One width in this group needs its reason stated, because no worked example used the operation. Every other operand in this dictionary is one byte, because everything else an operand names is counted by a one byte count: a file holds at most 255 names, the way section 9.2 stated, so a name number fits in a byte, and a function number fits the same way. A position in the data section names a place in a section whose size the header states in four bytes, and Chapter 9 uses four bytes for every position that can run past 255: the entry point, the section offsets, and the positions in the function mapping and the comments table. LOAD_DATA's position follows the format's own pattern, four bytes, and the entry above says so in the open rather than leaving the width to be guessed.

### Stores

| Operation | Code | Length | Takes from the pile | Leaves on the pile | What it does |
| --- | --- | --- | --- | --- | --- |
| STORE | 2 (hex 02) | 2 bytes | The top value | Nothing | Takes the top value off the pile and stores it in the slot of the name named by the byte after the code, replacing the value that was there. |

### Arithmetic and comparison

| Operation | Code | Length | Takes from the pile | Leaves on the pile | What it does |
| --- | --- | --- | --- | --- | --- |
| ADD | 5 (hex 05) | 1 byte | Two values | One value, their sum | Takes two values off the pile and puts their sum back. |
| LESS_THAN | 7 (hex 07) | 1 byte | Two values | One value, a true or false | Takes two values off the pile and puts back true if the first value is less than the second, and false if it is not. |
| SUBTRACT | 13 (hex 0D) | 1 byte | Two values | One value, the difference | Takes two values off the pile and puts back the first value minus the second. |
| MULTIPLY | 14 (hex 0E) | 1 byte | Two values | One value, the product | Takes two values off the pile and puts back the first value multiplied by the second. |
| DIVIDE | 15 (hex 0F) | 1 byte | Two values | One value, the quotient | Takes two values off the pile and puts back the first value divided by the second. |
| EQUAL | 16 (hex 10) | 1 byte | Two values | One value, a true or false | Takes two values off the pile and puts back true if the two values are the same, and false if they are not. |

First means the value put on the pile first, the deeper of the two, the order section 10.3's translation fixed when it popped the second value into a register and compared it against the first. The arithmetic follows the types the values carry, from Chapter 5, and a result too big for its type is the error of sections 5.4 and 7.3, a value the program can see and handle, never a number that quietly wraps around.

### Jumps

| Operation | Code | Length | Takes from the pile | Leaves on the pile | What it does |
| --- | --- | --- | --- | --- | --- |
| JUMP_IF_FALSE | 8 (hex 08) | 2 bytes | The top value, the test | Nothing | Takes the top value off the pile, and if it is false, moves the program to the position named by the byte after the code; if it is true, the program goes on to the next operation. |
| JUMP | 9 (hex 09) | 2 bytes | Nothing | Nothing | Moves the program to the position named by the byte after the code, forward or back, without testing anything. |

A target is a position in the code section, counted in bytes from its start, starting at 0, the one way of naming a place in the code that every part of the system uses, the way sections 4.4 and 10.2 counted them. The target is one byte, the width every worked example used and the width section 7.1 stated in words: a jump is followed by a byte naming where to jump to.

### Calls and returns

| Operation | Code | Length | Takes from the pile | Leaves on the pile | What it does |
| --- | --- | --- | --- | --- | --- |
| CALL | 10 (hex 0A) | 2 bytes | The function's input values, placed on the pile by the caller | Nothing of its own | Moves the program to the code of the function named by the byte after the code, a function number whose start position the file's function mapping states; the values already on the pile are the function's to work on. |
| RETURN | 11 (hex 0B) | 1 byte | The function's result, on top of its working values | One value, the result | Ends the function: the function's own values are freed in one act by moving the stack pointer back, the result stays on the pile for the caller, and the program goes on at the operation after the CALL. |

### Touching the world

| Operation | Code | Length | Takes from the pile | Leaves on the pile | What it does |
| --- | --- | --- | --- | --- | --- |
| PRINT | 3 (hex 03) | 1 byte | The top value | Nothing | Takes the top value off the pile and prints it. |

PRINT is the only operation in this group the chapters named, and section 11.3 said so plainly. The rest of the world, reading and writing files, asking the time, and the rest of the core library's work from Chapter 16, is reached the way section 10.2 designed: the middle form has one way to call the library, and the library has the rest. The chapters never gave that call an operation name of its own, so it has no entry and no number in this dictionary. The numbers it works by are the library's routine numbers, and those live in Appendix C with the middle form spec, not here.

There is likewise no error operation in this dictionary, and that absence is a design, not an omission. Section 7.3 made an error a value, and values are loaded, stored, and tested by the operations above. No operation raises or throws, because nothing in the middle form jumps in secret.

### The end

| Operation | Code | Length | Takes from the pile | Leaves on the pile | What it does |
| --- | --- | --- | --- | --- | --- |
| END | 6 (hex 06) | 1 byte | Nothing | Nothing | Stops the program; in translation it becomes a call to the loader's finish routine, which ends the program cleanly, the way Chapter 15 designs shutdown. |

## Reading a file with this dictionary

A dictionary is only useful if a reader can pick up a byte string and read it. Take the add program from section 12.1, the plain compiler's middle form, 19 operations and 33 bytes of code, which the book counted under the illustrative values. Under this dictionary, the same program reads as follows. Only the code bytes change. The operand bytes, the names they point at, and every position stay exactly where section 12.1 put them.

| Scheme | The 33 bytes of code |
| --- | --- |
| Illustrative, section 12.1 | `10 63 30 04 10 02 10 03 20 30 05 31 05 10 04 60 00 30 06 31 06 40 00 31 01 31 02 20 30 03 31 03 61` |
| Final, this dictionary | `00 63 02 04 00 02 00 03 05 02 05 01 05 00 04 0A 00 02 06 01 06 03 06 01 01 01 02 05 02 03 01 03 0B` |

Read the final string from the left. `00` is LOAD_CONST, and the byte after it, `63`, is 99 in ordinary counting, so 99 goes on the pile. `02` is STORE, and `04` names name 4, `spare`, so the 99 is stored and the pile is empty. `00 02` and `00 03` put 2 and 3 on the pile, and `05`, ADD, takes them off and puts back 5. The reading goes on in the same way, one code byte at a time, each code saying how many bytes follow it, until `0B`, RETURN, at the last position. The program is the program section 12.1 counted, and it prints 9 under both schemes.

The count does not change, and the difference is stated plainly: there is none in size. The code is 33 bytes under the illustrative values and 33 bytes under the final numbers, because renumbering changes what the code bytes are, not how many of them there are. Every operation in this program sits in the one byte tier under both schemes, with the same operand widths, so every byte lands in the same position and the positions a reader checks against section 12.1 still check.

One value deserves a warning. Under the illustrative scheme, `00` was END. Under this dictionary, `00` is LOAD_CONST, and END is `06`. A reader who reads a chapter's worked example with this dictionary in hand will misread it, END first of all. That is not a defect in either scheme. It is exactly the confusion the version byte in the file header exists to prevent: a real file names the dictionary it was written against, and the chapters' examples, which are illustrations and name no dictionary, are read with the illustrative values they were written with.

## What is reserved, and what lives elsewhere

The reserved ranges, in one line: codes 17 to 254 in the one byte tier, the whole two byte tier behind the marker 255, and the whole long tail behind the markers 255, 255 are unassigned in version 0.1, held for later versions, and a number once assigned is never reused for a different operation.

The version rule, in one line: these numbers are fixed for version 1's files, a file names its dictionary in its header's version byte, and a later dictionary never changes what an existing file means.

This appendix holds operation numbers only. The rest of the book's final numbers live in the appendices named for them. Appendix B, the binary file format spec, holds the file's fields byte by byte, including the type tag numbers for the type byte that starts each value in the data section, which section 9.2 left to the spec. Appendix C, the middle form spec, holds the form's layout in full, including the core library's routine numbers and the kernel call numbers. A reader who needs a number that is not an operation number will find it in one of those two, not here.

This appendix adds no operation to the set the chapters named, so it adds nothing for the test suite or the library to hold. Chapter 11's seventeen operation tests already test the seventeen operations named here, one each, and a test checks what an operation does, not which number it wears. The suite stays at 217 tests, and the core library stays at 39 routines, the closing counts Chapter 34 left as the first version's state.

---

The dictionary is complete at seventeen operations, the exact set Chapters 4, 7, 10, 11, and 12 named, no more and no fewer. The numbers follow the count of the book's own worked examples, 44 uses across three programs: LOAD_CONST 0, LOAD 1, STORE 2, PRINT 3, LOAD_STR 4, ADD 5, END 6, LESS_THAN 7, JUMP_IF_FALSE 8, JUMP 9, CALL 10, RETURN 11, and the five operations no example used, LOAD_DATA 12, SUBTRACT 13, MULTIPLY 14, DIVIDE 15, EQUAL 16, in the order Chapter 10 named them. The rest of the code space, 238 one byte codes, the two byte tier, and the long tail, stands reserved for later versions, and no assigned number will ever be reused. The illustrative values the chapters taught with are retired, and the add program from Chapter 12 reads the same 33 bytes under the final numbers as it did under them, printing 9 either way.
