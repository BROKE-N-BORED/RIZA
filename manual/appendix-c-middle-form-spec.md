# Appendix C: Middle form spec

This appendix is the shared middle form stated completely, in one place. Chapter 10 designed the form: its six groups of operations plus the end, its layout in the code section, and the step by step path from the form to the machine's own instructions. This spec states the whole design in the order an implementer needs it, so a reader could write the translator from this appendix alone: the layout, the one way to call the core library with every routine's number, the calls a program can make to the kernel with their numbers, and the translation to native code, operation by operation, as far as the chapters fixed it.

What the chapters left open, this appendix fixes, and the list is short. No chapter ever numbered the core library's routines: section 16.1 named them and said a call names the routine by number, and the numbers are assigned below, in the order the chapters named the routines. No chapter numbered the kernel's calls either: sections 22.2 and 25.1 said the calls are made by number and left the numbering to the appendices, and the numbers are assigned below, in the order the chapters designed the calls. Appendix A gave the library call no operation number and pointed its numbers here, and Appendix B pointed here for the form's layout in full. Nothing else in this appendix is new. Every routine, every call, and every native instruction count below is a chapter's, restated in spec order.

The version rule governs this spec the way it governs the dictionary and the format. The form is versioned with them: a file's header names the dictionary it was written against, that byte is also the file format version, and the middle form is the dictionary's operations in the format's code section, so a change to the form is a change to the dictionary or the format, a new version made in the open, with old files still readable. Every number in this appendix is version 1's number.

## The form's layout

A program in middle form is a list of steps, in order, sitting in the file's code section, the section Appendix B places in the file. Each step is one operation, written as its operation code from Appendix A, with its operands inline right after the code: a load is followed by the number saying what to load, a jump by the number saying where to jump to, a call by the number saying which function to call. The number of operands an operation takes, and the width of each, is fixed by the operation itself, in Appendix A's table. Positions are counted in bytes from the start of the code section, starting at 0, and that is the one way of naming a place in the code that every part of the system uses: jump targets are counted that way, the entry point is counted that way, and the function mapping and the comments in Appendix B point that way.

Each operation code says its own length. The first byte says whether the operation sits in the one byte tier, behind the two byte marker, or in the long tail, by the tiers Appendix A states, and the operation itself says how many operand bytes follow. So a reader can walk forward through the code one operation at a time, without a table and without guessing. That one walk is the validity walk of section 9.4, the walk the translator makes, and the walk the debugger makes, because the form is self describing.

The steps work on a pile, a stack of values, and the pile discipline is fixed here in full. A load puts one value on the pile. A store takes the top value off the pile and puts it in a named slot. An arithmetic operation takes two values off the top and puts one result back. A comparison takes two values off and puts back a true or false, one byte holding 0 or 1, the way section 5.1 defined it. In every two value operation, the first value is the one put on the pile first, the deeper of the two, the order Appendix A fixed. JUMP moves the program to a position without touching the pile. JUMP_IF_FALSE takes the top value off the pile and moves the program to its target position if that value is false, and goes on to the next step if it is true. A jump to a position outside the code section, or to the middle of an operation, is a broken file, and the fifth check in Appendix B catches it at the door. CALL moves the program to a function's code, with the function's input values already on the pile, placed there by the caller. RETURN ends the function: the function's own values are freed in one act, by moving the stack pointer back, the result stays on the pile for the caller, and the program goes on at the step after the CALL. PRINT takes the top value off the pile and prints it. END stops the program, and the translation section below says what it becomes.

Types travel with the operations, and where each type is known from is fixed here. For a value loaded from the data section, the type byte at its position in that section says the kind and the size, by the tags in Appendix B. For a constant, the operation code itself says the size. For a named value, the type comes from the name's definition, because the types are checked at build time, the way Chapter 5 describes and Chapter 11 enforces. An arithmetic or comparison operation follows the type it carries: signed numbers get the signed comparison, unsigned numbers get the unsigned one, the way section 10.3 worked it. The translator never guesses what kind of value it is moving, because the kind is stated, in the operation or in the value.

In translation, the pile lives on the machine's own stack, the stack the processor already has, and the middle form's pile maps straight onto it. Named values live in fixed slots in static memory, one slot per name, sized to the name's type. The processor's registers are used briefly for the work between a pop and a push: a value is popped into a register, worked on, and pushed back. The loader counts the deepest the pile goes on the translator's first pass, and the pile run is sized by that count, the way Chapter 25 designed it.

## The library call

The middle form has one way to call the core library, and the library has the rest. That is section 10.2's design, and it is why the form stays small: reading a file, writing a file, asking the time, and every other act that touches the world is a library routine's work, not a separate operation, so the form does not grow with every device and every act. The chapters never gave that call an operation name of its own, so it has no entry and no number in Appendix A's dictionary. The numbers it works by are the routine numbers in the table below.

The mechanism is fixed here, in the shape the chapters fixed it. A call names its routine by number, the way CALL names a function by number, in one byte, the width CALL's function number takes in Appendix A, because the chapters fixed this call in CALL's shape and 39 routines fit in a byte with room to spare. The values the routine takes are placed on the pile before the call, in the order the table below lists them. The routine's answer comes back on the pile, in the place a result sits: the value the table says it hands back, or an error value if it failed, because an error is a value, the way section 7.3 designed it, 2 bytes holding the code in 1 byte and the message's table number in 1 byte, the way section 16.3 fixed it. A routine that hands back nothing leaves nothing on the pile. The caller checks the answer before using it: the check is the `is_error` routine below, written in the text form as the branch `if answer is error:`, and the compiler refuses a program that uses a result without the check, the way section 16.3 said. Ownership follows section 6.1: a routine that makes a value hands ownership to the caller, a routine that takes a value and gives it back hands ownership back, and a routine that only reads a value borrows it. In translation, the call becomes a native call to the routine's own code, the way section 10.3 showed PRINT becoming a call to the print routine: the routine itself is native code, in the system, shared by every program, the opposite of every app shipping its own copy of the world.

### The routine numbers

The numbers are assigned in the order the chapters named the routines, starting at 0, chapter by chapter, and inside a chapter in the order the chapter's own tables name them. A number, once assigned, keeps its routine in every later version. If a later version ever removes a routine, its number is retired with it, never handed to a different routine. Numbers 39 to 255, 217 numbers, are unassigned and reserved: they are where a later version puts a routine a later chapter earns, without renumbering anything already assigned. Reserved does not mean free for private use: a routine number this version does not assign is a call the system does not know, and a program that makes it has asked for work that does not exist.

Chapter 16 named the first routines, in three sections. The running count at the end of the chapter is 28.

| Number | Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- | --- |
| 0 (hex 00) | print | A value | Nothing, it prints the value | An error, if there is nowhere to print to |
| 1 (hex 01) | join | Two pieces of text | One piece, the two joined | An error, code 4, if the joined text runs out of memory |
| 2 (hex 02) | split | Text and a mark | A list of the pieces | An error, code 4, if the list runs out of memory |
| 3 (hex 03) | compare | Two pieces of text | 0 if the same, below 0 if the first comes first, above 0 if it comes second | Does not fail |
| 4 (hex 04) | find | Text and a piece | The position of the piece, counting from 0 | An error, code 7, if the piece is not in the text |
| 5 (hex 05) | length | Text or a list | The number of characters, or the number of items | Does not fail |
| 6 (hex 06) | text_of | A number | The text of the number | An error, code 4, if the text runs out of memory |
| 7 (hex 07) | append | A list and a value | The list, with the value added at the end | An error, code 4, if the grown list runs out of memory |
| 8 (hex 08) | get | A list and a position | The value at that position, counting from 0 | An error, code 6, if the position is past the end |
| 9 (hex 09) | open_file | A name | The file | An error, code 1, if the file does not exist |
| 10 (hex 0A) | read | An open file | The file's remaining text | An error, code 2, if the file cannot be read |
| 11 (hex 0B) | read_lines | A name | A list of the file's lines | An error, code 1, if the file does not exist; code 2, if it cannot be read |
| 12 (hex 0C) | write | A file and text | The number of bytes written | An error, code 3, if the disk is full |
| 13 (hex 0D) | write_line | A file and text | The number of bytes written, text plus newline | An error, code 3, if the disk is full |
| 14 (hex 0E) | flush | A file | Nothing; the waiting bytes go to the disk | An error, code 3, if the disk is full |
| 15 (hex 0F) | close | A file | Nothing; the file is released | An error, code 3, if the final flush fails |
| 16 (hex 10) | now | Nothing | The current time, as a number of seconds since the start of 1970 | Does not fail: the system always has a clock |
| 17 (hex 11) | wait | A number of seconds | Nothing; it waits that long, then goes on | An error, code 6, if asked to wait less than 0 seconds |
| 18 (hex 12) | elapsed | A time, from `now` | The seconds since that time | An error, code 6, if the time is after now |
| 19 (hex 13) | remainder | Two numbers | What is left after dividing the first by the second | An error, code 9, if the second number is 0 |
| 20 (hex 14) | square_root | A number | The number that multiplied by itself gives the number | An error, code 6, if the number is below 0 |
| 21 (hex 15) | power | A base and a count | The base raised to the count | An error, code 5, if the answer is too big for its type |
| 22 (hex 16) | random_between | Two numbers, a low and a high | A number between them, both ends included | An error, code 6, if the low is above the high |
| 23 (hex 17) | random_choice | A list | One item from the list, picked at random | An error, code 7, if the list is empty |
| 24 (hex 18) | make_error | A code and a message | An error value | Does not fail |
| 25 (hex 19) | is_error | A value | True if the value is an error, false if it is not | Does not fail |
| 26 (hex 1A) | log_write | Text, one line | Nothing; the line goes on the log | An error, code 3, if the disk is full |
| 27 (hex 1B) | log_read | Nothing | The log's lines, as a list of text, in the order they were written | Does not fail |

Two entries in Chapter 16's tables are not new routines, and the count depends on saying so. `length` stands in the chapter's text table and again in its list table, because it works on both; it is one routine, with the one number above, counted once. And section 16.3's third error entry, passing an error up, is not a routine at all, which the chapter states as the point: passing a value up is returning it, and the library needs no name for something the language already does. It has no number here.

Chapter 18 named five more, four in section 18.1 and one in section 18.2, and the file routines grew from 7 to 12. The running count at the end of the chapter is 33.

| Number | Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- | --- |
| 28 (hex 1C) | list_folder | A folder name | A list of records, one per entry, each holding the entry's name, whether it is a file or a folder, and its size in bytes | An error, code 1, if the folder does not exist; code 2, if it cannot be read |
| 29 (hex 1D) | create_file | A name | The new file, empty | An error, code 3, if the disk is full |
| 30 (hex 1E) | move_file | An old name and a new name | Nothing; the file has its new name | An error, code 1, if the source file does not exist |
| 31 (hex 1F) | delete_file | A name | Nothing; the file is gone | An error, code 1, if the file does not exist |
| 32 (hex 20) | read_line | An open file | The next line, without its newline | An error, code 7, when there are no more lines; code 2, if the file cannot be read |

One shape Chapter 19 fixed belongs with `move_file`'s entry: section 18.1 did not say what happens when the destination already exists, and section 19.1 fixed it, `move_file` replaces the destination. The move tool still refuses to ask for a replacement, because the tool checks the destination first; the routine itself replaces.

Chapter 19 named one more, for reading dense text, and the file routines grew from 12 to 13. The running count at the end of the chapter is 34.

| Number | Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- | --- |
| 33 (hex 21) | read_dense_line | An open dense file | The next line, as plain text, without its newline | An error, code 7, when there are no more lines; code 2, if the file cannot be read; code 8, if the file is not dense text |

Chapter 24 named one more, in the text routines, the complement of `text_of`. The running count at the end of the chapter is 35.

| Number | Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- | --- |
| 34 (hex 22) | number_of | Text holding whole digits | The number the text holds | An error, code 8, if the text holds anything but digits; code 5, if the number is too big to hold |

Chapter 28 named the last four, the network routines a program uses to reach the network service. The running count at the end of the chapter is 39, and the roster is complete: 28 from Chapter 16, plus 5 is 33, plus 1 is 34, plus 1 is 35, plus 4 is 39.

| Number | Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- | --- |
| 35 (hex 23) | open_talk | A name | The talk | An error, code 7, if the name is not in the service's record; an error naming the network, if there is no network to reach |
| 36 (hex 24) | send | A talk and text | The number of bytes sent | An error naming the network, if there is no network to reach |
| 37 (hex 25) | receive | A talk | The text that has arrived on that talk, or nothing arrived | Does not fail: nothing arrived is a normal answer |
| 38 (hex 26) | close_talk | A talk | Nothing; the talk is released | Does not fail |

The time service and the update service named no routines, the way Chapter 28 said: the time is read through `now` and set by asking the service, and updates are asked for by running the service, so neither needs a routine of its own. Chapters 20, 21, 22, 23, and 25 through 27, and 29 through 34, each stated in their own text that they add no routines, and the roster above is the whole library: 39 routines, each traced to the chapter that named it.

## The kernel calls

A running program reaches the kernel the same way it reaches the library: by a call, naming the call by number, in one byte, the way section 22.2 fixed it. The kernel's calls are the kernel's own set, separate from the library's numbers, because the library is code the call reaches inside the system and the kernel is the part that owns the hardware and hands out what programs share. A kernel call that fails hands back an error value, a code and a message, from the standard codes section 16.3 fixed, the way section 22.2 said, and the program deals with it the way section 7.3 gives every program: check it, pass it up, or give up cleanly.

The numbers are assigned in the order the chapters designed the calls, starting at 0, by the same rule as the routines. A number, once assigned, keeps its call in every later version, and a removed call's number is retired, never reused. Numbers 10 to 255, 246 numbers, are unassigned and reserved. Where a chapter designed a call without giving it a name of its own, the name below is fixed here and marked as fixed here; the call itself is the chapter's.

| Number | Call | Designed in | Takes | Hands back |
| --- | --- | --- | --- | --- |
| 0 (hex 00) | finish | Chapter 15 | Nothing | Nothing; the program is over. The finish routine runs its five steps, flush, close, free, release, report, and reports the exit to whoever started the program |
| 1 (hex 01) | start | Chapters 22 and 24 | A program file's name | The started program's number, or an error: code 1 if the file does not exist, code 8 if the file fails its checks, code 4 if the memory cannot be had, code 10 if the permission data forbids the start |
| 2 (hex 02) | time | Chapter 22 | Nothing | The count of seconds since the start of 1970, as an 8-byte number |
| 3 (hex 03) | stop | Chapter 24 | A running program's number | Nothing the chapters name beyond the outcome; the report goes to whoever started the program. An error, code 10, if the permission data forbids the stop |
| 4 (hex 04) | running | Chapter 24 | Nothing | A list, one entry per running program, each holding the program's number, its file's name, its state, running or waiting, and the memory it holds, the field Chapter 25 added |
| 5 (hex 05) | ask | Chapter 25 | A size, in bytes | The run handed out, named by its place in memory, the name the give back call takes. An error, code 4, if no free run is big enough |
| 6 (hex 06) | give_back | Chapter 25 | A run, named the way the ask handed it back | Nothing; the run is free, and joins with any free run beside it |
| 7 (hex 07) | next_key (name fixed here) | Chapter 27 | Nothing | One key: a character for a character key, the newline character for the Enter key. If no key has arrived, the asker waits, its core given up, and that waiting is not an error |
| 8 (hex 08) | set_clock (name fixed here) | Chapter 28 | The count of seconds to set, the world's count | Nothing; the kernel writes the count to the clock device itself, the one write the clock ever takes |
| 9 (hex 09) | secret_bytes (name fixed here) | Chapter 29 | A count of bytes needed | That many bytes, gathered fresh at the ask from the machine's own noise, the timings of the disk, the keys, and the clock, mixed so no one source decides any byte. The bytes are never logged |

Three of these calls carry guards from Chapter 29, and the guard is part of the call's shape. The start and stop calls are checked against the asking program's entry in the permission record before they run, and a refused start or stop hands back code 10. The set_clock call is the set time act: it is the time service that asks, at the user's ask, and the service's own entry must allow it. The secret_bytes call is unguarded, the way the count through `now` is unguarded: the bytes are gathered for the asker, at the ask, and belong to the asker.

Four pieces of kernel work are not in this table, because no chapter shaped them as a call a program makes, and this spec does not invent calls for them. The permission checks are the kernel's own work, done before a start, a stop, and a guarded ask, not a call. The crossings under the file routines and the talk routines, where the routine asks the kernel and the kernel asks the service that owns the job, were shaped by Chapters 26 and 28 as the routines' own asks, not as a separate numbered call. The drivers' calls from Chapter 27, the screen's write text, the disk's read blocks and write blocks, the network's send bytes and bytes arrived, and the rest, are calls the kernel makes to its own modules, the other direction from this table, and no chapter fixed numbers for them. And the turn check, the sharing of the two cores, is the kernel's own work on its own schedule of checks, which no program calls.

## Translation to native code

The rule is section 10.3's, stated as this spec's rule. Translation happens at load time, inside the loader, between placing the code section in memory and starting the program, on the target machine's processor, the Ryzen 3 3250U, whose own instructions are x86-64 instructions. The file on disk holds the middle form, because the middle form is the shared form the tools, the debugger, and the text view all read. The translation is done once per load, and its cost is part of the load. It is not a parse: it does no analysis, no tree building, and no error recovery. It reads one step and writes the native instructions that step becomes.

The translator works in two passes, because a jump target in the middle form is a position counted in middle form bytes, and the native code is not the same length as the middle form. The first pass walks the middle form and counts how many native bytes each step becomes, so it knows where each middle form position lands in the native code. The second pass writes the native instructions, and every jump target is looked up in the table the first pass built. Two passes, one table, and every jump lands where it should.

One exception stands, the one the book already made. The bootloader is translated once, at build time, and stored as native code, because the firmware runs native code and no translator is running yet when it starts, the way Chapter 23 designed it. The bootloader carries the loader, the translator, and the disk and screen drivers in that second stored form. Every other program in the system, the kernel included, is stored in middle form and translated at load, by the one translator.

The table below gives each operation in Appendix A, in Appendix A's number order, with the native instructions section 10.3 worked for it and the count of those instructions, where a chapter fixed them. Chapter 10 worked the hello loop's 14 operations, and only the operations that program uses appear in its table. For the operations no chapter worked, the entry says so plainly: the translation is not fixed by the book. This spec does not invent instructions or counts for them, and it prints no machine code bytes, because no chapter printed any: the chapters named the native instructions and counted them, and the byte encodings of those instructions are the processor's own, not numbers this book assigns.

| Operation | Code | Native instructions, where the chapters fixed them | Count |
| --- | --- | --- | --- |
| LOAD_CONST | 0 | push the number: push the constant onto the machine stack | 1 |
| LOAD | 1 | push from the slot: push the value in the named slot | 1 |
| STORE | 2 | pop into the slot: pop the top value into the named slot | 1 |
| PRINT | 3 | pop rdi; call print, the print routine | 2 |
| LOAD_STR | 4 | mov rax, the string's address; push rax | 2 |
| ADD | 5 | pop rax; add the register to the top of the pile, in place | 2 |
| END | 6 | call finish, the loader's finish routine, which is kernel call 0 above | 1 |
| LESS_THAN | 7 | pop rax; cmp the top of the pile against rax; setl al; movzx eax, al; mov the answer back onto the top of the pile | 5 |
| JUMP_IF_FALSE | 8 | pop rax; test rax, rax; jz to the target's native position | 3 |
| JUMP | 9 | jmp to the target's native position | 1 |
| CALL | 10 | Not fixed by the book | Not fixed |
| RETURN | 11 | Not fixed by the book | Not fixed |
| LOAD_DATA | 12 | Not fixed by the book | Not fixed |
| SUBTRACT | 13 | Not fixed by the book | Not fixed |
| MULTIPLY | 14 | Not fixed by the book | Not fixed |
| DIVIDE | 15 | Not fixed by the book | Not fixed |
| EQUAL | 16 | Not fixed by the book | Not fixed |

The comparison's five instructions each have their reason, and section 10.3 gave them: the machine's compare works on registers and the values live on the pile, so the second value is popped into a register, the first is compared against it, a byte is set to 1 if the first is less, that byte is widened to a full value, and the answer is written back onto the pile. The set instruction follows the type the operation carries, signed or unsigned: `setl` in the table is the signed form the hello loop used, and an unsigned comparison takes the unsigned form of the same instruction. DIVIDE's failure is fixed even though its translation is not: a number divided by zero gives the error of code 9, the way section 16.2 fixed it for the operation and for the remainder routine alike.

One saving belongs to the optimizer, and its proof shape is part of this spec because section 10.3 named it and section 12.1 proved it. When a comparison's result is used only by the jump that follows it, and by nothing else, which the optimizer proves by walking the middle form, the translation of the pair fuses: the widening and the write back are skipped, because the jump can read the compare's flags directly. LESS_THAN is 5 native instructions and JUMP_IF_FALSE is 3, 8 together. Fused, the pair is 4: pop the second value into a register, compare the first value with it, pop the first value into a spare register, because the pile still holds it after the compare, and jump if the first is not less than the second. The middle form keeps both steps; the saving is in the native code, and it is the optimizer's work, not the translator's. The translator's own work stays mechanical, and the count below is the plain translator's count, with the fused count beside it.

The worked example is the hello loop, the same program sections 4.4, 10.3, and 15.1 counted. Its 14 middle form operations become 23 native instructions under the plain translation: 1 plus 1 is 2, plus 1 is 3, plus 1 is 4, plus 5 is 9, plus 3 is 12, plus 2 is 14, plus 2 is 16, plus 1 is 17, plus 1 is 18, plus 2 is 20, plus 1 is 21, plus 1 is 22, plus 1 is 23. Nine of the fourteen steps translate one for one, the most any one step needs is five, and the average is just over one and a half native instructions per middle form step. With the fusion applied to its one comparison and jump pair, the same program becomes 19 native instructions: 23 minus the 4 the fusion saves, 8 for the pair unfused against 4 fused, is 19. Both counts are counts of instructions, not of bytes and not of time, and how long either takes is Chapter 31's measurement, not this spec's claim.

## What this spec does not fix

This appendix holds the middle form's layout, the library's routine numbers, the kernel call numbers, and the translation counts the chapters fixed, and nothing else. The operation numbers are Appendix A's. The file's fields and the type tag numbers are Appendix B's. A reader who needs a number that is not one of this spec's will find it in one of those two, not here.

This appendix assigns numbers the chapters left to it, so it adds nothing for the test suite or the library to hold. The suite stays at 217 tests, and the core library stays at 39 routines, now numbered, the closing counts Chapter 34 left as the first version's state.

The version rule, in one line: these numbers and this layout are fixed for version 1's files, a file names its version in its header, and a change to the form, the routine numbers, or the kernel call numbers is a new version made in the open, never a quiet edit to this spec.

---

The middle form is stated whole in one place: a list of steps in the code section, each step an operation code with its operands inline, positions counted from 0, lengths self describing, types travelling with the operations, and the pile discipline fixed in full. The library is reached by one call that names its routine by number, and the 39 routines stand numbered in the order the chapters named them, 28 from Chapter 16, 5 from Chapter 18, 1 from Chapter 19, 1 from Chapter 24, and 4 from Chapter 28. The kernel is reached by its own numbered calls, 10 of them, from the finish call a program's END becomes to the secret bytes call Chapter 29 designed. And the translation is fixed as far as the chapters fixed it: at load time, in two passes, with counts for the 10 operations the hello loop worked, the fused pair at 4, and the whole loop at 23 native instructions, 19 fused, and the 7 operations no chapter worked stated plainly as not fixed, rather than covered with invented numbers.
