# Chapter 4: The dictionary: short codes for common work

Chapter 1 was the diagnosis. Chapter 2 was the idea. Chapter 3 was the rules. Part 1 is done, and Part 2 begins here, with the language itself.

This chapter is where the idea from section 2.1 becomes a real design. Prediction is compression: what you can predict, you do not have to store. A program is highly predictable. The same operations appear again and again. The same names appear again and again. So this chapter builds the dictionary: a shared codebook that gives the most used operations the shortest codes, and stores every name once instead of writing it out every time it is used. The dictionary is the first piece of the language, and every program in the system shares it.

There are four sections. The first says how the ranking is decided: by counting what real programs actually do, not by guessing. The second lays out the encoding: one byte for the common operations, two bytes for the next tier, and a long tail for everything rare. The third covers names and symbols: stored once in a table, called by number everywhere else. The fourth works one small program all the way through, text form against dense form, with every byte counted so any reader can check the arithmetic.

One warning, stated up front so it is not missed. This chapter designs the dictionary. It does not fix the final list of codes. The final list is built later, from measured frequency, and it lives in Appendix A, Opcode dictionary v0.1. The code values shown in this chapter are illustrations of the scheme, clearly marked as such, so the reader can see how the scheme works before the real counts exist. A design chapter that pretended the counts were already in would be guessing, and Chapter 3 forbids guessing.

## 4.1 Frequency ranking: most used gets shortest code

Morse code is the oldest example of the rule this chapter is built on, and Chapter 2 already named it. In Morse code, the most common letter in English, E, gets the shortest code: a single dot. T gets a single dash. A rare letter like Q gets a long code, because a rare letter is a surprise and a surprise is worth paying for. The code is short where the letter is common, and long where the letter is rare. That is prediction turned into storage, and it is exactly the rule for the dictionary.

Programs follow the same shape. A program is not a random string of operations. The same operations appear again and again: add, compare, jump, load, store. Section 2.1 listed them, and the list is short because the work a program does is built from a small set of steps repeated endlessly. A loop that walks a list does the same few operations once per item. A check that a value is in range does the same compare every time it runs. The same names appear again and again too: a variable defined once and used fifty times. A program is much closer to the repeated message from section 2.1 than to the random one. The message `ha ha ha ha ha ha ha ha` shrank from 184 bits to 32 bits because the pattern was stored once and the count with it. A program shrinks the same way, because its patterns repeat the same way.

So the rule for the dictionary is simple: the most used operation gets the shortest code. The next most used gets the next shortest. The rare operations get the long codes, because they are used so seldom that the extra bytes cost almost nothing in total. The ranking is the whole design. Get the ranking right and the dictionary is small. Get it wrong and the dictionary wastes bytes on the common path, which is the one place waste matters.

How the ranking is decided is as important as the ranking itself, because a ranking decided by taste is a guess. The ranking is decided by counting. When the compiler from Chapter 11 exists, it counts the operations in real programs: the standard library, the tools, the programs people actually write and run. It counts how often each operation appears, sorts the counts, and the most used operations sit at the top of the list. That list is data, not taste. Two people running the same count on the same programs get the same ranking, and if they disagree, the count settles it. The ranking is not a matter of opinion. It is a matter of arithmetic.

There is a name for this idea, and it is worth knowing, because the reader will meet it elsewhere. In 1952, David Huffman published a method for building codes exactly this way: give short codes to common things and long codes to rare things, choosing the lengths so the total size is as small as the counts allow. It is called Huffman coding, after him. The dictionary in this chapter is the same idea applied to operations instead of letters. The difference is in the details, which section 4.2 covers: Huffman codes are measured in bits, and the dictionary is measured in bytes, for a reason that section gives. The principle is the same, and it is older than this book by more than seventy years.

One honest question has to be answered before the design goes further. What if the counts come back flat? What if real programs use every operation about equally often, so that no ranking helps? Then the scheme in this chapter gains nothing, and the book would say so, because rule 7 of section 1.5 requires a loss to be reported as a loss. But the bet is a safe one, and the reason is visible in ordinary language. In English, a small number of words do most of the work: `the`, `and`, `of`, `to`, and a handful more make up a large share of everything written, while most words in the dictionary appear rarely. Code is the same shape, for the same reason: a program is built from a small set of steps, and the steps that build everything are the steps that appear everywhere. The counts will confirm the shape, and the shape is what the dictionary is built on.

The ranking is not fixed forever. As programs change, the counts change, and a dictionary built from old counts slowly drifts away from what programs actually do. So the dictionary is versioned. The binary file format in Chapter 9 carries a version number in its header, and that version says which dictionary the file was written against. A new dictionary version is a deliberate act, taken when the counts have moved enough to matter, and every program written against the old version still runs, because the old dictionary is kept. A dictionary that changed silently would break every program written against it. A dictionary that never changed would slowly stop matching the programs it serves. Versioning is the middle way: the ranking follows the data, and the data is allowed to move, but only in the open, one version at a time.

One more thing about what is being counted. The dictionary codes the operations of the shared middle form from section 2.3, the simple steps that sit between what the programmer writes and what the machine runs. That matters, because it means one dictionary serves the whole system. The compiler produces middle form operations. The tools read them. The shell executes them. The kernel schedules them. Every layer uses the same codes, because every layer uses the same form. A dictionary built for one layer would save that one layer. A dictionary built for the shared form saves every layer at once, and the tools that understand it understand every program in the system. Chapter 10 defines the middle form itself. This chapter gives it short codes.

## 4.2 One byte ops, two byte ops, the long tail

A byte holds 256 values, numbered 0 to 255. That is the whole space the encoding has to work with, one byte at a time, and the scheme in this section spends it carefully. There are three tiers, and the tier an operation sits in is decided by its frequency rank from section 4.1.

The first tier is one byte. Values 0 to 254 are one byte operations, 255 of them. Each one is a single byte in the file, and reading it tells the loader everything it needs to know: which operation it is, and how many bytes follow it, if any. The most used operations live here. In the illustrations in this chapter, ADD is a one byte operation, and so is PRINT, because adding and printing are the kind of work a program does constantly. The final assignments come from the measured ranking and live in Appendix A. The illustrations are there to show the shape, not to fix the list.

The second tier is two bytes. Value 255 is reserved as a marker. It never means an operation by itself. It means "the operation is two bytes long, read the next one." A two byte operation is 255 followed by a second byte, and that second byte can be any value from 0 to 254, which gives another 255 operations. These are the operations that are common enough to matter but not common enough for the first tier. They cost one extra byte each time they appear, and that cost is acceptable precisely because they appear less often.

The third tier is the long tail. The value 255 followed by a second byte of 255 is a second marker. It means "the operation is in the long tail, read two more bytes." Those two bytes form a number from 0 to 65,535, which is room for 65,536 rare operations, each costing four bytes in total: the two markers and the two number bytes. Four bytes is a lot for one operation, but the operations in the long tail are rare by definition. A rare operation that appears once in a program costs four bytes once. A common operation that appears a thousand times costs one byte a thousand times. The total is what matters, and the total is dominated by the common path, which is exactly where the codes are shortest.

Here is the scheme in one table:

| Tier | First byte | What follows | Operations it holds | Cost each time |
| --- | --- | --- | --- | --- |
| One byte | 0 to 254 | Nothing | 255 | 1 byte |
| Two byte | 255 | One byte, 0 to 254 | 255 | 2 bytes |
| Long tail | 255, then 255 | Two bytes, 0 to 65,535 | 65,536 | 4 bytes |

Say plainly why the codes are measured in bytes and not in bits. A Huffman code measured in bits would be smaller still, because the most common operation could take fewer than eight bits. But a CPU reads bytes, not bits. A bit packed code has to be unpacked on every operation: shift, mask, and reassemble, before the operation can even start. A byte code is read directly. The machine reads one byte, and that byte is the operation. The book trades a little size for a lot of simplicity, and simplicity is speed on a machine with two cores and no GPU. Chapter 3 named the machine. This is one of the places the machine shapes the design. A design that saved a few bytes per program and cost time on every operation would be the wrong trade on this hardware.

There is a second advantage to the byte scheme, and it is about finding your place. In this encoding, the length of an operation is visible in its first byte. If the first byte is 0 to 254, the operation is one byte long, plus whatever operands follow it. If it is 255, the operation is longer, and the second byte says how much longer. The loader never needs a table to find where the next operation starts. It reads forward, one operation at a time, and the code itself says where each operation ends. A bit packed code cannot do that: the length of a code is only known by looking it up, and a single wrong bit throws off everything after it. The byte scheme is self describing in a way the bit scheme is not, and self describing is easier to check, which matters for Chapter 9, where the loader verifies a file before running it.

The trade-off of the whole scheme is worth stating in one sentence, because it is the same trade-off the book makes everywhere: short codes save space on the common path and cost nothing on the rare path, because the rare path was already long. An operation used once in a program can cost four bytes without anyone noticing. An operation used ten thousand times must cost one byte, because every extra byte is paid ten thousand times. The scheme spends its cheapest codes where the spending is heaviest. That is the whole idea, and it is why the ranking from section 4.1 matters more than any single code value.

A note on what follows an operation. An operation code says what to do. The bytes after it say what to do it to. A load operation is followed by a number saying which value to load. A store operation is followed by a number saying which name to store into. A jump is followed by a number saying where to jump to. These following bytes are called operands, from the Latin for "things to be worked on." They are part of the operation's cost, and the worked example in section 4.4 counts them, byte by byte, so the reader can see the full price of each operation, code and operands together. The number of operands an operation takes is fixed by the operation itself, so the loader always knows how many bytes to read after the code.

One last point, and it is about growth. The long tail holds 65,536 operations, which is far more than any language needs today. That room is deliberate. A dictionary that ran out of room would have to be rebuilt, and a rebuild would break every program written against it. The long tail is the room to grow: new operations go there first, and if one of them turns out to be common, the next dictionary version promotes it to a shorter code. The promotion is a versioned change, made in the open, the way section 4.1 describes. The dictionary starts small, grows slowly, and never runs out of room.

## 4.3 Names and symbols: intern once, reuse

Operations are only half the repetition in a program. The other half is names. A variable defined once is used again and again. A function defined once is called again and again. A text string written once is printed again and again. In text form, every use pays the full price of the name, spelled out in full, every single time. This section removes that cost.

The method is called interning. To intern a name means to store it once in a table and use its number everywhere else. The table holds the name itself, spelled out, exactly once. Every place the program uses the name holds a short number that points at the table entry. The name is written once, in the table, and referred to by number everywhere else. That is the whole method, and it is the same idea as section 4.1 applied to names instead of operations: the repeated thing is stored once, and the uses point at it.

Work through the arithmetic on a real name. Take `calculate_total`, a variable name a programmer might actually write. Count it: 15 characters. Suppose the program uses it 50 times. In text form, each use costs 15 bytes, so 50 uses cost 750 bytes, and that is counting only the name itself, not the rest of the lines it sits on. Interned, the name is stored once in the table: 15 characters plus one byte saying how long it is, 16 bytes. Each use costs one byte, the table number. Fifty uses cost 50 bytes. Total: 16 plus 50 is 66 bytes, against 750 for the text form. Under a tenth of the size, and the saving grows with every additional use, because each new use costs one byte in the dense form and 15 in the text form. The longer the name and the more it is used, the bigger the win, which is exactly the shape of real programs, where the most used names are the ones the programmer gave clear, long, readable names to.

The table itself has a simple shape, and it is worth writing down here, because the worked example in section 4.4 uses it. The table starts with one byte saying how many entries it holds. Then each entry is a length byte followed by the characters of the name, spelled out once. That is all. No separators, no padding, no wasted bytes. A reader who wants to find entry 3 walks past entries 0, 1, and 2, using each length byte to know where the next entry starts, the same way the loader walks through operations using the first byte to know each operation's length. The shape is simple on purpose, for the same reason the operation codes are simple: the loader reads forward, and the data itself says where each piece ends.

Text strings live in the same table as names. A string like `hello` is stored once, as an entry, and used by number, exactly like a variable name. There is no difference in the mechanism, and there does not need to be one. A name and a string are both text the program repeats, and the table does not care what the text means. It only stores it once. The full story of text, what it is and how it behaves, belongs to Chapter 5. This section only needs the storage rule: text is stored once and called by number.

Two dictionaries, and they must not be confused. The opcode dictionary is the one this chapter is about: the shared codebook of operations, the same for every program in the system, published as Appendix A. The name table is different: it belongs to one file, and it lives in the dictionary section of that file's binary form, which Chapter 9 defines. Every file carries its own name table, because every file has its own names. The opcode dictionary is shared because every program uses the same operations. The name table is per file because every program has its own names. The name table is sometimes called a symbol table, and the reader may meet that name elsewhere. It means the same thing: the table where a file's names live.

The connection to Chapter 1 is direct. Section 1.1 counted the waste of text parsed and thrown away on every use: the same names read, split into tokens, and arranged into a tree, every time the program runs, and the tree thrown away when the program ends. Interning removes the repetition at the source. The name is written once, in the table, and never spelled out again. The loader reads the table once, in one run, and every use of the name after that is a number, not a string to be parsed. There is nothing left to re-translate, because the translation already happened, once, when the program was written. Binary first, text second, from section 2.2, is what makes this possible: the stored form is the dense form, and the text view the programmer sees is generated from it, names and all, whenever it is needed.

The connection to Chapter 3 is about the disk. Section 3.1 said the spinning drive reads fastest in long sequential runs and slowest when it jumps between scattered files, and that files in Riza are laid out to be read in long runs. The name table is one run: a single stretch of bytes, read from start to finish, with no jumps. The code section is another run. The loader reads the file in a small number of long reads, instead of hunting through many small ones. Fewer bytes and longer runs are two sides of the same saving, and interning gives both: the file is smaller, because names are stored once, and the reading is simpler, because the names sit together in one place.

There is one more gain, and it is about the programmer, not the machine. In text form, a long clear name costs bytes every time it is used, so programmers are pushed toward short cryptic names to save space and typing. In the dense form, a long name costs bytes once, in the table, and one byte per use after that, no matter how long it is. The cost of a clear name is nearly nothing. So the system rewards the names a human would want to read: `calculate_total` instead of `ct`, `customer_address` instead of `ca`. The machine does not care how long a name is, because it only ever sees the number. The human sees the name, in the text view, every time. Chapter 8 defines that text view and proves the round trip is exact: text to binary and back produces the same text, with nothing lost. The name the programmer wrote is the name the programmer sees, and the number the machine stored is the number the machine runs. Neither side pays for the other's form.

Small numbers are handled differently, and the difference is worth one paragraph. A small number, like 3 or 100, rides in the code itself, as a byte right after the operation that uses it, the way the worked example in section 4.4 shows. It does not need a table entry, because it is already short: one byte is the shortest a reference can be, and the number itself is one byte. Putting it in the table would cost a byte for the reference plus a byte for the value, which is more, not less. The table is for things that are long enough to be worth storing once. Numbers get their full treatment in Chapter 5, where the language decides how numbers of every size are stored. For now, the rule is simple: short things ride inline, long things live in the table.

## 4.4 Worked example: same program, text vs dense form

This section works one small program all the way through, both ways, with every byte counted. The program is a loop that prints the word `hello` three times. It is small on purpose: small enough that the reader can count every byte by hand, and real enough that the counting means something. The text form is shown in a plain readable form close to Python. The exact text syntax of Riza is defined in Chapter 8. The byte count here is of the text as shown, and the text as shown is a realistic human form, not a shortened one.

First, the text form, with the byte count of each line. Every line ends with a newline character, the invisible character that marks the end of a line, and the newline counts as one byte, because it is stored in the file like any other character.

| Line | Text | Bytes |
| --- | --- | --- |
| 1 | `count = 0` | 9 characters + 1 newline = 10 |
| 2 | `while count < 3:` | 16 characters + 1 newline = 17 |
| 3 | `    print("hello")` | 18 characters + 1 newline = 19 |
| 4 | `    count = count + 1` | 21 characters + 1 newline = 22 |
| Total | | 68 bytes |

Check the counts by hand. Line 1 is `count`, a space, `=`, a space, `0`: 5 plus 1 plus 1 plus 1 plus 1 is 9. Line 2 is `while`, a space, `count`, a space, `<`, a space, `3`, `:`: 5 plus 1 plus 5 plus 1 plus 1 plus 1 plus 1 plus 1 is 16. Line 3 is four spaces of indentation, then `print("hello")`, which is 14 characters: 4 plus 14 is 18. Line 4 is four spaces, then `count = count + 1`, which is 17 characters: 4 plus 17 is 21. Add the four newlines, and the total is 68 bytes. Nothing is hidden. A reader with the text above can recount it and get the same number.

Now the dense form. It has two parts: the code, and the name table from section 4.3.

The code uses the illustrative operation values from section 4.2. They are illustrations, not the final codes: the final codes come from measured frequency and live in Appendix A. The code works on a stack, which is a pile of values: an operation puts values on top of the pile, or takes values off the top and puts a result back. LOAD puts a value on the pile. ADD takes two values off the top, adds them, and puts the answer back. PRINT takes the top value off and prints it. The positions are counted in bytes from the start of the code, starting at 0, so the reader can check that each operation starts where the one before it ended.

| Position | Bytes | Meaning |
| --- | --- | --- |
| 0 | `10 00` | LOAD_CONST 0: put the number 0 on the pile |
| 2 | `30 00` | STORE name 0: take the top value off and store it as `count` |
| 4 | `31 00` | LOAD name 0: put `count` on the pile |
| 6 | `10 03` | LOAD_CONST 3: put the number 3 on the pile |
| 8 | `21` | LESS_THAN: take two values off, put back true if the first is less than the second |
| 9 | `50 17` | JUMP_IF_FALSE to position 23: if the top value is false, jump to END |
| 11 | `11 01` | LOAD_STR entry 1: put the string `hello` on the pile |
| 13 | `40` | PRINT: take the top value off and print it |
| 14 | `31 00` | LOAD name 0: put `count` on the pile |
| 16 | `10 01` | LOAD_CONST 1: put the number 1 on the pile |
| 18 | `20` | ADD: take two values off, put their sum back |
| 19 | `30 00` | STORE name 0: take the top value off and store it as `count` |
| 21 | `51 04` | JUMP to position 4: go back to the loop test |
| 23 | `00` | END |

Check the positions. The first operation is 2 bytes, so the second starts at position 2. The second is 2 bytes, so the third starts at 4. The fifth operation, LESS_THAN, is 1 byte at position 8, so the sixth starts at 9. The last operation, END, is 1 byte at position 23, so the code is 24 bytes in total. The jump targets check out too: JUMP_IF_FALSE carries the byte `17` in hexadecimal, which is 23 in ordinary counting, the position of END. JUMP carries `04`, position 4, the start of the loop test. A reader can follow the loop by hand: count starts at 0, the test passes, `hello` prints, count becomes 1, back to the test, and so on until count reaches 3, the test fails, and the jump lands on END.

The name table holds the two pieces of text the program uses: the name `count` and the string `hello`. It follows the shape from section 4.3: one byte saying how many entries, then each entry as a length byte followed by the characters.

| Bytes | Meaning |
| --- | --- |
| `02` | Two entries in the table |
| `05 63 6F 75 6E 74` | Entry 0: length 5, then `count` |
| `05 68 65 6C 6C 6F` | Entry 1: length 5, then `hello` |

The table is 1 byte for the count, plus 6 bytes for the first entry, plus 6 bytes for the second: 13 bytes in total.

Now the totals, side by side:

| Form | Bytes |
| --- | --- |
| Text form | 68 |
| Dense form, code | 24 |
| Dense form, name table | 13 |
| Dense form, total | 37 |

The dense form is 37 bytes against 68 for the text form: 54 percent of the size, a saving of 31 bytes, or 46 percent smaller. The arithmetic is all above, and any reader can recount it. That is the point of listing every byte. Section 3.3, part 3, says every claim in this book must be checkable, and this section is checkable in the plainest way there is: count the bytes.

Read the result honestly, because a single small example proves less than it might look like. The saving is real, but the program is small, and on a small program the fixed costs loom large. The name table costs 13 bytes, more than a third of the dense total, and it would cost the same 13 bytes for a program ten times longer, because the same two entries would serve hundreds of uses. The full binary file, defined in Chapter 9, adds a header of a few fixed bytes on top of the code and the table, and those bytes are not counted here. On a program this small, the header matters. On a large program, it fades to nothing. The honest statement is that the dense form wins by more as the program grows, because the dictionary is paid once per file and the uses are paid per use, and the uses are where the program's size lives. A claim that every program shrinks by 46 percent would be a lie built on one example, and this book does not build claims that way.

Two more honest notes, and then the chapter closes. The first is about what this example is not. It is not the output of a compiler, because no compiler exists yet. Chapter 11 builds the first one. The dense form above was written by hand, following the scheme in sections 4.2 and 4.3, to show what the scheme produces. When the compiler exists, it will produce dense forms like this one automatically, and Chapter 11 tests every opcode to prove it. The second note is about the scoreboard. Section 3.3, part 4, says a chapter that claims an improvement fills in its scoreboard table. This chapter claims no measured improvement on real hardware, so no scoreboard table is due. The numbers in this section are arithmetic on an illustration, counted by hand, not measurements of a running system. The measured comparison, dense form against today's stack, on the target machine, comes in Chapter 32, where every scoreboard table in the book is collected and every claim is checked against real numbers.

The dictionary is the first piece of the language, and it is built on one rule: the most used gets the shortest code, and the repeated gets stored once. The next chapter says what the values are that the dictionary moves around: numbers, text, true and false, and how they sit in memory.

---

The dictionary is the idea from section 2.1 made into a design. Count what real programs do, and give the most used operations the shortest codes: one byte for the common work, two bytes for the next tier, and a long tail with room for 65,536 rare operations. Store every name and every string once in a table, and call it by number everywhere else. A small program worked through by hand shows the shape of the saving: 37 bytes of dense form against 68 of text, every byte listed, every byte checkable. The final codes come from measured frequency, and they live in Appendix A, built when the compiler in Chapter 11 can count for real.
