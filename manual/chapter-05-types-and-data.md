# Chapter 5: Types and data

Chapter 4 gave the language its short codes. This chapter says what the values are that those codes move around.

A type is a promise about what a value is and how big it is. A number is not just a number. It is a number of a stated size, and the size says how large it can be and how many bytes it costs. Text is not just text. It is characters stored a stated way, with a stated cost per character. True and false are not just ideas. They are values with a size, like everything else. The type says the size, and the size says the cost, and nothing in this chapter is left vague about either.

There are four sections. The first names the basic values: numbers, text, true and false, with every size shown as arithmetic the reader can check. The second builds the compound values from them: lists, records, and files. The third says how data sits in memory so the processor's cache can do its job. The fourth is the rule that gives the chapter its spine: what the language refuses to hide.

One warning, stated up front the way Chapter 4 stated its own. This chapter designs the types. It does not fix the final text syntax a programmer types. That is Chapter 8. It does not build the compiler that checks the types. That is Chapter 11. The sizes and shapes here are the design, and the exact names a programmer writes are Chapter 8's work. A design chapter that pretended the syntax was settled would be guessing, and Chapter 3 forbids guessing.

## 5.1 Numbers, text, true/false

Start with the smallest pieces, because everything in this chapter is built from them. A bit is one yes or no, from section 2.1. A byte is 8 bits. A byte holds 2 to the power of 8 values, because each bit doubles the count: 1 bit holds 2 values, 2 bits hold 4, 3 bits hold 8, and 8 bits hold 256. So a byte holds the numbers 0 to 255, and that is the whole range. There is no room for 256 in a byte, and the language does not pretend there is.

Bigger numbers take more bytes, and each extra byte multiplies the range. Here is the table, with the arithmetic shown so any reader can check it:

| Bytes | Values it holds | Range |
| --- | --- | --- |
| 1 | 2 to the power of 8 = 256 | 0 to 255 |
| 2 | 2 to the power of 16 = 65,536 | 0 to 65,535 |
| 4 | 2 to the power of 32 = 4,294,967,296 | 0 to 4,294,967,295 |
| 8 | 2 to the power of 64 = 18,446,744,073,709,551,616 | 0 to 18,446,744,073,709,551,615 |

Read the table from the top down. One byte counts to 255. Two bytes count to 65,535, which is the same number of values as the long tail of operations in section 4.2, for the same reason: two bytes hold 65,536 values, whatever they are used for. Four bytes count to about 4.3 billion. Eight bytes count to about 18.4 quintillion, a number so large that no program in this book will ever need to count that high, but the size is stated anyway, because the rule is that every size is stated.

Negative numbers take a sign, and the sign takes half the range. A byte that can hold a negative number splits its 256 values in two: half for the numbers below zero, half for zero and the numbers above it. A signed byte holds -128 to 127. That is 128 values below zero, from -128 to -1, and 128 values at or above zero, from 0 to 127. The usual way to do this is called two's complement, and the name matters less than the shape: half the range below zero, half at and above zero. The signed ranges follow the same doubling as the unsigned ones:

| Bytes | Signed range |
| --- | --- |
| 1 | -128 to 127 |
| 2 | -32,768 to 32,767 |
| 4 | -2,147,483,648 to 2,147,483,647, about -2.1 billion to +2.1 billion |
| 8 | about -9.2 quintillion to +9.2 quintillion |

Why sizes matter is worth one concrete example, because it ties the table to the machine on the desk. A 4-byte number can count every byte in 4 GB of memory, because 2 to the power of 32 is 4,294,967,296, and 4 GB is 4,294,967,296 bytes. The target machine has 8 GB, which is 8,589,934,592 bytes, more than a 4-byte number can count. So a number that names a position anywhere in this machine's memory needs 8 bytes, not 4. That is one reason the language's numbers come in sizes, and why the size is part of what a number is: the size says what the number can be used for, and a size too small for the job is a size that fails.

Small numbers ride inline, the way section 4.3 described. A number like 3 or 100 rides in the code itself, as a byte right after the operation that uses it, the way the worked example in section 4.4 showed. It does not need a table entry, because it is already short. A bigger number takes more bytes, and the language says so openly: the type carries its size, so a reader can see how many bytes a number takes by looking at its type. There is no number type with a secret size. If a number takes 8 bytes, the program says so, and the reader knows.

Not every number is whole. A measurement can be 3.5. A price can be 12.99. The language has a kind for numbers with a fractional part, and it follows the same rule: a fixed size, stated openly, like the rest. One honest limit belongs here, because it is easy to get wrong. A fraction stored in a fixed number of bytes cannot hold every fraction exactly. One tenth has no exact end in binary, the way one third has no exact end in decimal: 0.333 and on forever, never landing. A program that needs exact fractions must know that limit, and the language does not hide it. The exact layout of fractional numbers, and the limits of their exactness, are set down in Chapter 16, where the core library's math lives.

Text is characters, and a character is a byte for plain letters. The old standard table, called ASCII, gives each plain character a number: A is 65, a is 97, 0 is 48, a space is 32. A string is a run of characters, one after another, and its length in bytes is the number of characters it holds. Strings live in the name table from section 4.3: stored once, called by number, the same as a variable name. The string `hello` costs 5 bytes in the table, once, and one byte per use after that, no matter how many times the program prints it.

Wider characters exist, and the book says so plainly instead of pretending all text is English. Text in other languages needs more than one byte per character, because one byte holds only 256 values and the world's writing systems hold far more characters than that. The table stores bytes, and the length of an entry counts bytes, so a wider character takes more bytes and the table holds it the same way, with no change to the mechanism. The full text story, including how non-English text is stored and worked with, belongs to Chapter 16, where the core library's text handling lives.

True and false are the smallest values in the language. A true or false value carries one bit of meaning: yes or no. It is stored as a byte in practice, because the machine reads bytes, the same reason section 4.2 gave for byte sized operation codes. A byte holding 0 means false. A byte holding 1 means true. The other 254 values a byte can hold are not valid true or false values. A file that holds 7 where a true or false is expected is a broken file, and Chapter 9's validity check catches it at the door, before the program runs. That is the binary form paying for itself again: a fixed shape can be checked, and a value outside its shape is caught instead of silently misread.

The rule of this section is simple, and it runs through the rest of the chapter: every basic value has a stated size, and the size is the cost. A number's type says how many bytes it takes. A string's length says how many bytes it takes. A true or false value takes one byte. Nothing is free, nothing is secret, and the arithmetic above is checkable by any reader with a pencil. Section 5.4 defends that rule. The next two sections build on it.

## 5.2 Lists, records, files

Basic values are the bricks. This section is the shapes built from them: lists, records, and files. Each one has a layout, and the layout is part of the design, because the layout is what decides how fast the value is to use and how much room it takes.

A list is an ordered sequence of values of the same kind. A list of numbers. A list of names. A list of records, which this section defines below. The list is stored together in one run, one value after another, with no gaps. The list knows its own length, stored with it, so the program always knows how many items the list holds without counting them. Finding an item is counting forward from the start: the item's position times the size of one item. In a list of 4-byte numbers, item 5 starts at byte 20, because 5 times 4 is 20. Items are counted from 0, so item 0 is the first, item 1 is the second, and item 5 is the sixth. The arithmetic is the whole lookup. There is no search, no hunt, no table to consult. The position is computed, and the value is there.

A list grows, and growing is real work. A list lives in a run of memory of a fixed size. When the program adds an item and the run is full, a new, bigger run is made, the old contents are copied into it, and the old run is freed. That copy costs time and memory, and the cost grows with the size of the list. Section 5.4 says the copy must be visible, not hidden. Chapter 6 says where the new run comes from and when the old run is freed. This section only needs the shape: a list is one run, it knows its length, and growing it is a copy.

A record is a group of named fields of different kinds, bundled together. A person's name, age, and score, in one bundle. A file entry's name, size, and date. A window's position, size, and title. Each field has its own type and its own size, and the record lays the fields out one after another in memory, in a fixed order, so the compiler knows where each field sits. Work through one record by hand, a contact with three fields:

| Field | What it holds | Size | Position in the record |
| --- | --- | --- | --- |
| name | Table number for the name | 1 byte | 0 |
| age | Whole number, 0 to 255 | 1 byte | 1 |
| score | Whole number, 0 to 65,535 | 2 bytes | 2 |
| Total | | 4 bytes | |

The name is text, so it lives in the name table from section 4.3, and the record holds the 1-byte table number, not the name itself. The age fits in one byte, because nobody is older than 255. The score needs two bytes, because it can run past 255. The record is 4 bytes in total: 1 plus 1 plus 2. The age is always 1 byte from the start of the record. The score is always 2 bytes from the start. Finding a field is arithmetic, not a search: the compiler adds the field's position to the record's start, and the value is there. The positions never change, because the fields never move, and that fixedness is what makes a record fast to use.

The list rule and the record rule combine. A list of these 4-byte records is a run of 4-byte records, one after another. Record 3 starts at byte 12, because 3 times 4 is 12. The program walks the list by stepping 4 bytes at a time, and each step lands on the next record. Section 5.3 shows why that steady stepping matters for the cache. For now, the point is that the two shapes fit together: lists hold records, records hold values, and every position in the whole structure is arithmetic the reader can check.

A file is data on disk, treated as a value the program can read and write, in the same language, not through a separate tool. A file is a run of bytes with a name. Reading a file gives the program the bytes. Writing a file stores bytes. The program reads a file the way it reads a list: in order, from the start, or by jumping to a position. That in-order reading is what the spinning drive does fastest, from section 3.1, so a file laid out as one run is a file the drive can read at its best speed. A file can be passed around like any other value, opened, read, written, and closed. Chapter 18 builds the file tools on these values: list, copy, move, delete, search. Chapter 26 builds the filesystem the files sit on. Chapter 6 says where the bytes live while the program holds them. This section only needs the shape: a file is a run of bytes with a name, and the language treats it as a value, not as a foreign thing that needs a separate tool to touch.

No syntax has been invented in this section, and none is needed yet. The shapes are the design: a list is one run with a length, a record is fields at fixed positions, a file is a run of bytes with a name. Chapter 8 defines the text form a programmer writes to make these shapes, and proves the round trip is exact. The compiler in Chapter 11 turns that text into the dense form, with the layouts in this section, and tests every operation to prove it. The middle form in Chapter 10 carries these same types, so the tools see the same sizes the compiler sees, and no tool has to guess what a value costs.

## 5.3 How data sits in memory for cache

The processor is fast. Main memory is slow. That gap is the reason for the cache, and the cache is the reason this section exists.

A cache is a small, very fast memory built into the processor, much faster than the main 8 GB, that holds the data the program is using right now. When the processor needs a value, it looks in the cache first. If the value is there, the work goes on at once. If it is not, the processor waits while the value is fetched from main memory. That wait is the cost this section is about. The program cannot remove the wait. What it can do is lay its data out so the wait happens as seldom as possible, by keeping the data the program uses together in one place.

Here is the cache on the target machine, from AMD's own spec page for the Ryzen 3 3250U:

| Cache | Size | Shared or per core |
| --- | --- | --- |
| L1 | 192 KB total, 96 KB per core: 32 KB for data, 64 KB for instructions | Per core |
| L2 | 1 MB total, 512 KB per core | Per core |
| L3 | 4 MB | Shared by both cores |

Read the sizes against the main memory. The L3 cache is 4 MB out of 8 GB, one part in 2,048. The L1 data cache for one core is 32 KB out of 8 GB, one part in 262,144. The fast memory is tiny. That is the whole point of this section: the memory the processor can reach at once is a small fraction of the memory the machine holds, so what sits in it matters, and data laid out badly wastes it.

The cache moves data in runs, not in single bytes. Each run is called a line, and on this chip a line is 64 bytes (CPU-World's spec listing for the 3250U). When the processor needs one byte, it loads the whole 64-byte line that byte sits in. One line holds sixteen 4-byte numbers, because 16 times 4 is 64. One line holds eight 8-byte numbers, because 8 times 8 is 64. So a single load can bring sixteen useful values, or one useful value and sixty bytes of dead space, depending entirely on how the data is laid out. The processor does not choose. The layout chooses.

The design rule follows from that: data the program uses together sits together in memory, so one load brings a whole run of useful values instead of one value and a lot of dead space. Work through two layouts of the same small set of values to see the difference. Take 16 four-byte numbers, 64 bytes of numbers in all.

In the packed layout, the 16 numbers sit one after another, in one run of 64 bytes. One load brings the whole line, and all 16 numbers are in the cache. The program walks the list, and every number it needs is already there. One load, sixteen values.

In the scattered layout, each number sits alone in its own 64-byte line, with other data or empty space between them. The program needs 16 loads, one per number. Each load brings 4 useful bytes and 60 bytes that are not the numbers. Total moved: 16 lines times 64 bytes is 1,024 bytes, to use 64 bytes of numbers. Dead weight: 960 bytes, moved for nothing. Sixteen loads, sixteen values, and 960 bytes of waste.

| Layout | Loads to read all 16 numbers | Bytes moved | Useful bytes | Dead bytes |
| --- | --- | --- | --- | --- |
| Packed, one run | 1 | 64 | 64 | 0 |
| Scattered, one per line | 16 | 1,024 | 64 | 960 |

The count is loads, not time. This chapter claims no timings, because none were measured. A load count is arithmetic, and the arithmetic is above. The measured timings, how long a cache hit and a cache miss actually take on the target machine, belong to Chapters 31 and 32, where the numbers are measured on the real PC and reported with their method. What the layout table shows is enough for the design: the packed layout moves one sixteenth of the bytes the scattered layout moves, for the same sixteen numbers.

The scattered layout is what you get when each number lives in its own separate box somewhere in memory, which is how some languages store the numbers in a list. The packed layout is what a list already gives, from section 5.2: a list is one run by design. The language does not need a special cache feature to get the packed layout. It gets it from the list shape itself. That is the design working the way it should: the simple shape is also the fast shape, and the programmer gets the fast shape by using the ordinary shape.

Records work the same way. A list of the 4-byte contact records from section 5.2 packs 16 records into one 64-byte line. A program walking the list to read every age loads one line and gets 16 records at once, ages and all. If the ages lived in a separate list from the names and scores, the program would load one line for the ages and another for the rest. The fields used together sit together, in the record, and the record is the unit that moves. The kernel in Chapter 25 follows the same rule for its own data, because the cache does not care which layer the data belongs to. It only cares whether the data sits together.

Two cores add one more wrinkle, stated briefly because it is real. Each core has its own L1 and L2, and they share the L3. A value one core is using lives in that core's cache. Passing data between cores costs a trip through the shared memory, because the second core cannot read the first core's private cache. The design answer is the same as everywhere else in this chapter: keep the data a core is working on together, and pass as little as possible between cores. Chapter 24, on processes and scheduling, works out what that means for two cores sharing work.

Two honest notes close the section. The first is about control. The cache is managed by the hardware, not the program. The program cannot order the cache around, cannot pin a value in it, and cannot ask what is in it. What the program can do is lay its data out so the cache's natural behavior helps instead of hurts: runs instead of scatter, together instead of apart. The second note is about what this section does not claim. The numbers here are sizes, not speeds. No claim is made about how many nanoseconds a cache hit or a cache miss takes, because that was not measured on the target machine, and Chapter 3 forbids numbers without a method. The sizes are sourced, the load counts are arithmetic, and the timings wait for the real PC.

The connection to Chapter 1 is the reason this section is in the book at all. Section 1.5 named four numbers, and moving data touches all of them: moving fewer bytes is faster, uses less RAM, and uses less energy, because moving data costs power. Section 1.2 counted the copies a string suffers on its way to the screen. The cache is the same lesson at a smaller scale: every byte moved for nothing is a byte the machine paid for and the user did not ask for. Pack the data, move it once, use all of it. That is the rule, and it costs the programmer nothing, because the list and the record already lay data out that way.

## 5.4 What the language refuses to hide

The first three sections said what values are and how they sit. This section says what the language refuses to hide about them, and it is the section that gives the chapter its spine.

The rule is one sentence: a line of code must look like what it does, and cost what it looks like it costs. If a line copies a large value, the line says copy. If a line changes a number from one kind to another, the line says convert. If a line grows a list, the line says grow. Nothing expensive happens in silence. Nothing cheap-looking does ten times the work it appears to do. The programmer reads the program and knows what the machine will do, because the program says what the machine will do.

Say plainly what is refused, with a concrete example for each.

Silent type changes are refused. Adding a 1-byte number to a 2-byte number does not silently become a 2-byte number. The conversion is written out, in the code, where the reader can see it, or the compiler refuses the line. A number that quietly becomes a different kind of number is a bug the language cannot see, so the language does not allow it to happen quietly. The programmer who wants the conversion writes it. The programmer who did not want it finds out at compile time, not when the program misbehaves.

Hidden copies are refused. Passing a list to a function does not silently copy the list. Copying a large value is written out as a copy, and the cost is visible in the line that does it. A program that copies a list of ten thousand records does ten thousand records' worth of copying, and the line that does it should look like the work it is. A language that copies in silence lets a programmer write a one-line call that does a mountain of work, and then wonder why the program is slow. Riza does not let that happen. If the work is big, the line says so.

Silent overflow is refused. Adding 1 to 255 in a 1-byte number does not silently become 0. The language says what happens, and what happens is an error the program can see and handle, the way Chapter 7 describes errors as values, not crashes. Or the programmer picks a bigger size, and the number fits. Either way, the answer is stated, not hidden. A number that wraps around in silence is a lie the program tells itself, and a program built on lies is a program that fails in ways nobody can explain.

Hidden growth is refused. A list that grows past its room does not silently move to a bigger run. Growing is an act the code performs, with a visible cost, the way section 5.2 described the copy. The programmer who grows a list in a loop sees the growth in the code, and can decide whether the loop is the right shape. The programmer who never sees the growth cannot make that decision, because the language made it for them, in secret.

Hidden memory is refused. No value appears from nowhere. Every value's storage is accounted for, and the program's own structure says when memory is done. Chapter 6 builds that: ownership in plain words, arenas that free together, and no background collector scanning memory on a schedule nobody can see. The refusal here is the same refusal as in section 2.4: no hidden work. A background collector is hidden work. A hidden copy is hidden work. A silent conversion is hidden work. The language refuses all of them, for the same reason.

The payoff is the reason the refusals are worth their price. The programmer can read the program and know what the machine will do. The cost of any line is visible in the line itself. A line that looks cheap is cheap. A line that looks expensive is expensive, and the programmer who wrote it knew that when they wrote it. There is no gap between the program as written and the program as run, and that gap is where most performance mysteries live. Close the gap, and the mysteries go away with it.

The tie to Chapter 3 is direct, and it is why the refusals are in this book at all. Section 3.3 says a chapter counts as done when every number is measured or sourced and every claim is checkable. A language that hides its costs cannot be measured honestly, because the costs are not in the program to be counted. The scoreboard in section 1.5 asks for the RAM, the speed, and the energy of a task. If the language copies a list in silence, the RAM number is a guess. If it grows a list in silence, the speed number is a guess. If it converts a number in silence, the energy number is a guess. A scoreboard built on guesses is worse than no scoreboard, because it looks like proof and it is not. The language refuses to hide, so the book can refuse to guess.

The honest counterpoint is that visibility costs the programmer some typing. Writing a conversion is a line of code. Writing a copy is a line of code. A language that hides those lines feels lighter to write, and that lightness is real. The book's answer is that the line is the price of knowing, and the price is worth paying. A language that saves the programmer a line and hides a cost has charged the user instead, in RAM, in time, in power, and the user never agreed to pay. The programmer writes the program once. The user runs it every day. The cost belongs where it can be seen, which is in the program, not where it cannot, which is in the user's machine. And the editor helps: Chapter 8's text view can show the cost next to the line, so the programmer sees it without writing it twice. The visibility is in the tool, not only in the typing.

The refusals are what make the rest of the chapter's arithmetic mean something. A size that is stated is a size that can be checked. A cost that is shown is a cost that can be measured. A program that says what it does can be judged against the four numbers, and a program that hides what it does cannot. The language refuses to hide, and the book refuses to guess, and those two refusals are the same refusal, stated twice.

The values are defined now: numbers of stated sizes, text stored once and called by number, true and false in a byte, lists in one run, records at fixed positions, files as runs of bytes with names, all laid out so the cache can do its job, and none of it hidden. The next chapter says where those values live, and when they are freed: memory without a garbage collector.

---

The types are the values the dictionary moves around, and every one of them has a stated size. Numbers come in 1, 2, 4, and 8 bytes, with the ranges shown as powers of two and the sign taking half the range. Text is characters stored once in the name table and called by number, with wider characters taking more bytes the same way. True and false are one bit of meaning in a byte. Lists are one run with a length, records are fields at fixed positions, and files are runs of bytes with names. Data used together sits together, so one 64-byte cache line brings sixteen 4-byte numbers instead of one, and the cache sizes on the target machine are stated from AMD's own spec page. And the language refuses to hide: no silent type changes, no hidden copies, no silent overflow, no hidden growth, no hidden memory. The cost of any line is visible in the line itself, so the book can measure what the program does instead of guessing.
