# Chapter 20: Developer tools

Chapter 18 built the file tools, which handle files whole and search inside them: list, copy, move, delete, and search. Chapter 19 built the text tools, which show, change, and compare the text inside a file: view, edit, and compare, and the dense form for text. This chapter builds the tools the programmer runs on programs: the formatter and the checker, and the tester. Part 5 continues here, with the tools the user runs every day, written in Riza, on the runtime Parts 3 and 4 designed.

The programmer writes a program, and four tools help: the formatter, which holds the text to the canonical form; the checker, which runs the compiler's checks without building; the tester, which runs the tests that could have changed; and the debugger, which reads the traces Chapter 17 designed. Chapter 17 promised this chapter would build the developer tools, including the debugger, and this chapter names all four. The roadmap gives this chapter two subsections: the first covers the formatter and the checker, and the second covers the tester. The debugger is named here, in the framing, because Chapter 17 designed the records it reads, the program file, the map file, and the trace file, and this chapter names the tool that reads them: the debugger reads those three files and shows the programmer the text view with the values beside the lines, the way section 17.2 designed it. This chapter does not redesign the debugger. Chapter 17 designed what it reads. This chapter says where it lives: among the developer tools, alongside the formatter, the checker, and the tester.

There are two sections. The first designs the formatter and the checker: one tool that reshapes the text, one that checks it, neither of which builds the program. The second designs the tester, which runs only the tests that could have changed.

One warning, stated up front the way Chapters 4 through 19 stated their own. This chapter designs the developer tools. It does not build any of them. No tool exists yet, and no claim in this chapter says one does. The chapter also claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own examples, counted by hand. No time is claimed anywhere in this chapter, because no tool has been measured on the target machine, and Chapter 3 forbids numbers without a method. The measured proof comes in Chapter 31, on the real PC.

## 20.1 Formatter and checker

Start with the job, in one sentence. The formatter holds the programmer's text to the canonical form, and the checker runs the compiler's checks without building the program.

The formatter first. Section 8.2 designed the canonical form: the form the fixed rule produces, the same spacing, the same four space indentation, one newline per line. Two programs that mean the same thing look the same on the page, because the same rule shaped them both. The editor holds the text to that form as the programmer works, the way section 8.3 said, and the formatter is the tool that does it. The formatter is also a tool the programmer can run on its own, with the `format` command, naming the file.

The formatter works in five steps, in order.

First, read the file. The formatter reads the program's text with read_lines from section 16.1, which opens the file, reads its lines into a list, and closes it, in one call. If the answer is an error, code 1 or code 2, the formatter reports it and stops.

Second, parse the text. The formatter parses with the shared parser from section 2.2, the one parser, maintained once, used by every tool: the editor uses it while the programmer writes, the compiler uses it at build time, the way section 11.1 described, and the formatter uses it here. Because it is the same parser, the three can never disagree about what a program means. If the text does not parse, the formatter reports the error, with the line number and the reason in plain words, and changes nothing.

Third, generate the canonical text from the tree, by the fixed rule from section 8.2. It is the same rule the generator uses to produce text from the binary, so the formatter's output is the text the system would store. The spacing is the same everywhere: one space around an `=` or a `+`, one space after a comma, no space inside parentheses. The indentation is four spaces per level, the way section 8.1 designed it. Each line ends with one newline, including the last line. There are no blank lines, because the generator writes one line per step, and a blank line is not a step. A comment stays with the line it belongs to, the way section 8.2 promised the notes survive the round trip.

Fourth, compare. If the canonical text is the same as the text the formatter read, the program is already in canonical form, and the formatter reports that and stops. It writes nothing: a tool that rewrites a file it did not change is a tool doing work the user did not ask for, and section 2.4 forbids it.

Fifth, write. The formatter writes with the edit tool's shape from section 19.1: it creates a new file under a temporary name with create_file, writes each canonical line with write_line, closes the file, and moves it into the old file's place with move_file, which replaces the destination, the way section 19.1 settled it. If a write gives an error, code 3, the disk is full: the formatter closes the new file, deletes it with delete_file, and reports the failure. The original file is untouched, because it was never written to, only read. Then the formatter reports what it did: how many bytes it read, how many it wrote, and how many lines changed.

Say plainly what the formatter changes and what it never changes, because the line between them is the whole design. It changes spacing, indentation, blank lines, and the newline at the end of the file. It never changes a name, a value, the order of the lines, or a comment. It never changes what the program does, only how it looks: the middle form the compiler produces is the same before formatting and after, operation for operation, and the worked example below shows it. The costs the editor shows next to a line, the way section 5.4 promised, are unchanged too, because the program did not change.

Say plainly what the formatter does not do, because the list keeps it small. It does not optimize: the optimizer in Chapter 12 works on the middle form, and the formatter works on the text, so `2 + 3` stays `2 + 3` in the text and the optimizer folds it when the program is built. It does not check the program, and it does not build it. Each tool does one job, the way section 2.4 requires.

Now the checker. The checker runs the compiler's stage-one checks from section 11.1, without building the program. Stage one reads and checks the text: it checks the types, from Chapter 5; the ownership, from Chapter 6; the errors, from Chapter 7; the calls, from section 7.1; and the names, defined before use. They are the same checks the compiler runs, so the two can never disagree: a program the checker passes, the compiler's stage one passes, and a program the checker refuses, the compiler refuses. The editor runs the checker as the programmer writes, the way section 8.3 said, so the programmer finds out about a mistake while writing, not after building. The checker is also a tool the programmer can run on its own, with the `check` command, naming the file.

The checker works in three steps. First, read the file with read_lines, the way the formatter does. Second, parse it with the shared parser, and run the stage-one checks on the tree. Third, report. The report names each problem it found, with the line number and the reason, in plain words, in line order. If the program has no problems, the checker reports that, and writes nothing.

The checker reports every problem it finds, not just the first. The compiler stops when a check fails, because it cannot build a file from a program it refused. The checker has no file to write, so it finishes the whole list, and the programmer sees every problem at once. The checks are the same. The reporting fits the job.

The checker is fast because it does so little: no stage two, so no middle form; no stage three, so no file; no translation, which happens at load time, the way section 10.3 designed it; no optimization. It parses and checks, and that is all. No time is claimed here, because none was measured, and Chapter 31 measures the tools on the real PC.

Now work one format through, step by step, the way sections 4.4 and 7.2 worked their examples, with every byte counted. The program is `adder`, the add function from section 8.1, called once, in one file, written messily: odd spacing, wrong indentation, a blank line, and no newline at the end of the file.

First, the messy text, with the byte count of each line, newlines included the way section 4.4 counted them.

| Line | Text | Bytes |
| --- | --- | --- |
| 1 | `define add(first,second):` | 25 characters + 1 newline = 26 |
| 2 | `      total=first+second` | 24 characters + 1 newline = 25 |
| 3 | `  return total` | 14 characters + 1 newline = 15 |
| 4 | (blank) | 0 characters + 1 newline = 1 |
| 5 | `answer=add(2,3)` | 15 characters + 1 newline = 16 |
| 6 | `print( answer )` | 15 characters, no newline = 15 |
| Total | | 98 bytes |

The messy text is 98 bytes: 26 plus 25 is 51, plus 15 is 66, plus 1 is 67, plus 16 is 83, plus 15 is 98. Line 2 is indented six spaces, not four, with no spaces around the `=` or the `+`. Line 3 is indented two spaces. Line 4 is blank. Line 5 has no spaces around the `=` and no space after the comma. Line 6 has spaces inside the parentheses, and the file ends without a newline.

The formatter reads the 98 bytes, parses them, and generates the canonical text.

| Line | Text | Bytes |
| --- | --- | --- |
| 1 | `define add(first, second):` | 26 characters + 1 newline = 27 |
| 2 | `    total = first + second` | 26 characters + 1 newline = 27 |
| 3 | `    return total` | 16 characters + 1 newline = 17 |
| 4 | `answer = add(2, 3)` | 18 characters + 1 newline = 19 |
| 5 | `print(answer)` | 13 characters + 1 newline = 14 |
| Total | | 104 bytes |

The canonical text is 104 bytes: 27 plus 27 is 54, plus 17 is 71, plus 19 is 90, plus 14 is 104. The blank line is gone. The indentation is four spaces on lines 2 and 3. The spacing is one space around the `=` and the `+`, one space after each comma, no space inside the parentheses. The last line ends with a newline.

Read the result honestly, because rule 7 of section 1.5 says to report the number that loses too. The canonical text, 104 bytes, is 6 bytes bigger than the messy text, 98 bytes. The formatter is for humans, not for size: it makes the text readable, and readability costs a few bytes of spacing. The saving is in the dense form, and the dense form does not change, because the middle form does not change, which is the next count.

The middle form is the same before formatting and after. The compiler reads the messy text or the canonical text and produces the same 13 operations, 22 bytes of code, the same name table, 31 bytes, and the same file, 93 bytes. The main program comes first, at the entry point, and the function body follows it, the way section 12.1 laid its example out. The code uses the illustrative operation values from sections 4.2 and 7.1, not the final codes, which live in Appendix A.

| Position | Bytes | Meaning |
| --- | --- | --- |
| 0 | `10 02` | LOAD_CONST 2: put the number 2 on the pile |
| 2 | `10 03` | LOAD_CONST 3: put the number 3 on the pile |
| 4 | `60 00` | CALL function 0: call `add` |
| 6 | `30 04` | STORE name 4: take the top value off and store it as `answer` |
| 8 | `31 04` | LOAD name 4: put `answer` on the pile |
| 10 | `40` | PRINT: take the top value off and print it |
| 11 | `00` | END |
| 12 | `31 01` | LOAD name 1: put `first` on the pile |
| 14 | `31 02` | LOAD name 2: put `second` on the pile |
| 16 | `20` | ADD: take two values off, put their sum back |
| 17 | `30 03` | STORE name 3: take the top value off and store it as `total` |
| 19 | `31 03` | LOAD name 3: put `total` on the pile |
| 21 | `61` | RETURN: put the result on the pile and jump back to the caller |

Check the positions. The main program is 7 operations, 12 bytes, positions 0 to 11. The function body is 6 operations, 10 bytes, positions 12 to 21. The code is 22 bytes in total, 13 operations. The name table holds the five names, in the shape section 4.3 defined: add 0, first 1, second 2, total 3, answer 4, 31 bytes in total. The dense form is 22 bytes of code plus 31 bytes of table, 53 bytes. The complete file, in the shape Chapter 9 defined, is 33 bytes of header, 22 of code, 0 of data, and a dictionary section of 38 bytes, the 31 byte name table plus a 6 byte function mapping plus a 1 byte comments count: 93 bytes in total. The program prints 5: add of 2 and 3 is 5, and print of answer prints 5.

The formatter's report, in plain words, reads like this:

```
Formatted adder: read 98 bytes, wrote 104 bytes, 5 lines.
The middle form is unchanged: 13 operations, 22 bytes of code.
```

Check the counts by hand. The formatter read 98 bytes and wrote 104. It made 10 library calls: 1 to read_lines, 1 to create_file, 5 to write_line, 1 to close, 1 to move_file, 1 to print. One plus 1 is 2, plus 5 is 7, plus 1 is 8, plus 1 is 9, plus 1 is 10. The middle form it produces is the same 13 operations the messy text produced.

Now the text that does not parse, because a tool that only works on good input is a tool the programmer cannot trust. The programmer writes `answer = add(2, 3` and forgets the closing parenthesis. The formatter reads the file, parses it, and the parser reports the error: the line is missing its closing parenthesis. The formatter reports it in plain words, with the line number, and changes nothing. The file is unchanged, and the programmer fixes the line and runs the formatter again. A formatter that guessed at a broken line would be a formatter that rewrote the programmer's meaning, and the formatter never guesses.

Now the checker, worked through on two short programs with mistakes in them. The first takes the canonical add program above, and changes line 4 to hand `add` the wrong kind of value.

```
answer = add(2, "three")
```

The call on line 4 hands `add` a number and a piece of text, and `add` expects two numbers. The checker reports it:

```
line 4: add expects a number for its second value, and "three" is text.
```

The second uses a result without checking for the error.

```
lines = read_lines("notes.txt")
print(lines)
```

`read_lines` can give back an error, code 1, if the file does not exist, and the program uses `lines` on line 2 without checking whether it holds an error. Section 7.3 made the check the program's promise, and section 11.1 said a result used without the check is refused. The checker reports it:

```
line 2: lines may hold an error, and the program uses it without checking for the error first.
```

On the canonical add program from the worked example above, the checker reports no problems and writes no file, in 2 library calls: 1 to read_lines, 1 to print.

What the tools do when something goes wrong is the same shape as the file tools and the text tools: every failure is an error value with a code from section 16.3 and a message in plain words, the way section 7.3 designed errors.

| What went wrong | Code | What the tool does |
| --- | --- | --- |
| A file that does not exist | 1 | Reports the file's name and stops |
| A file that cannot be read | 2 | Reports the file's name and stops |
| A disk that is full | 3 | Reports the disk is full and stops; the formatter deletes its new file first |
| A line that runs out of memory | 4 | Reports the line is too big and stops |
| A program that does not parse | None, the report names the line | Reports the line and the reason, and changes nothing |

The tools also write to the debug log layer from section 17.1, so the record is there when something goes wrong. The log holds what the tool did and when: the format of a file, the check of a program, the problems found and their line numbers. The log holds the names and the counts, not the contents: a format logs the file and the byte counts, 98 read and 104 written, not the text, because the text might hold a secret, and section 16.3 said to log the act, never the secret.

The formatter and the checker are designed now. The next section designs the tester.

## 20.2 Tester: run only what changed

Start with the job, in one sentence. The tester runs the test suite, and it runs only the tests that could have changed since the last run.

The suite is the one section 11.3 started, 33 tests, and the chapters since have grown it: 34 after Chapter 13, 63 after Chapter 16, 74 after Chapter 17, 85 after Chapter 18, 95 after Chapter 19. Section 11.3 said why a growing suite matters: a slow suite is a suite nobody runs, and a suite nobody runs is a suite that might as well not exist. The tester is the answer section 11.3 promised: it runs only what changed, so the suite stays fast as it grows. A change to one operation runs the tests for that operation, and the full suite runs when the change touches something every test depends on.

The build tool calls the tester. Section 14.1 made running the tests step five of the build, and the tester is the tool it calls: the tool writes the new file under a temporary name, runs the tests, and moves the file into place only when they pass. The programmer can also run the tester on its own, with the `test` command, to check a change without building.

How the tester knows what changed is the heart of the design, and it follows the build tool's record from section 14.1. The tester keeps a record, a small file in the project, with one entry per test: the test's name, the files the test touches, and for each file its size in bytes and the time the filesystem says it was last written. On a new run, the tester checks each test's files against its entry: a file whose size or last-written time differs has changed, and a file whose size and time match is unchanged.

How the tester decides what to run follows from the record and from the dependencies. A test runs if any file it touches changed. A test also runs if a file that file depends on changed: a test of `main` runs when `add` changes, because `main` uses `add`, and the change to `add` can change what `main` does. The tester reads the dependency list from the build tool's record, the way section 14.1 recorded it, because the list is kept once, not twice. A test runs when the change could reach it, through a file it touches or through a dependency, and it is skipped when the change cannot reach it.

Say plainly what the record trusts, because the trust is the limit, the same limit section 14.1 stated for the build tool. The tester trusts the filesystem's record of when a file was last written, and the file's size as a second check. An edit that changes neither would be missed, and the tester would skip a test that should have run. That is a real limit, stated here in the open, and the answer is a second command: the programmer types `test all`, and the tester throws the record away and runs every test, the way `build all` rebuilds every file. When the tester does not know whether a test's files changed, because the record is missing or unreadable, it runs the test: when in doubt, the tester runs, because a run that was not needed costs a little time, and a skipped run that was needed costs a wrong pass.

Say plainly what else the tester does not do, because the list is the design. It does not run the whole suite when one file changed. It does not guess whether a change matters: a change to a comment still runs the tests, because the record sees the file changed and cannot see that the meaning did not. It does not change the source files, and it does not run in the background, because section 2.4 forbids hidden work. It runs when the programmer types the command, or when the build tool calls it, and only then.

One test has a rule of its own, and it is stated here because section 13.2 stated it as a promise. The bootstrap test runs on every change to the compiler, without exception. The bootstrap test touches the compiler's source files, so the tester's rule already runs it on any compiler change, and section 13.2 states it as a promise so the guarantee is on the record: the compiler is the one program every other program passes through, and a change to it is a change to everything downstream. And the full suite runs when the change touches something every test depends on, the way section 11.3 said: a change to the core library runs every test, because every test program calls the library's routines, and a change to the compiler runs every test, because every test program is built by it.

What the tester reports is the answer to the test in section 2.4: can the user see what the tool did? The report names which tests ran and which were skipped, which passed and which failed, and where a failure was, with the line number. A failure names the test, what was expected, what happened instead, and the line in the test file where the check failed. The tester writes the report to the debug log too, the way section 17.1 designed it: which tests ran, how many passed, how many failed, and no secrets, because a test's output might hold one, and section 16.3 said to log the act, never the secret.

A test file is a small file naming a program to run and the result it must print. The tester reads each test file with read_line from section 18.2, line by line, the way the search tool reads a file one line at a time, and learns the program and the expected result. Then it runs the program, the way the build tool runs a test in section 14.1, and compares what the program printed with the result the test names. A match passes. A miss fails, and the report names the difference.

Now work one test run through, step by step, with the tests counted. The project has three source files: `add`, the add function; `greet`, a function that prints a greeting for a name; and `main`, which uses both. The suite has five tests.

| Test | What it checks | Files it touches |
| --- | --- | --- |
| 1 | add(3, 4) gives 7 | add |
| 2 | add(0, 0) gives 0 | add |
| 3 | greet prints `hello Ana` for Ana | greet |
| 4 | greet prints `hello Bo` for Bo | greet |
| 5 | main prints the greeting and the answer | main |

The programmer changes one line in `add`: the local name `total` becomes `sum`. The work is the same; the name changes. The programmer types `test`.

The tester checks the record: `add` has changed, its last-written time is new; `greet` and `main` are unchanged, same size and same last-written time. The dependency list from the build tool's record says `main` uses `add` and `greet`, and `add` and `greet` use nothing.

The tester decides, test by test.

| Test | Runs? | Why |
| --- | --- | --- |
| 1 | Yes | It touches `add`, and `add` changed |
| 2 | Yes | It touches `add`, and `add` changed |
| 3 | No | It touches `greet`, and `greet` is unchanged |
| 4 | No | It touches `greet`, and `greet` is unchanged |
| 5 | Yes | It touches `main`, and `main` depends on `add`, which changed |

Three tests run, and two are skipped. The three that run all pass, because the rename changed a name, not the work. The report, in plain words, reads like this:

```
Ran 3 tests, skipped 2.
add(3, 4) gives 7: passed.
add(0, 0) gives 0: passed.
main prints the greeting and the answer: passed.
Skipped: greet prints hello Ana, greet prints hello Bo.
3 passed, 0 failed.
```

Check the counts by hand. The tester checked 3 source files against the record and decided 5 tests: 3 run, 2 skipped, and 3 plus 2 is 5. It read 5 test files, each 2 lines, with read_line: 3 calls per file, 2 lines and the code 7 that says the file is done, 15 calls in all. It made 30 library calls: 5 to open_file, 15 to read_line, 5 to close, 5 to print. Five plus 15 is 20, plus 5 is 25, plus 5 is 30. The saving is in the runs: 3 programs run instead of 5, and the 2 skipped tests are the ones the change could not reach.

Read the run honestly, because a single small example proves less than it might look like. The saving is 2 tests of 5, and on a suite of 5 the saving is small. On a suite of 95, where one line changed in one file, the saving is the difference between a run that takes a moment and a run that takes a chore. The tester claims a work saving it can count, and the count is above.

Now the failure run, because a tester that only reports passes is a tester the programmer cannot trust. Suppose the change to `add` was not a rename but a mistake: the programmer writes `total = first - second` where the program needs `total = first + second`. The tester runs the same 3 tests. Test 1 fails: add(3, 4) gives -1, not 7. Test 2 passes: add(0, 0) gives 0 either way. Test 5 fails: main prints -1 where it should print 7. The report names the failures with the line numbers:

```
Ran 3 tests, skipped 2.
add(3, 4) gives 7: failed. Expected 7, got -1, at line 2 of the test.
add(0, 0) gives 0: passed.
main prints the greeting and the answer: failed. Expected 7, got -1, at line 2 of the test.
Skipped: greet prints hello Ana, greet prints hello Bo.
1 passed, 2 failed.
```

Two failures, and both name the same wrong number, -1, which points at the same wrong line in `add`. The tester names the difference; finding the wrong line is the debugger's job. When a test fails, the programmer can ask for a traced run and read it in the debugger, which shows the text with the values beside the lines, the way section 17.2 designed it. The trace is made only when the programmer asks for it, the way section 17.2 designed it.

The test suite grows with this chapter. Nine new tests cover the three tools in this chapter. Three cover the formatter: the messy program formats to the canonical form, 98 bytes to 104, with the same 13 operations; a program already in canonical form is unchanged, and the formatter writes nothing; and a program that does not parse is reported and unchanged. Three cover the checker: the checker reports the wrong-kind call with the right line number; the checker reports the unchecked error with the right line number; and the checker on a clean program reports no problems and writes no file. Three cover the tester: the five-test run runs 3 tests, skips 2, and reports 3 passed; `test all` runs all 5 tests; and a test whose record is missing runs. That is 9 new tests, and the suite grows from 95 to 104: 95 plus 9 is 104. Each test runs on the target PC, the way section 2.5 demands.

This chapter adds no new routines to the core library. The tools are built on the 34 routines already named: the text routines and the file routines from section 16.1, the five Chapter 18 named, and read_dense_line from Chapter 19. The parser is shared code, the way section 2.2 designed it, not a numbered routine. The library stays at 34 routines, and the suite grows to 104 tests.

The chapter counts as done when the tools run on the target PC from section 3.1: the formatter formats the messy program to the canonical form, the checker reports the two problems with their line numbers, the tester runs the five-test suite and skips the two tests the change cannot reach, and the times are measured in Chapter 31. Until that run happens, the chapter's claims are marked plainly as waiting for the real-PC run, the way part 5 of the done test in section 3.3 requires.

Chapter 21 builds the shell, where the tools are run as commands. Chapter 31 measures the tools on the real PC, including the standard task from section 1.5: formatting a 1,000 line file. Chapter 32 benchmarks them against today's stack.

---

The developer tools are the tools the programmer runs on programs. The formatter holds the text to the canonical form section 8.2 designed, the same spacing, the same four space indentation, one newline per line, parsing with the shared parser and writing with the edit tool's shape, a new file moved into place only on success. One messy program worked through by hand shows the proof: 98 bytes become 104, reported as the growth it is, and the middle form is the same 13 operations before and after. The checker runs the compiler's stage-one checks without building, types, ownership, errors, calls, and names, and reports each problem with its line number and its reason in plain words, writing no file. The tester runs the test suite and runs only the tests that could have changed, following the build tool's record, and one run of five tests shows the saving: 3 run, 2 skipped, 3 passed. The debugger is the fourth developer tool, and it reads the program file, the map file, and the trace file Chapter 17 designed, showing the text with the values beside the lines. No new routines join the library, and the suite grows from 95 tests to 104. The tools do no hidden work, run nothing in the background, and log the act, never the secret.
