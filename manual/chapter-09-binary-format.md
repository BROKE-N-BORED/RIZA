# Chapter 9: The binary format

Part 2 designed the language: short codes for common work, values of stated sizes, memory freed by ownership, functions called by number, errors as values, and a readable text form that round trips exactly. But a language needs a home on disk. A program has to live somewhere between the time it is written and the time it runs, and that somewhere is a file. This chapter defines the file.

Part 3 begins here, with the binary format and the compiler. The binary format is the shape of a program as it sits on the disk: what comes first, what follows, how big each part is, and how the loader finds its way through it. Every piece of Part 2 lives in this file: the code from Chapter 4, the values from Chapter 5, the static memory from Chapter 6, the functions from Chapter 7, and the text and notes from Chapter 8. The file is where the language stops being a design and starts being something the machine can hold.

There are four sections. The first names the file header and the version number. The second names the three sections that follow: code, data, and dictionary. The third says how loading works: a few long reads, no re-parse, straight to run. The fourth says how the loader checks a file is valid before running it, and what happens when it is not.

One warning, stated up front the way Chapters 4 through 8 stated their own. This chapter designs the file format. It does not build the compiler that writes it. That is Chapter 11. The full formal spec, every field and every byte in the order it appears, is Appendix B. This chapter designs the shape and proves the arithmetic on one complete small file. The chapter also claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own example, counted by hand. The measured proof comes in Chapter 31, on the real PC, with the comparison against today's stack in Chapter 32.

## 9.1 File header and version

The header is the first thing in the file, and the loader reads it first. It is small, fixed in size, and it tells the loader everything it needs to know before it reads the rest. The header is 33 bytes long, always, for every file, big or small. That fixed size is the point: the loader's first read is always 33 bytes, and it never has to guess how much to read.

Here is the header, field by field, with the byte count of each:

| Field | Bytes | What it holds |
| --- | --- | --- |
| Magic | 4 | The letters R I Z A, the file's name tag |
| Version | 1 | The dictionary version the file was written against |
| Entry point | 4 | The position in the code where the program starts |
| Code offset | 4 | Where the code section starts in the file |
| Code size | 4 | How many bytes of code |
| Data offset | 4 | Where the data section starts in the file |
| Data size | 4 | How many bytes of data |
| Dictionary offset | 4 | Where the dictionary section starts in the file |
| Dictionary size | 4 | How many bytes of dictionary |
| Total | 33 | |

Check the total: 4 plus 1 is 5, plus 4 is 9, plus 4 is 13, plus 4 is 17, plus 4 is 21, plus 4 is 25, plus 4 is 29, plus 4 is 33. Every header is 33 bytes, and the loader knows that before it opens the file.

Take the fields one at a time.

The magic is a name tag. Magic bytes are a short fixed marker at the start of a file that says what kind of file it is, the way a label on a box says what is inside. The marker for a Riza program is the four letters R I Z A, stored as four bytes: `52 49 5A 41` in hexadecimal, which is R is 82, I is 73, Z is 90, A is 65 in ordinary counting, from the ASCII table Chapter 5 introduced. The loader reads the first four bytes of any file it is asked to run, and if they are not R I Z A, it knows at once that this is not a Riza program, or that the file is corrupted, and it stops.

The version is one byte, and it says which dictionary the file was written against. Section 4.1 set this rule: the dictionary is versioned, and the version says which dictionary the file was written against. The first dictionary is version 1. A new dictionary version is a deliberate act, taken when the counts have moved enough to matter, and every program written against the old version still runs, because the old dictionary is kept. The loader keeps the dictionaries it knows, and when it reads a file, the version byte tells it which one to use. A loader that meets a version it does not know does not guess. It says so, in plain words, and stops, the way section 9.4 describes. The version byte is also the file format version, because the dictionary and the format move together: a change to either one is a new version, made in the open, with old files still readable.

The entry point is four bytes, and it says where the program starts. It is a position in the code section, counted in bytes from the start of the code, starting at 0, the same way jump targets are counted in sections 4.4 and 7.2. The loader reads the entry point, adds the code section's offset to find the matching position in the file, and starts running there. For a program with no functions, the entry point is 0, the first operation in the code. For a program with functions, the entry point says where the main program starts, and the functions sit elsewhere in the code, reached by calls. The entry point is four bytes because a code section can be longer than 65,535 bytes, and four bytes count to about 4.3 billion, from the table in section 5.1.

The section offsets and sizes are the rest of the header, and they are the reason the header exists. Each section gets two numbers: where it starts in the file, and how many bytes it holds. The offsets are counted in bytes from the start of the file, starting at 0, so the first byte of the file is position 0 and the first byte after the header is position 33. The sizes are counted in bytes. Both are four byte numbers, for the same reason: a file can be longer than 65,535 bytes.

Say plainly why the positions are stated, not assumed. The sections follow each other in a fixed order, so the loader could work out where each one starts by adding up the sizes. But a position the loader works out is a position it cannot check, and a position the file states is a position it can. Section 9.4 uses the stated positions to check that every section sits inside the file and that no two sections overlap. Stating the positions costs 12 bytes, four per section, and buys a check that an assumed position could never give. That is the same trade the book makes everywhere: a few bytes, stated in the open, for a check the reader can follow.

One small read, 33 bytes, tells the loader the file's shape before it reads the rest, so it can plan its memory and its reads, and check the file before running anything. The header is the file introducing itself before the loader has to trust it with anything.

## 9.2 Code section, data section, dictionary section

After the header come three sections, in a fixed order: code, data, dictionary. Each one is a long run of bytes, one stretch after another, with no gaps and no jumps between them. Section 3.1 said the spinning drive reads fastest in long sequential runs, and that files in Riza are laid out to be read in long runs. This is that layout. The file is four runs in a row: the header, the code, the data, and the dictionary. The loader reads forward from the start of the file to the end of it, and the disk head moves forward with it, never jumping back.

Take the sections one at a time.

The code section holds the program itself: the operations from the dictionary, in order, one after another, with their operands. It is the middle form from section 2.3, written in the dictionary's short codes from Chapter 4. Positions inside the code section are counted in bytes from the start of the section, starting at 0, the way section 4.4 counted them. A jump target is a position in the code section. The entry point from the header is a position in the code section. The function mapping points at positions in the code section. There is one way to name a place in the code, and every part of the system uses it. Chapter 10 defines the middle form itself: its operations, its layout, and the step by step path from middle form to native machine code. This chapter only needs the shape: the code section is the operations, in order, in one run.

The data section holds the static values: the constants, the numbers and text that never change, and the lists and records that exist from the start of the program. Section 6.3 said static memory holds the program's own data, the data that lives for the whole run, and that it is loaded once, with the program, in the data section and the dictionary section of the binary file. The data section is the first half of that. Each value in the data section starts with one byte saying what kind of value it is, a type byte, followed by the value itself, in as many bytes as its type takes. The loader reads the type byte, reads the value, and checks the value fits the kind, the way section 9.4 describes. A value that says it is a true or false value must hold 0 or 1, and a byte holding 7 where a true or false is expected is a broken file, the way section 5.1 promised. The exact numbers used for each kind are part of the formal spec in Appendix B. This chapter only needs the shape: kind first, then value, one after another, in one run.

A section can be empty, and the header says so plainly. A program with no static values has a data section of 0 bytes, and its offset is the position where the section would start, right after the code section. An empty section costs nothing, and the loader reads nothing for it. Zero means zero, and the file is smaller for it.

The dictionary section holds three tables, in a row, and it is the busiest part of the file: everything the program refers to by number, the names, the functions, and the notes.

The first table is the name table, in the shape section 4.3 defined: one byte saying how many entries it holds, then each entry as a length byte followed by the characters, spelled out once. It holds the program's names and strings, stored once and called by number everywhere else. Nothing about the name table changes in this chapter. It moves into the file in exactly the shape Chapter 4 gave it.

The second table is the function mapping, and it answers the question section 7.1 left to this chapter: the loader knows where each function's code starts, and Chapter 9 defines how that mapping is stored. The mapping starts with one byte saying how many functions the program has. Then each function gets one entry: the name table number of the function's name, one byte, and the position in the code section where the function's code starts, four bytes. The entry is 5 bytes in total. The entries are in function number order, so a CALL that names a function by number points straight at its entry. The name table number is there for the text view: the generator reads the mapping, finds the function's name, and writes the function with its name, the way Chapter 8 describes. A program with no functions has a count of 0 and a mapping of 1 byte, the count alone.

The third table is the comments, and it answers the question section 8.2 left to this chapter: the notes survive the round trip, because they are stored in the file with the program, and the exact place they are stored is this chapter's work. The comments table starts with one byte saying how many notes the program has. Then each note gets one entry: the position in the code section the note describes, four bytes, and the note text itself, as a length byte followed by the characters, in the same shape as a name table entry. The note is attached to the position in the code it describes, so the generator puts it back in the text where the programmer wrote it. A program with no notes has a count of 0 and a table of 1 byte, the count alone. The notes are for the human view, not for the run, and section 9.3 says the loader leaves them on disk when it runs the program.

One honest note about a limit, stated here because the book states its limits in the open. The name table's count is one byte, the way section 4.3 defined it, so a file holds up to 255 names. That is enough for the programs in this book, and the worked example below uses 2. But a large program could hold more than 255 names, and this format, as designed, would not hold it. The answer is the version number in the header: if a future version needs a bigger count, the format changes in the open, as a new version, with old files still readable. The version number exists for exactly this. The limit is stated here so no reader is surprised by it later.

Now work one complete small file all the way through, the way sections 4.4 and 7.2 worked their examples, with every byte counted. The program is the hello loop from section 4.4, the loop that prints `hello` three times, now as a full file with a header. Section 4.4 counted the dense form at 37 bytes, 24 of code and 13 of name table, and said the full file adds a header of a few fixed bytes on top. This section counts the whole file, header and all.

Start with the header, filled in for this file:

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

The offsets check out by adding. The header is 33 bytes, so the code starts at position 33. The code is 24 bytes, so the code ends at 33 plus 24, which is 57. The data section starts at 57 and holds 0 bytes, so it ends at 57 too. The dictionary starts at 57 and holds 15 bytes, so it ends at 57 plus 15, which is 72. The file is 72 bytes long, positions 0 to 71.

Now the sections, with the dictionary section broken down into its three tables:

| Part | Bytes |
| --- | --- |
| Header | 33 |
| Code section, the loop from section 4.4 | 24 |
| Data section, empty | 0 |
| Dictionary section, name table | 13 |
| Dictionary section, function mapping, count 0 | 1 |
| Dictionary section, comments, count 0 | 1 |
| Total | 72 |

Check the dictionary section: the name table is 13 bytes, from section 4.4, one count byte plus 6 bytes for `count` plus 6 bytes for `hello`. The function mapping is 1 byte, the count saying 0, because the loop has no functions. The comments table is 1 byte, the count saying 0, because the loop has no notes. 13 plus 1 plus 1 is 15. Check the file total: 33 plus 24 is 57, plus 0 is 57, plus 15 is 72. The whole file is 72 bytes, and every byte is accounted for.

Read the result honestly. The text form of this program is 68 bytes, from section 4.4. The complete file is 72 bytes. The file is 4 bytes bigger than the text. Section 4.4 warned that this would happen: on a program this small, the header matters. The header is a fixed cost of 33 bytes, paid once per file, and on a program this small it outweighs the saving the dense form makes. Rule 7 of section 1.5 says a loss is reported as a loss, and this is a loss, on this program, by 4 bytes.

The loss is real, and it is also small and fixed, which is why the design still stands. The header costs 33 bytes whether the program is 24 bytes of code or 24,000. On a program ten times longer than this one, the same 33 bytes is a small fraction of the file, and the dense form's saving, which grows with every use of every name and every operation, dwarfs it. And the header buys things the text form cannot: the loader's plan, the validity checks in section 9.4, and the version that lets the format grow. The honest statement is the one section 4.4 made: the dense form wins by more as the program grows, because the dictionary is paid once per file and the uses are paid per use. On a program this small, the fixed costs win. On a real program, the dense form wins.

## 9.3 Loading: no re-parse, straight to run

Loading is the loader's job, and the loader's job is small on purpose. It reads the file, checks it, places its parts in memory, and starts the program. That is all it does. There is no parse step, no tokenizing, no tree building, no bytecode cache to check, because the parse already happened once, when the program was written, and the binary form is the result of that parse, stored on disk. Section 2.2 made this promise: loading is reading, not re-translating. This section keeps it.

The loader reads the header first. One read, 33 bytes, the fixed size from section 9.1. From those 33 bytes, the loader knows the version, the entry point, and where each section sits and how big it is. It knows the file's shape before it reads the rest of it, so it can ask for exactly the memory it needs and plan exactly the reads it will make.

Then the loader reads each section in order: code, data, dictionary. One read per section that has anything in it, in file order, so the disk reads forward from the header through the sections without jumping back. For the hello loop file, that is three reads in total: the header, the code section, and the dictionary section. The data section is empty, so the loader reads nothing for it. Three reads, each a long run, each moving forward.

Compare that with what section 1.4 counted for today's startup. A Python program starts by reading its 8 MB interpreter binary, then its startup files, then the parts of its 55 MB standard library that startup requires, each a separate file in a separate place on the disk. An interpreter startup is a sequence of small reads scattered across the disk, and each jump between files costs a seek, and seeks on a spinning drive take milliseconds each. Riza's load is the opposite shape: one file, a few long reads, in one forward motion. The saving is in the shape of the reading, not in the speed of the disk, and the shape is set by the file format, which is why this chapter exists.

Placing the parts in memory is as simple as reading them. The code section goes where the program runs from. The data section goes into static memory, the place section 6.3 named for values that live for the whole run. The name table and the function mapping are held for the run, because the program looks names up by number and calls functions by number while it runs. The comments are the one part the loader does not keep in memory when running. They are for the text view, not for the run, and they stay on disk until a tool that generates text asks for them. The loader does not pay for what the run does not use.

Then the program starts, at the entry point. The loader jumps to the position in the code the header named, and the first operation runs. There is no warm up, no first run that pays a parse the later runs avoid. The first run is the same as every run, because the work of understanding the program was done once, when it was written, and never repeated. The step from the middle form in the code section to the machine's own instructions is Chapter 10's work. This chapter only needs the shape: read, place, start.

The loader is small, because it does so little: read a header, read three sections, check the file, place the parts, start the program. A small loader is a fast loader, and a fast loader is what startup needs, because section 1.4 showed that startup is mostly the cost of loading things before the work begins. The loader has no parser, because the parser lives in the editor, the way section 2.2 said. It has no cache to check, because the file is the truth and there is nothing to go stale. It has no hidden work, because section 2.4 forbids it.

One honest note about what this section does not claim. It claims no timings. No number here says how many milliseconds a load takes, because no load has been measured on the target machine, and Chapter 3 forbids numbers without a method. The load counts are arithmetic: three reads for the hello loop, each a stated number of bytes. The measured timings belong to Chapter 31, on the real PC, and the comparison against today's stack belongs to Chapter 32. What this section shows is enough for the design: the load is a few long reads and no parse, and that shape is what makes a fast start possible.

## 9.4 Checking a file is valid before running it

A binary file can be checked quickly, before it runs, because its shape is fixed. Section 2.2 said this, and it is the property this section uses. The loader can verify the header, confirm the sections are the sizes they claim to be, and check that every reference points inside the file, all before executing a single instruction. A malformed file is caught at the door. A text file cannot be checked this way, because its meaning only emerges from parsing it, and parsing it is the expensive step the binary form skips. Validity checking is not an extra feature. It is a property the binary form gives for free.

The checks happen in order, cheapest first, and each one depends on the ones before it. A file that fails any check is not run. The order is the design, so it is stated in full.

First, the magic bytes. The loader reads the first four bytes of the file and compares them with R I Z A. If they do not match, the file is not a Riza program, or it is corrupted, and the loader stops. This check is first because it is cheapest: four bytes, one comparison, and it answers the most basic question, whether the file is even the right kind.

Second, the version. The loader reads the version byte and asks whether it knows that dictionary. If the version is one the loader knows, it goes on, using the dictionary that version names. If the version is newer than anything the loader knows, the loader stops and says so plainly: this file needs dictionary version 3, and this loader knows up to version 1. The loader does not guess what a newer version means, because guessing at a file format is how a program runs the wrong code. The version check is second because the rest of the file's shape depends on the version: a file written against a dictionary the loader does not know may use codes the loader cannot read.

Third, the sizes add up. The loader adds the header size, 33 bytes, to the code size, the data size, and the dictionary size from the header, and compares the sum with the file's actual size, which it asks the filesystem for. The two must match exactly. If the sum is bigger than the file, the file is truncated. If the sum is smaller, the file has extra bytes the header does not account for. Either way, the file is broken, and the loader stops. For the hello loop file, the sum is 33 plus 24 plus 0 plus 15, which is 72, and the file is 72 bytes. The check passes. This check is arithmetic on numbers the header already states, and it costs nothing but an addition.

Fourth, the sections sit inside the file and do not overlap. The loader checks each section's offset plus its size against the file's size: a section that runs past the end of the file is broken. It checks that no two sections overlap: a file where the code section and the data section claim the same bytes is broken. An empty section, with a size of 0, sits at the position where it would start and overlaps nothing, so it passes. This check is possible because the header states the positions, the way section 9.1 described. The stated positions are what make the check possible, and the check is what makes them worth their 12 bytes.

Fifth, every reference points inside the file. The loader walks the code, operation by operation, from the start of the code section to the end of it, and checks every number that points at something.

A name table number in the code must be less than the number of entries in the name table. A LOAD_STR that names entry 5 in a table of 2 entries is a broken file, because entry 5 does not exist. For the hello loop, the only valid numbers are 0 and 1, and the code uses exactly those.

A jump target must be inside the code section, less than the code size, and it must land on the start of an operation, not in the middle of one. A jump to position 40 in a code section 24 bytes long is broken. A jump to position 5, in the middle of a two byte operation that starts at position 4, is broken too, because the machine would start reading in the middle of an operation and misread everything after it. The loader knows where each operation starts because it walked the code forward, and the operation codes say their own length, the way section 4.2 designed them. The walk is the check, and the check is one forward pass.

A function number in a CALL must have an entry in the function mapping. A call to function 3 in a program with 2 functions is broken. The entry point must be inside the code section and on the start of an operation, for the same reason a jump target must be. A comment's code position must be inside the code section too, because a note attached to a place that does not exist is a note about nothing.

Sixth, the values are in their valid shape. The loader walks the data section, reads each type byte, and checks the value that follows it fits the kind the type byte names. A true or false value must hold 0 or 1, and a byte holding 7 where a true or false is expected is a broken file, the way section 5.1 promised. An operation code in the code section must be one the dictionary version defines: a code the dictionary does not know is broken, because the machine would not know what to do with it. These checks are the binary form paying for itself: a fixed shape can be checked, and a value outside its shape is caught instead of silently misread.

What happens when a check fails is as much a part of the design as the checks themselves. The loader reports the failure as a clean error in plain words, the way section 7.3 describes errors as values, and stops. Nothing is half run. No instruction executes, no memory is placed, no program starts. The error names what failed, in words the user can read: not a Riza file, unknown dictionary version, file truncated, section outside the file, jump outside the code, name table number past the end of the table. The user sees the error and knows what happened, and the programmer who made the file knows what to fix.

The checks are cheap, and that matters, because a check that costs more than the load would be a tax on every program start. The magic is one comparison. The version is one comparison. The sizes are one addition. The section bounds are arithmetic on numbers the header already states. The references are one forward walk of the code, which the loader does anyway. The value shapes are one forward walk of the data section, which the loader does anyway to place the values in static memory. Nothing is read twice, nothing is parsed, and nothing is hidden. The checks are part of the load, not an extra step bolted on after it.

The file is defined now: a 33 byte header that names the version, the entry point, and the three sections; a code section holding the program; a data section holding the static values; and a dictionary section holding the names, the functions, and the notes. The loader reads it in a few long reads, checks it at the door, and starts the program at the entry point, with no parse and nothing hidden. The next chapter defines the middle form that fills the code section: the shared form from section 2.3, its operations, its layout, and the step by step path from middle form to native machine code.

---

The binary format is the file the loader reads: a 33 byte header with the magic R I Z A, the dictionary version, the entry point, and the offset and size of each section; a code section holding the program's operations; a data section holding its static values; and a dictionary section holding its name table, its function mapping, and its notes. The hello loop as a complete file is 72 bytes, every byte counted, and the chapter says plainly that on a program this small the file is 4 bytes bigger than the text, because the header is a fixed cost that fades as programs grow. The loader reads the file in a few long reads, checks it before running it, magic, version, sizes, bounds, references, and value shapes, in that order, and reports any failure as a clean error in plain words, with nothing half run. Part 3 has begun. The file is defined. Chapter 10 defines the form that fills it.
