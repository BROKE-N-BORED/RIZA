# Appendix B: Binary file format spec

This appendix is the binary file format stated completely, in one place, byte by byte. Chapter 9 designed the format and proved its arithmetic on one small file. This spec states the whole shape in the order the bytes appear, so a reader could write a loader from this appendix alone: read the header, find the three sections, walk the code, walk the data, and run the six checks, without hunting through the chapters for a width or a number the design left to the spec.

What the chapters left open, this appendix fixes, and the list is short. Section 9.2 left the exact numbers for the type byte that starts each value in the data section to this spec, and the type tag table below assigns them. Section 5.1 left the exact layout of fractional numbers to Chapter 16, which fixed the standard layout, called IEEE 754, in two sizes, 4 bytes and 8 bytes, and both sizes carry tags here. Section 5.2 stored a list's length with the list and fixed no width for that length; the width is fixed below, by the format's own pattern, and stated in the open beside the entry it belongs to. Nothing else in this appendix is new. Every other field, width, and count is Chapter 9's, Chapter 4's, or Chapter 5's, restated in spec order.

The version rule governs this spec the way it governs the dictionary in Appendix A. The header's version byte names the dictionary the file was written against, and it is also the file format version, because the dictionary and the format move together: a change to either one is a new version, made in the open, with old files still readable. A loader that meets a version it does not know does not guess. It says so, in plain words, and stops. Every number in this appendix is version 1's number.

## The file as a whole

A Riza file is four runs in a row, in a fixed order, with no gaps: the header, the code section, the data section, and the dictionary section. Each section starts where the one before it ends. The header states where each section starts and how many bytes it holds, so a reader finds every section from the header alone, and the stated positions can be checked, which is why the header states them instead of leaving the loader to work them out.

| Part | Where it starts | How its size is known |
| --- | --- | --- |
| Header | Position 0 | Always 33 bytes |
| Code section | The header's code offset | The header's code size |
| Data section | The header's data offset | The header's data size, which may be 0 |
| Dictionary section | The header's dictionary offset | The header's dictionary size |

The file's total size is 33 plus the code size plus the data size plus the dictionary size, and the third check below holds every file to exactly that sum. A section can be empty: a program with no static values has a data section of 0 bytes, and its offset is the position where the section would start, right after the code section. An empty section costs nothing and the loader reads nothing for it.

## The header

The header is the first 33 bytes of every file, always, big or small. Positions are counted in bytes from the start of the file, starting at 0.

| Position | Field | Bytes | What it holds |
| --- | --- | --- | --- |
| 0 | Magic | 4 | The letters R I Z A, the file's name tag, stored as the bytes `52 49 5A 41` |
| 4 | Version | 1 | The dictionary version the file was written against, which is also the file format version |
| 5 | Entry point | 4 | The position in the code section where the program starts, counted from the start of the code at 0 |
| 9 | Code offset | 4 | Where the code section starts in the file |
| 13 | Code size | 4 | How many bytes of code |
| 17 | Data offset | 4 | Where the data section starts in the file |
| 21 | Data size | 4 | How many bytes of data |
| 25 | Dictionary offset | 4 | Where the dictionary section starts in the file |
| 29 | Dictionary size | 4 | How many bytes of dictionary |

Check the total: 4 plus 1 is 5, plus 4 is 9, plus 4 is 13, plus 4 is 17, plus 4 is 21, plus 4 is 25, plus 4 is 29, plus 4 is 33. Chapter 9 fixed every width in this table, and this spec changes none of them. The positions and sizes are four bytes each because a file can be longer than 65,535 bytes, and four bytes count to about 4.3 billion, from the table in section 5.1. The entry point is four bytes for the same reason: a code section can be longer than 65,535 bytes. The entry point names a position inside the code section, not a position in the file; the loader adds the code offset to find the matching place in the file. For a program with no functions, the entry point is 0, the first operation in the code.

One fact about every multi-byte field in this format is fixed here, because no chapter fixed it and a loader cannot be written without it: a number wider than one byte is stored with its lowest byte first. A 4 byte field holding 33 is stored as `21 00 00 00`, the lowest byte, then the rest in rising order. The reason is the machine the book is built for: the processor in section 3.1's machine keeps its own numbers in that order, and the native code Appendix C describes reads them that way, so the file, the loader, and the machine all meet a number in the same order, and no step spends work turning a number around. The rule covers every multi-byte field in this spec: the header's positions and sizes, the mapping's starts, the comments' positions, and the whole and fractional numbers in the data section. It is stated in the open as a decision of this spec, version 1's rule like every other number in it.

## The code section

The code section holds the program itself: the operations of the shared middle form, in order, one after another, each operation's operands inline right after its code, the way sections 4.2 and 10.2 laid them out. The operation numbers and each operation's length are Appendix A's; this spec fixes only how the section sits in the file. The section's end is known from the header's code size, not from any marker inside the code. Positions inside the section are counted in bytes from the start of the section, starting at 0, and that is the one way of naming a place in the code that every part of the system uses: jump targets are counted that way, the entry point is counted that way, and the function mapping and the comments point that way.

The loader reads the section with the dictionary the header's version byte names. Its walk runs from the start of the section to the end, one operation at a time: each code says its own length, by the tiers in section 4.2 and the lengths in Appendix A, so the walk always knows where the next operation starts. That walk is the fifth check below, and it is also how the translator and the debugger read the code, because the form is self describing.

## The data section

The data section holds the static values: the constants, the numbers and text that never change, and the lists and records that exist from the start of the program, loaded once into static memory with the program, the way section 6.3 described. The values sit end to end in one run. Each value starts with one byte saying what kind of value it is, its type tag, followed by the value itself, in as many bytes as its kind takes. A value is addressed by its position in the section, counted in bytes from the section's start at 0, which is the position a LOAD_DATA operation carries, from Appendix A. The loader's walk reads a type tag, then reads the value in the bytes its kind takes, then reads the next tag, until the section's size from the header is used up. That walk is the sixth check below, and it only works because the tag alone says the kind and the size, the way section 10.2 required.

### The type tags

The tags follow Chapter 5's order of presentation, kinds in the order the chapter names them and sizes small to large inside a kind, one byte each, starting at 0, and a tag once assigned is never reused for a different kind.

| Type | Tag | Size of the value | Layout |
| --- | --- | --- | --- |
| Whole number, unsigned, 1 byte | 0 (hex 00) | 1 byte | The number itself, 0 to 255 |
| Whole number, unsigned, 2 bytes | 1 (hex 01) | 2 bytes | The number itself, 0 to 65,535 |
| Whole number, unsigned, 4 bytes | 2 (hex 02) | 4 bytes | The number itself, 0 to 4,294,967,295 |
| Whole number, unsigned, 8 bytes | 3 (hex 03) | 8 bytes | The number itself, 0 to 18,446,744,073,709,551,615 |
| Whole number, signed, 1 byte | 4 (hex 04) | 1 byte | The number itself, -128 to 127, in two's complement, the shape section 5.1 described |
| Whole number, signed, 2 bytes | 5 (hex 05) | 2 bytes | The number itself, -32,768 to 32,767 |
| Whole number, signed, 4 bytes | 6 (hex 06) | 4 bytes | The number itself, -2,147,483,648 to 2,147,483,647 |
| Whole number, signed, 8 bytes | 7 (hex 07) | 8 bytes | The number itself, -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807 |
| Fractional number, 4 bytes | 8 (hex 08) | 4 bytes | The IEEE 754 layout Chapter 16 fixed, holding about 7 digits exactly |
| Fractional number, 8 bytes | 9 (hex 09) | 8 bytes | The IEEE 754 layout Chapter 16 fixed, holding about 15 digits exactly |
| Text | 10 (hex 0A) | 1 byte | The name table number of the string; the characters themselves live in the name table |
| True or false | 11 (hex 0B) | 1 byte | 0 for false, 1 for true; any other byte is a broken file |
| List | 12 (hex 0C) | Counted below | The item kind's type tag, then the length in 4 bytes, then the items end to end |
| Record | 13 (hex 0D) | The sum of its fields' sizes | The fields end to end, in the definition's fixed order, each in its own kind's layout |
| File | 14 (hex 0E) | The file's own size on the disk | A run of bytes with a name, held on the disk, the way section 5.2 defined it |

The whole numbers take eight tags because the size is part of the type, the way section 5.1 fixed it and section 5.4 defended it: a 1 byte number and a 4 byte number are different kinds, no conversion between them happens in silence, and the tag alone must tell the loader how many bytes follow. The fractional kind takes two tags for the same reason, one per size Chapter 16 fixed.

A text value takes one byte because section 5.1 fixed the storage rule: strings live in the name table, stored once, and cost one byte per use after that. A static text value in the data section is a use, so it holds the string's name table number, and the characters sit in the table, not in the data section. Wider characters change nothing here: they take more bytes in the table, where the entry's length counts bytes, and the number that names the entry stays one byte.

A list value is laid out as its tag, then one byte holding the type tag of the kind every item shares, then the length, then the items. Two widths in that layout are fixed here, in the open. The item kind stands in the value as its own type tag byte, because the format's rule is that a value states its kind where it sits, and a list whose items' kind was stated nowhere in the file could not be walked or checked from the file alone. The length takes four bytes, by the format's own pattern: Chapter 9 uses four bytes for every position or size that can pass 255, and a list's length is a count that can pass 255 with no cap stated anywhere in the design, unlike the name table's count, which the design caps at 255 and so states in one byte. The items themselves carry no tags of their own. They sit bare, one after another, each in its kind's layout, because section 5.2's arithmetic only works that way: item 5 in a list of 4 byte numbers starts at byte 20, 5 times 4, with no room for a tag byte per item. A list's total size is its tag, the item kind byte, and the 4 length bytes, plus the length times one item's size, and an item's size is known from its kind by the same rows of this table.

A record value is its tag followed by the fields, end to end, in the fixed order of the record's definition, each field in its own kind's layout, with no tags between the fields. Section 5.2's worked example fixes that shape: the contact record is 4 bytes in total, 1 for the name's table number, 1 for the age, and 2 for the score, and 1 plus 1 plus 2 leaves no room for tag bytes. The record's size is the sum of its fields' sizes, fixed by the definition the compiler checked the program against, the way Chapter 11 enforces the types at build time.

The file kind carries a tag so that every kind Chapter 5 designed has a number, and its row states its limit plainly: a file is a run of bytes with a name on the disk, and no chapter stores a file value inside a data section. Chapter 9's own list of what the data section holds is numbers, text, lists, and records. The tag completes the set; it does not add a stored form the chapters never designed.

Tags 15 to 255 are unassigned in version 1 and reserved. They are where a later version puts a kind that has earned a tag, without renumbering anything already assigned. Reserved does not mean free for private use: a type tag this version does not define is a value the sixth check refuses, so an unassigned tag in a version 1 file is a broken file, not an extension.

## The dictionary section

The dictionary section holds three tables in a row, in this order: the name table, the function mapping, and the comments. Everything the program refers to by number lives here.

The name table holds the program's names and strings, stored once and called by number everywhere else. It starts with one byte saying how many entries it holds. Then each entry is a length byte followed by the characters, spelled out once, in the shape section 4.3 defined. Entries are numbered from 0, in the order they stand. The length counts bytes, so a wider character takes more bytes inside its entry and the shape does not change. The count is one byte, so a file holds at most 255 names. That is enough for the programs in this book, and Chapter 9 stated the limit in the open: if a future version needs a bigger count, the format changes as a new version, with old files still readable. The version byte exists for exactly that.

The function mapping says where each function's code starts. It starts with one byte saying how many functions the program has. Then each function gets one entry of 5 bytes:

| Field | Bytes | What it holds |
| --- | --- | --- |
| Name | 1 | The name table number of the function's name |
| Start | 4 | The position in the code section where the function's code starts |

The entries stand in function number order, so the entry's place in the table is the function number itself, counted from 0, and a CALL that names a function by number points straight at its entry. The name is there for the text view: the generator reads the mapping, finds the function's name, and writes the function with its name, the way Chapter 8 describes. The start position takes four bytes because it is a position in the code section, and code positions take four bytes in this format wherever they can pass 255, the same pattern as the entry point. A program with no functions has a count of 0, and its mapping is 1 byte, the count alone.

The comments table holds the program's notes, so the notes survive the round trip into the text view, the way section 8.2 required. It starts with one byte saying how many notes the program has. Then each note gets one entry:

| Field | Bytes | What it holds |
| --- | --- | --- |
| Position | 4 | The position in the code section the note describes |
| Length | 1 | How many bytes of note text follow |
| Text | As many as the length says | The note's characters |

The position takes four bytes because it is a code position, by the same pattern as the mapping's start. The note text is a length byte followed by the characters, the same shape as a name table entry. A program with no notes has a count of 0, and its table is 1 byte, the count alone. The notes are for the human view, not for the run, and the loader leaves them on the disk when it runs the program, the way section 9.3 described.

## The six checks

A loader written from this spec runs the six checks from section 9.4 in this order, cheapest first, each one depending on the ones before it. A file that fails any check is not run, nothing is half run, and the failure is reported as a clean error in plain words, the way section 7.3 describes errors.

1. **Magic.** Read the first 4 bytes and compare them with R I Z A. Refuses any file whose first four bytes are anything else: not a Riza file, or a corrupted one.
2. **Version.** Read the version byte at position 4 and ask whether this loader knows that dictionary. Refuses a version it does not know, naming the version the file needs, because the rest of the file's shape depends on the version.
3. **Sizes.** Add 33 to the code size, the data size, and the dictionary size from the header, and compare the sum with the file's actual size. Refuses a file where the two differ: a bigger sum means the file is truncated, a smaller sum means the file holds bytes the header does not account for.
4. **Bounds.** Check each section's offset plus its size against the file's size, and check that no two sections overlap. Refuses a section that runs past the end of the file and a file where two sections claim the same bytes. An empty section, size 0, sits where it would start and overlaps nothing, so it passes.
5. **References.** Walk the code section operation by operation, from its start to its end, and check every number that points at something. A name table number must be less than the name table's count. A jump target must be inside the code section and land on the start of an operation, not in the middle of one. A function number in a CALL must have an entry in the function mapping. The entry point must be inside the code section and on the start of an operation. A comment's code position must be inside the code section. Refuses the first reference that points outside what it names.
6. **Value shapes.** Walk the data section value by value, reading each type tag and checking that the value after it fits the kind the tag names, by the layouts in the tag table above. A true or false value must hold 0 or 1. A tag this version does not define is refused. The same check covers the code: an operation code the file's dictionary version does not define is refused, because the machine would not know what to do with it.

## One complete file: the hello loop

The hello loop from section 4.4, the loop that prints `hello` three times, as a complete file, every byte accounted for. Chapter 9 counted this file at 72 bytes; this section lays the same file out in full, in spec order, so the spec can be checked against the book's own example end to end.

The header, filled in for this file:

| Field | Value | Bytes |
| --- | --- | --- |
| Magic | R I Z A | 4 |
| Version | 1 | 1 |
| Entry point | 0, the first operation | 4 |
| Code offset | 33, right after the header | 4 |
| Code size | 24 | 4 |
| Data offset | 57, right after the code | 4 |
| Data size | 0, no static values | 4 |
| Dictionary offset | 57, right after the data | 4 |
| Dictionary size | 15 | 4 |
| Total | | 33 |

The code section is the 24 bytes at file positions 33 to 56, the 14 operations from section 4.4 in the same positions, read with Appendix A's dictionary, which is the version the header names:

| Position | Bytes | Meaning |
| --- | --- | --- |
| 0 | `00 00` | LOAD_CONST 0: put the number 0 on the pile |
| 2 | `02 00` | STORE name 0: store the top value as `count` |
| 4 | `01 00` | LOAD name 0: put `count` on the pile |
| 6 | `00 03` | LOAD_CONST 3: put the number 3 on the pile |
| 8 | `07` | LESS_THAN: put back true if the first value is less than the second |
| 9 | `08 17` | JUMP_IF_FALSE to position 23: if the top value is false, jump to END |
| 11 | `04 01` | LOAD_STR entry 1: put the string `hello` on the pile |
| 13 | `03` | PRINT: take the top value off and print it |
| 14 | `01 00` | LOAD name 0: put `count` on the pile |
| 16 | `00 01` | LOAD_CONST 1: put the number 1 on the pile |
| 18 | `05` | ADD: take two values off, put their sum back |
| 19 | `02 00` | STORE name 0: store the top value as `count` |
| 21 | `09 04` | JUMP to position 4: go back to the loop test |
| 23 | `06` | END |

The data section is empty: 0 bytes at position 57, where it would start.

The dictionary section is the 15 bytes at file positions 57 to 71:

| Bytes | Meaning |
| --- | --- |
| `02` | Name table: two entries |
| `05 63 6F 75 6E 74` | Entry 0: length 5, then `count` |
| `05 68 65 6C 6C 6F` | Entry 1: length 5, then `hello` |
| `00` | Function mapping: count 0, no functions |
| `00` | Comments: count 0, no notes |

Now the totals. The name table is 1 byte of count plus 6 bytes for `count` plus 6 bytes for `hello`, which is 13. The function mapping is 1 byte and the comments table is 1 byte, so the dictionary section is 13 plus 1 plus 1, which is 15, matching the header's dictionary size. The whole file is 33 plus 24 plus 0 plus 15, which is 72 bytes, positions 0 to 71, matching Chapter 9's count exactly. Chapter 12 counted a complete file in the same shape, 33 bytes of header and a dictionary section of a name table plus a function mapping plus a comments count; its mapping was 6 bytes because its program has one function, 1 count byte plus one 5 byte entry, where this file's mapping is the count alone because the loop has none. Same shape, both counts check.

## What this spec does not fix

This appendix holds the file's fields and the type tags, and nothing else. The operation numbers are Appendix A's. The middle form's layout in full, the core library's routine numbers, the kernel call numbers, and the native machine code bytes are Appendix C's, the way Appendix A pointed and Chapter 34 listed. A reader who needs a number that is not a file field or a type tag will find it in one of those two, not here.

This appendix assigns numbers the chapters left to it, so it adds nothing for the test suite or the library to hold. The suite stays at 217 tests, and the core library stays at 39 routines, the closing counts Chapter 34 left as the first version's state.

The version rule, in one line: these fields and tags are fixed for version 1's files, a file names its version in its header, and a change to the format or the tags is a new version made in the open, never a quiet edit to this spec.

---

The format is complete in one place: a 33 byte header naming the magic, the version, the entry point, and the offset and size of each of the three sections; a code section of operations read with Appendix A's dictionary; a data section of values each starting with its type tag, the fifteen tags assigned here in Chapter 5's order, from unsigned whole numbers at 0 through the file kind at 14, with tags 15 to 255 reserved; and a dictionary section holding the name table, the 5 byte per function mapping, and the comments. The six checks stand in Chapter 9's order as the spec's conformance list, and the hello loop's file is laid out in full at 72 bytes, agreeing with Chapter 9's count and Chapter 12's shape. A reader can write the loader from this appendix, and check the writing against the book's own example, byte by byte.
