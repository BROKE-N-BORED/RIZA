# Chapter 11: The bootstrap compiler

Chapter 9 defined the file. Chapter 10 defined the form. Every worked example so far was written by hand: the dense form in section 4.4, the branch in section 7.2, the round trip in section 8.2, the complete file in section 9.2, the translation in section 10.3. A hand written example proves the design is checkable. It does not produce programs. This chapter builds the first tool that does: the compiler.

A compiler turns text into the binary file. It reads the programmer's text, checks it, produces the middle form from Chapter 10, and writes the file from Chapter 9. That is the whole job, and this chapter designs the smallest version that does it, says what language the first one is built in, and names the tests that prove it works.

There are three sections. The first names the smallest compiler that works: three stages, one job each, and a plain list of what it does not do. The second says why the first compiler is built in an existing language, and which one. The third names the tests: one for every opcode, one for every refusal, one for every validity check, and one for every round trip the book has counted.

One warning, stated up front the way Chapters 4 through 10 stated their own. This chapter designs the compiler. It does not build it. No compiler exists yet, and no claim in this chapter says one does. The chapter also claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own example, counted by hand. The measured proof comes in Chapter 31, on the real PC, with the comparison against today's stack in Chapter 32.

## 11.1 The smallest compiler that works

Start with the job, in one sentence. A compiler reads the programmer's text, checks it, produces the middle form, and writes the file. Text in, file out, with the checks in between. Everything in this section is that sentence, worked out in detail.

The smallest compiler that works has three stages, and each stage is one job.

Stage one reads and checks the text. It takes the programmer's text, in the form Chapter 8 designed, and reads it: splits it into tokens, builds the tree, and reports mistakes in plain words, naming the line. Then it checks the program, and the checks are the rules the earlier chapters promised the compiler would enforce. It checks the types, from Chapter 5: a number added to a string is refused, a value used as the wrong kind is refused, a conversion the code does not state is refused. It checks the ownership, from Chapter 6: a value freed twice is refused, a value used after its owner freed it is refused, a value with no owner is refused. It checks the errors, from Chapter 7: a result used without checking for the error is refused. It checks the calls, from section 7.1: a call that hands a function the wrong kind of value is refused, a call that hands it the wrong number of values is refused. And it checks the names: a name used before it is defined is refused, because a name that was never defined has no value to load.

If any check fails, the compiler stops. It reports the failure in plain words, naming the line and saying what is wrong, and it writes no file. A program the compiler refuses is a program that never becomes a file, and that is the point of the checks: the refusal happens at build time, when the programmer is looking at the code, not at run time, when the user is waiting. Section 6.1 said the checking costs the program nothing while it runs, because it already happened, once, when the program was built. This is where that once happens.

Stage two produces the middle form. It takes the checked tree and turns it into the list of simple steps Chapter 10 defined, the form Appendix C specifies: loads, stores, arithmetic, jumps, calls, returns, and the steps that touch the world. It builds the name table, in the shape section 4.3 defined, storing each name and string once and numbering the entries. It assigns positions, counting in bytes from the start of the code, the way sections 4.4 and 9.2 counted them, and it computes the jump targets, so a jump forward lands past the branch and a jump back lands on the loop test. It builds the function mapping, in the shape section 9.2 defined, recording where each function's code starts. The output of stage two is exactly the middle form: the operations, in order, with their operands, and the tables that go with them.

Stage three writes the file. It takes the middle form and the tables and lays them out in the shape Chapter 9 defined: the 33 byte header, with the magic, the version, the entry point, and the offset and size of each section; the code section; the data section; and the dictionary section, with the name table, the function mapping, and the comments. The output of stage three is the file, ready for the loader to read. The compiler's job is done when the file is written. It does not run the program. The loader does that, the way Chapter 9 describes.

Say plainly what the smallest compiler does not do, because the list is as much a part of the design as the three stages.

It does not optimize. The optimizer is Chapter 12, and it only does what pays. The smallest compiler produces the middle form straight from the text, step for step, with no cleverness. A program that computes a value and never uses it keeps the computation, because removing it is the optimizer's job, not the compiler's. The compiler's job is to be correct, and correctness is easier to prove when the compiler does the plain thing.

It does not translate to native code. The translation to the machine's own instructions happens at load time, inside the loader, the way section 10.3 designed it. The compiler writes the middle form, and the middle form is what the file holds. The translator reads it at load and writes the native instructions then. The compiler never sees a native instruction, and the translator never sees the programmer's text. Each does its own job, on its own form.

It does not build the full grammar's every corner at once. Section 8.1 said the full grammar, the exact rules for every corner, grows with the compiler in this chapter, as a reference document, written down and checked. The smallest compiler reads the shapes Chapter 8 fixed: values, lists, records, functions, branches, loops, error checks. As the language grows, the grammar grows with it, one rule at a time, each rule written down and tested before it is used. A grammar built all at once is a grammar nobody has checked. A grammar built one rule at a time is a grammar the tests can keep up with.

It does not have a fancy interface. The compiler is a tool that reads a file and writes a file. The editor is Chapter 8's, and it shows the text, the costs, the errors, and the ownership. The build tool is Chapter 14's, and it runs the compiler with one command. The compiler itself needs no window, no menu, and no settings panel. It needs an input, an output, and an error message in plain words when the input is wrong.

Now the parser question, because a careful reader will ask it, and it needs an honest answer. Section 2.2 said the parser lives in the editor, where a human is writing and needs it, and not in the loader, which runs every time a program starts. The compiler needs a parser too: stage one reads the text, and reading the text is parsing. So where does the compiler's parser live?

The answer is the one section 2.2 already gave: one parser, maintained once, used by every tool. The editor's parser and the compiler's parser are the same parser, shared, not two parsers written separately. The parser is one piece of code, with one set of rules, and both tools call it. The editor calls it while the programmer writes, to show the errors where they happen. The compiler calls it at build time, to read the text it is about to compile. Because it is the same parser, the editor and the compiler can never disagree about what a program means. A program the editor accepts, the compiler accepts. A program the compiler refuses, the editor already marked. If the parser were written twice, the two copies would drift, and a program one tool accepted the other might refuse, or worse, read differently. One parser removes that whole class of bug, and it is why section 2.2 said one parser is better than a parser reimplemented inside every runtime.

The parser is written first as part of the host compiler, in section 11.2, because the compiler is the first tool that needs it. The editor uses the same parser when it is built. The loader never uses it, because the loader never parses. One parser, two users, and the loader is not one of them.

One more job belongs to the compiler, and it is the job that makes Appendix A possible. Section 4.1 said the ranking of operations is decided by counting what real programs actually do, and that when the compiler from Chapter 11 exists, it counts the operations in real programs: the standard library, the tools, the programs people actually write and run. The compiler counts while it compiles, at no extra cost, because it already walks every operation in every program it builds. It keeps a tally: how many times each operation appears, across all the programs it has compiled. That tally is the data the final dictionary is built from, and it is data, not taste. Two people running the same count on the same programs get the same ranking, and if they disagree, the count settles it.

Now work one small program through the compiler, stage by stage, the way sections 4.4 and 7.2 worked their examples, with every byte counted. The program is the hello loop from section 4.4, the loop that prints `hello` three times, because its numbers are already counted and checked in Chapters 4 and 9, and the compiler's output must match them exactly.

Stage one reads the text. The text is 68 bytes, the four lines from section 4.4, and the parser reads them into the tree: an assignment, a loop with a test and two lines inside it. Then the checks run. The types check: `count` is a number, because 0 is a number; `count < 3` compares two numbers, so it gives a true or false; `print` takes a string, and `hello` is a string. The ownership checks: `count` is owned by the program, made on line 1 and used until the loop ends; `hello` is a string in the table, stored once. The error checks: no call in this program can fail, so no error check is needed, and the compiler notes that instead of demanding one. The name checks: `count` is defined on line 1, before it is used on line 2. Every check passes, and stage one hands the tree to stage two.

Stage two produces the middle form. The tree becomes the 14 operations from section 4.4, 24 bytes of code, with the jump targets computed: JUMP_IF_FALSE to position 23, the END, and JUMP back to position 4, the loop test. The name table is built: two entries, `count` and `hello`, 13 bytes. The dense form is 37 bytes, 24 of code and 13 of table, exactly the count section 4.4 reached by hand.

Stage three writes the file. The header is 33 bytes: the magic R I Z A, version 1, entry point 0, the code at offset 33 with size 24, the data at offset 57 with size 0, the dictionary at offset 57 with size 15. The code section is the 24 bytes from stage two. The data section is empty, because the program has no static values. The dictionary section is 15 bytes: the 13 byte name table, the function mapping with a count of 0, and the comments table with a count of 0, because the program has no functions and no notes. The file is 72 bytes, exactly the count section 9.2 reached by hand.

| Stage | What goes in | What comes out | Bytes |
| --- | --- | --- | --- |
| Text | | The programmer's four lines | 68 |
| Stage 1: read and check | 68 bytes of text | The tree, every check passed | No file yet |
| Stage 2: middle form | The tree | 14 operations, name table | 37 |
| Stage 3: write the file | Middle form and tables | The complete file | 72 |

The table is the compiler's job in miniature. Text in, checks, middle form, file out. And the numbers must match the hand counts, because the hand counts are the design and the compiler's output is the design carried out. If the compiler produced a 73 byte file for this program, the compiler would be wrong, not the hand count. Section 11.3 tests exactly that: the compiler's output against the counts the book already made by hand.

One honest note about what stage one does not produce. The tree is the compiler's working form, not something stored in the file. It lives in the compiler's memory while the compiler works, and it is gone when the compiler finishes. So stage one has no byte count in the table, because there is nothing to count. The tree is a tool the compiler uses, not a product it ships.

## 11.2 Built first in an existing language

The compiler in section 11.1 has a problem, and it is the oldest problem in computing. The compiler is a program, and a program needs a tool to run it. But the tool that runs Riza programs is the compiler itself, and it does not exist yet. You cannot write the first compiler in a language that does not exist yet, because there is no tool to run it. The language needs the compiler, and the compiler needs the language, and neither one exists first. That is the bootstrap problem.

The answer is to write the first compiler in a language that already exists on the target machine. Not in Riza, because Riza has no compiler yet. In a language the machine can already run, using tools the machine already has. The first compiler is written in that language, it reads Riza text, and it writes Riza binary. Once it works, it can build the first Riza programs, and one of those programs is the next compiler, written in Riza itself, which is Chapter 13's work. The first compiler lifts the second one into existence, and then the first one is no longer needed.

The host language is Python 3.12. It is already on the machine: the table in section 3.1 lists Python 3.12.3, installed on the Mint PC as it stands today. It costs nothing, which the hard $0 rule in section 3.2 requires. It needs no new tools to install, no new language to learn, and no network to fetch anything, which the offline first rule requires. And it is the language this manual's own tooling is written in, the way section 1.1 said, so the machine that writes the manual is the machine that builds the first compiler, with nothing added.

Say plainly what the host compiler must do, because the list is short and strict. It must read Riza text, in the form Chapter 8 designed, and write exactly the middle form Chapter 10 defines and exactly the file Chapter 9 defines. Exactly means exactly: the same operations, the same layout, the same header, the same sections, the same byte counts. The file format is the contract between the compiler and the loader, and a file the loader cannot read is a file the compiler wrote wrong. The host compiler's output must match the hand counts in Chapters 4 and 9, byte for byte, because those counts are the design. Section 11.3 tests that it does.

Say plainly what the host compiler must not do. It must not add features the language does not have. If the language has no feature, the compiler does not invent one, because a feature the compiler invents is a feature the language never agreed to, and the self-hosted compiler in Chapter 13 would have to invent it too, or the two compilers would disagree. It must not depend on anything the target machine does not already have. No packages fetched from the network, no libraries that need installing, no tools beyond Python itself. A compiler that needs a download to build is a compiler that breaks the offline first rule, and a compiler that needs a paid library breaks the hard $0 rule. The host compiler is Python and nothing else, running on the machine as it stands.

The host compiler is a ladder, not the building. It exists to produce the first Riza programs and the first Riza compiler written in Riza itself, and then it comes down. Chapter 13 ports the compiler into Riza, and once the Riza compiler builds itself, the system stands on its own: Riza programs built by a Riza compiler, running on a Riza loader, with no Python anywhere in the chain. The ladder is temporary by design, and the plan says so from the start, because a temporary tool with no plan to remove it has a habit of becoming permanent. If the ladder never comes down, the system never stands on its own, and the whole point of the build, a system that belongs to the person using it, is lost. So the ladder comes down in Chapter 13, and this chapter says so now, so the plan is on the record.

The host compiler does not need to be fast, and it does not need to be small. Those are not the qualities it is judged on, and saying so plainly matters, because the book judges everything else on size and speed. The host compiler runs once per build, not once per program run. A program the user runs a hundred times a day pays its startup cost a hundred times. A compiler the programmer runs once to build that program pays its cost once. Python's startup, 28 milliseconds and 10 MB from section 1.4, is paid once per compile, and once per compile is a cost the build can carry. And the host compiler is a tool, not part of the shipped system. Its size is not the size of any program it builds. A 10 MB compiler that builds a 72 byte program has not made the program 10 MB. The program is 72 bytes. The compiler's size stays in the workshop.

What the host compiler needs is to be correct, because every Riza program passes through it. A bug in an ordinary program is a bug in that program. A bug in the compiler is a bug in every program the compiler builds, because the compiler writes the code every program runs. A compiler that writes the wrong jump target breaks every loop it compiles. A compiler that miscounts the name table breaks every name it stores. So correctness is the one quality the host compiler is judged on, and it is judged strictly: the compiler's output must match the design, byte for byte, on every program it builds. Section 11.3 is how that is proved, test by test.

Section 3.1 set one limit on the build itself, and it applies here. A compile that takes an hour is a compile nobody runs twice. The compiler must be fast enough that building is not a chore, because a slow compiler slows down every chapter that follows it. The smallest compiler is small, three stages and no cleverness, so it compiles quickly. The exact compile times, how long the host compiler takes on a real program on the target machine, are measurements, and they belong to Chapter 31, on the real PC. This chapter claims no timings, because none were measured, and Chapter 3 forbids numbers without a method. What this chapter designs is a compiler small enough to be fast, and Chapter 31 measures whether it is.

One honest note about the first programs. The first Riza programs are written in an ordinary text editor, the kind already on the Mint PC, and compiled by the host compiler from the command line. The Riza editor from Chapter 8 comes later, and it uses the same parser the host compiler uses, the way section 11.1 described. The first programs are small, because the language is small at first: the hello loop, the branch, the add function. They are small on purpose. A small program the compiler builds correctly is worth more than a large program it builds wrong, because the small one proves the compiler works and the large one only proves the programmer is hopeful. The programs grow as the compiler grows, and the tests in section 11.3 grow with them.

## 11.3 Tests for every opcode

A compiler that is never tested is a compiler nobody should trust, and a compiler every program passes through is a compiler that must be trusted. So the tests are not an afterthought. They are built alongside the compiler, not after it, and this section names them: one test for every operation, one test for every refusal, one test for every validity check, and one test for every round trip the book has counted.

Why alongside, not after, is worth one paragraph, because it is the difference between a test and a rubber stamp. A test written after the compiler is written to match what the compiler does, because the writer already knows what the compiler does. A test written from the design, before or beside the compiler, checks what the compiler should do, because the writer knows the design and not yet the implementation. The tests in this section come from the design: from the operation list in section 10.2, from the refusals in Chapters 5, 6, and 7, from the validity checks in section 9.4, and from the round trips in section 8.2. The compiler is built to pass them. A test the compiler fails is a bug in the compiler, not a test to be rewritten, unless the design itself was wrong, in which case the design changes in the open, the way Chapter 3 requires.

The first kind is the operation tests: one test for every operation the book has named. Each test is a small program that uses the operation, compiled by the bootstrap compiler, run, and checked against the expected result by hand. The test passes when the program does what the dictionary says the operation does, and the result is the result the reader computed in advance. Count the operations from the six groups in section 10.2, plus the end:

| Group | Operations | Count |
| --- | --- | --- |
| Loads | LOAD_CONST, LOAD, LOAD_STR, LOAD_DATA | 4 |
| Stores | STORE | 1 |
| Arithmetic and comparison | ADD, SUBTRACT, MULTIPLY, DIVIDE, LESS_THAN, EQUAL | 6 |
| Jumps | JUMP, JUMP_IF_FALSE | 2 |
| Calls and returns | CALL, RETURN | 2 |
| World-touching | PRINT | 1 |
| End | END | 1 |
| Total | | 17 |

Check the addition: 4 plus 1 is 5, plus 6 is 11, plus 2 is 13, plus 2 is 15, plus 1 is 16, plus 1 is 17. Seventeen operations named so far, and seventeen tests, one each. The count is a floor, not a ceiling: the full operation list lives in Appendix A, and every operation added to it gets a test when it is added.

Work through a few of the seventeen, so the reader can see what a test looks like. The ADD test is a program that adds 2 and 3 and prints the result. The expected result is 5, computed by hand before the test runs. The test passes when the program prints 5. The CALL and RETURN tests use the add function from section 8.1: a program that calls add(3, 4) and prints the result. The expected result is 7. The PRINT test is the line from section 8.2: `print("hello")`, and the expected result is the word `hello` on the screen. The jump tests use the programs the book already counted: the hello loop, which must print `hello` three times and stop, and the branch from section 7.2, which must print `done` and skip `too young` when the age is 20.

One honest note about the world-touching group. PRINT is the only operation in it that this book has named, and it gets a test. The rest of the group is the core library's work, which Chapter 16 defines: reading a file, writing a file, asking the time. The middle form has one way to call the library, the way section 10.2 said, and the library call gets its test when Chapter 16 names the routines it calls. A test for a routine that has not been named yet would be a test for a guess, and Chapter 3 forbids guessing.

The second kind is the refusal tests: tests for the checks in stage one. Each test is a program the compiler must refuse, and the test passes when the compiler refuses it, with a plain-words error naming the line. The refusals are tested too, because a refusal that never fires is no refusal. A compiler that accepts a program it should refuse has a hole in it, and the hole is found by testing the refusal, not by hoping it works.

There are seven refusal tests, one for each refusal the book has promised:

| Refusal | The test program | Where the rule comes from |
| --- | --- | --- |
| Type error | Adds a number to a string | Chapter 5 |
| Unchecked error | Uses a result without checking for the error | Section 7.3 |
| No owner | Makes a value with no owner | Section 6.1 |
| Double free | Frees a value twice | Section 6.1 |
| Use after free | Uses a value after its owner freed it | Section 6.1 |
| Wrong kind to a function | Hands a function the wrong kind of value | Section 7.1 |
| Wrong number to a function | Hands a function the wrong number of values | Section 7.1 |

Seven tests, and each one must produce a refusal, in plain words, naming the line. A test that produces a file instead of a refusal is a test that failed, and the compiler goes back. A test that produces a refusal naming the wrong line is a test that failed too, because an error shown somewhere else is an error the programmer has to hunt for, the way section 8.3 said.

The third kind is the validity tests: one test for each check in section 9.4. Each test is a malformed file, built by hand, that the loader must refuse at the door. The compiler's tests and the loader's tests meet here, because a file the compiler writes must pass every check, and a file that fails any check must be refused. Six checks, six tests:

| Check | The malformed file |
| --- | --- |
| Magic | First four bytes are not R I Z A |
| Version | Version byte names a dictionary the loader does not know |
| Sizes | Header sizes add up to a number that is not the file's size |
| Bounds | A section runs past the end of the file |
| References | A jump target outside the code, or a name number past the end of the table |
| Value shapes | A byte holding 7 where a true or false is expected |

Six tests, and each one must produce a clean error in plain words, with nothing half run, the way section 9.4 designed it. The compiler's own output must pass all six checks on every program it builds, so the validity tests also test the compiler: a file the compiler writes that fails a check is a file the compiler wrote wrong.

The fourth kind is the round-trip tests, from section 8.2. Each test takes a program the book has already counted, compiles it, generates the text back from the binary, and compares the two texts, byte for byte. The test passes when the text that comes out is the text that went in, character for character. Three programs, three tests: `print("hello")`, 15 bytes; the hello loop, 68 bytes; the branch, 59 bytes. A mismatch is a failure, reported as a failure, the way rule 7 of section 1.5 demands, because a text view that says one thing and a binary that does another is a broken generator or a broken parser, the way section 8.2 said.

Now the total. Seventeen operation tests, seven refusal tests, six validity tests, three round-trip tests. 17 plus 7 is 24, plus 6 is 30, plus 3 is 33. The smallest suite has 33 tests, and 33 is the floor. The count grows as the language grows: every new operation in Appendix A gets an operation test, every new grammar rule gets a refusal test and a round-trip test, and every new validity check gets a validity test. A suite that does not grow with the language is a suite that stops covering it.

The tests run on the target PC, the way section 2.5 demands: nothing counts without a real number from real hardware, and a test that passes elsewhere and fails on the target machine has proved nothing about the target machine. The tests run fast, because a slow suite is a suite nobody runs, and a suite nobody runs is a suite that might as well not exist. Every change to the compiler runs the full suite, and a change that breaks a test is a change that goes back, the way section 3.3 said a chapter that fails the test goes back. The tester in Chapter 20 runs only what changed, so the suite stays fast as it grows: a change to one operation runs the tests for that operation, and the full suite runs when the change touches something every test depends on.

One test deserves a paragraph of its own, because it ties the whole chapter together. The first end-to-end test compiles the hello loop and compares the compiler's output with the file section 9.2 counted by hand: 72 bytes, header and all, every byte in its place. The hand count is the expected output, and the compiler must produce exactly that file, byte for byte. The test passes when the two files are identical. It is the test that proves the compiler writes what the book designed, and it is the test the whole chapter is building toward: text in, file out, and the file is the one the reader already counted.

The compiler is designed now: the smallest version that works, three stages and no cleverness; built first in Python, a ladder that comes down in Chapter 13; and tested, one test for every operation, every refusal, every validity check, and every round trip, 33 tests to start, growing with the language. The next chapter builds the optimizer, which only does what pays, on the same middle form the compiler writes.

---

The bootstrap compiler is the first tool that writes the middle form and the file: three stages, read and check the text, produce the middle form, write the file, with the checks from Chapters 5, 6, and 7 enforced at build time and one parser shared by the editor and the compiler, so the two can never disagree. It is built first in Python 3.12, already on the target machine, a ladder that exists to produce the first Riza programs and the first Riza compiler, and comes down in Chapter 13. And it is tested from the start: 17 operation tests, 7 refusal tests, 6 validity tests, and 3 round-trip tests, 33 in all, with the hello loop's hand-counted 72 byte file as the first end-to-end expected output. The compiler is designed. Chapter 12 optimizes what it writes.
