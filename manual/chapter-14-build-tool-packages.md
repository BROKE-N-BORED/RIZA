# Chapter 14: Build tool and packages

Chapter 9 defined the file. Chapter 10 defined the form. Chapter 11 built the compiler. Chapter 12 optimized what it writes. Chapter 13 took the ladder down. This chapter is the last chapter of Part 3, and it designs the tool that runs them all with one command, and the package that carries a program from one machine to another.

Building a program, as the last five chapters designed it, is a sequence of steps: run the compiler, run the optimizer, run the tests, write the file. A sequence the programmer must remember is a sequence the programmer will get wrong, skip, or run in the wrong order. Section 13.2 said the next chapter builds the build tool, which runs the Riza compiler with one command, so that building a program is one act, not a sequence of steps the programmer must remember. This is that chapter, and this is that tool.

And a program that is built must travel. A program written on one machine must run on another, and the other machine needs more than the program file: its name, its version, and what it needs to run. The package is that file: the program plus its description, carried from one machine to another the way the system itself travels, as a file, with no server in between.

There are two sections. The first names the build tool: one command builds, rebuilding only what changed, failing cleanly, and reporting what it did in plain words. The second names the package format and the versions: what a package holds, how versions are numbered, and what happens when a package needs something the machine does not have.

One warning, stated up front the way Chapters 4 through 13 stated their own. This chapter designs the build tool and the package format. It does not build either one. No build tool exists yet, no package exists yet, and no claim in this chapter says either does. The chapter claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own examples, counted by hand.

## 14.1 One command builds

Start with the job, in one sentence. The build tool runs the compiler with one command. The programmer types `build`, and the tool does the rest: it checks what changed, runs the compiler on the changed files, runs the optimizer, runs the tests, writes the file, and reports what it did. One command, one act.

The tool is a Riza program, built by the Riza compiler, like every other tool in the system. It runs the Riza compiler, not the Python one, the way section 13.1 said: the ladder is down, and every build is Riza building Riza.

A program is one or more source files. So far the book's examples have been one file each: the hello loop, the branch, the print line. A real program is bigger, and it is split into files, one file per part, so the programmer can find each part and work on it alone. The build tool is what puts the files together. The compiler compiles one source file at a time, through stages one and two from section 11.1: read and check the text, produce the middle form. The build tool joins the middle forms into one, runs the optimizer on the joined form, and runs stage three, which writes the file. The joining is work the reader can follow: the code sections go in order, the main file first, then the rest, with the positions shifted so each operation lands where it belongs; the name tables are merged into one, with the entries renumbered and the name numbers in the code adjusted to match; and the function mapping is built from the joined form, the way section 9.2 defined it. One file in the project is the main file, named when the project is started, and its code runs first, at entry point 0.

How the tool knows what changed is the heart of the design, because rebuilding only what changed is what makes the build fast. Section 3.1 set the limit: a compile that takes an hour is a compile nobody runs twice, and a build that recompiles every file every time grows slow as the program grows. So the tool keeps a record, a small file in the project, with one entry per source file: the file's name, its size in bytes, the time the filesystem says it was last written, and the middle form the compiler produced from it last time. On a new build, the tool checks each source file against its entry. A file whose size or last-written time differs from the record has changed, and it is rebuilt. A file whose size and time match is unchanged, and its middle form is reused from the record, without reading the text again.

Say plainly what the check trusts, because the trust is the limit. The tool trusts the filesystem's record of when a file was last written, and the file's size as a second check: an edit that changes the size is caught even if the time is wrong. An edit that changes neither the size nor the time would be missed, and the tool would reuse the old middle form for a changed file. That is a real limit, stated here in the open, and the answer is a second command: the programmer types `build all`, and the tool throws the record away and rebuilds every file from scratch. And when the tool does not know whether a file changed, because the record is missing or unreadable, it rebuilds the file. When in doubt, the tool builds, because a rebuild that was not needed costs a little time, and a skipped rebuild that was needed costs a wrong program.

A change to one file can mean rebuilding another. A file that uses a function from another file depends on it: if the function's file changes, the user must be rebuilt too, because the call must be checked against the new definition. The record holds, for each file, the other files it uses, as the compiler reported them. So the tool rebuilds a changed file and every file that uses it, and nothing else. In the worked example below, `main` uses `add`, so a change to `add` would rebuild both files, and a change to `main` rebuilds only `main`.

The order of the steps is the design, so it is stated in full.

First, check what changed, against the record, the way this section described.

Second, compile the changed files, and only the changed files, through stages one and two. The unchanged files contribute their middle form from the record. The compiler checks each changed file the way section 11.1 described: types, ownership, errors, calls, names. A call in one file to a function defined in another is checked against the definition the record holds, so a wrong call across files is refused the same way a wrong call inside one file would be.

Third, join the middle forms into one, the way this section described, and run the optimizer on the joined form, the way Chapter 12 designed it: dead code, constants, inlining, only what pays. The optimizer runs on the whole joined form every build, changed or not, because the form is small and the optimizer is cheap, and the optimized output is always the optimizer's view of the whole program, not a patchwork of old and new passes.

Fourth, write the new file under a temporary name. The file is complete, in the shape Chapter 9 defined, but it does not yet replace the last good build. It waits for the tests.

Fifth, run the tests that belong to the project. For an ordinary program, they are the program's own tests: small programs that check the built file's output against results computed by hand. For the compiler itself, they are the 34-test suite from section 13.2, including the bootstrap test, which runs on every change to the compiler, without exception, because the compiler is the one program every other program passes through. Each test runs at the stage it belongs to, the way sections 12.1 and 13.2 described: the round-trip tests run on the compiler's middle form, before the optimizer, and the validity tests run on the file the tool just wrote. The tester in Chapter 20 runs only what changed, the same way the build tool builds only what changed.

Sixth, move the new file into place, and only then. If every test passed, the temporary file becomes the program file, replacing the last good build, and the record is updated with the new sizes, times, and middle forms. If any step failed, the temporary file is deleted, the last good build stays in place, untouched, and the record is not updated.

Seventh, report. The tool reports what it did, in plain words: what it built, what it skipped because nothing changed, what failed and where. The report is the answer to the test in section 2.4: can the user see what the tool did? A build that says nothing is a build the programmer cannot trust, because the programmer cannot tell whether it built the new code or the old code.

It does not fetch anything from the network on its own. Section 3.2 says the system works with no network at all, and the build tool is part of the system. A build that needs a download stops working when the network does.

It does not run in the background. Section 2.4 forbids hidden work, and a build tool that rebuilt on its own schedule would be work the user did not ask for, on a schedule the user cannot see. It runs when the programmer types the command, and only then.

It does not change the source files. It reads the text and never writes it, so the round trip from section 8.2 is untouched: the text the programmer wrote is the text the editor shows, before the build and after it.

It does not guess. If the record is missing, it rebuilds everything. If a file's state is unclear, it rebuilds the file. If a test fails, it stops and says so. A tool that guesses is a tool the programmer cannot trust, and the build tool is the tool every program passes through.

Now work one small build through, step by step, the way sections 4.4 and 7.2 worked their examples, with every byte counted. The program is made of two source files: `add`, the add function from section 8.1, and `main`, which calls it and prints the answer. The program is called `adder`, and `main` is the main file.

First, the text, with the byte count of each line, newlines included the way section 4.4 counted them.

| File | Line | Text | Bytes |
| --- | --- | --- | --- |
| add | 1 | `define add(first, second):` | 26 characters + 1 newline = 27 |
| add | 2 | `    total = first + second` | 26 characters + 1 newline = 27 |
| add | 3 | `    return total` | 16 characters + 1 newline = 17 |
| main | 1 | `answer = add(3, 4)` | 18 characters + 1 newline = 19 |
| main | 2 | `print(answer)` | 13 characters + 1 newline = 14 |
| Total | | | 104 bytes |

The `add` file is 71 bytes. The `main` file is 33 bytes. The project is 104 bytes of text. The program prints 7: add of 3 and 4 is 7, and print of answer prints 7.

The programmer types `build`. There is no record yet, because this is the first build, so both files are new and both are compiled.

The compiler reads `add`, 71 bytes, checks it, and produces the function body: 6 operations, 10 bytes of code, the same body section 12.1 counted. It reads `main`, 33 bytes, checks it, and produces the main program: 7 operations, 12 bytes of code. The call to `add` is checked against the definition in the `add` file: two numbers in, one number out, and the call hands it 3 and 4, so the check passes.

The tool joins the two middle forms, main first, then the body. The joined code is 13 operations, 22 bytes. The name table is merged: add 0, first 1, second 2, total 3, answer 4, 31 bytes. The code uses the illustrative operation values from sections 4.2 and 7.1, not the final codes, which come from measured frequency and live in Appendix A.

| Position | Bytes | Meaning |
| --- | --- | --- |
| 0 | `10 03` | LOAD_CONST 3: put the number 3 on the pile |
| 2 | `10 04` | LOAD_CONST 4: put the number 4 on the pile |
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

Check the positions. The main program is 7 operations, 12 bytes, positions 0 to 11. The function body is 6 operations, 10 bytes, positions 12 to 21. The code is 22 bytes in total, 13 operations.

The name table holds the five names, in the shape section 4.3 defined: add 0, first 1, second 2, total 3, answer 4. The table is 1 byte for the count, plus 4 plus 6 plus 7 plus 6 plus 7 for the entries: 31 bytes in total. The plain file, in the shape Chapter 9 defined, would be 33 bytes of header, 22 of code, 0 of data, and a dictionary section of 38 bytes, the 31 byte name table plus a 6 byte function mapping plus a 1 byte comments count: 93 bytes in total. But the build does not write the plain file. It runs the optimizer first.

The optimizer folds the program, the way section 12.1 folded its example. Add is called in exactly one place, so it is inlined. The constants 3 and 4 are folded to 7. The 7 is carried to the print, and the stores of total and answer go dead, with the loads that fed them. The program settles at three operations: LOAD_CONST 7, PRINT, END, 4 bytes of code, and an empty name table, 1 byte, the count saying 0. The file is 33 bytes of header, 4 of code, 0 of data, and a dictionary section of 3 bytes, the three counts saying 0: 40 bytes in total.

The tool writes the 40 byte file under a temporary name, and runs the program's test: the test runs the file and checks the output. The program prints 7, as expected. 1 passed, 0 failed. The tool moves the file into place as `adder`, and writes the record: `add`, 71 bytes, its last-written time, and its 6 operations; `main`, 33 bytes, its last-written time, and its 7 operations.

The report, in plain words, reads like this:

```
Checked 2 source files. Both are new.
Compiled add: 71 bytes of text, 6 operations, 10 bytes of code.
Compiled main: 33 bytes of text, 7 operations, 12 bytes of code.
Optimized: 13 operations to 3, 22 bytes of code to 4.
Tests: 1 passed, 0 failed. The program printed 7, as expected.
Wrote adder: 40 bytes.
```

Now the second build, because the second build is where the tool earns its keep. The programmer changes one line in `main`: `add(3, 4)` becomes `add(5, 6)`. The line is still 19 bytes, so the file is still 33 bytes, but the last-written time is new. The programmer types `build`.

The tool checks the record. `add` is unchanged: same size, 71 bytes, same last-written time. It is skipped, and its middle form is reused from the record, without reading the text again. `main` has changed: the last-written time is new. It is rebuilt. The tool reads 33 bytes of text instead of 104.

The compiler reads the new `main`, checks it, and produces 7 operations, 12 bytes of code, with the new constants: LOAD_CONST 5 and LOAD_CONST 6 where the old form had 3 and 4. The tool joins the new main with the stored add body: 13 operations, 22 bytes of code, the same shape as before. The optimizer folds it the same way: inlined, folded to 11, carried to the print, settled at three operations: LOAD_CONST 11, PRINT, END, 4 bytes of code. The file is 40 bytes, the same size as the last build.

The tool writes the new file under a temporary name, and runs the test: the program prints 11, as expected. 1 passed, 0 failed. Before moving the file into place, the tool compares the new file with the last build, byte for byte, the same comparison section 13.2 used for the bootstrap test. The two files differ in exactly one byte: the constant, at file position 34, which holds 7 in the old file and 11 in the new one. The tool moves the file into place, updates the record, and reports:

```
Checked 2 source files.
Compiled main: 33 bytes of text, 7 operations, 12 bytes of code.
Skipped add: unchanged since the last build.
Optimized: 13 operations to 3, 22 bytes of code to 4.
Tests: 1 passed, 0 failed. The program printed 11, as expected.
Wrote adder: 40 bytes. The new file differs from the last build in 1 byte.
```

Read the second build honestly, because it shows both the saving and its limit. The saving is real: one file read instead of two, one file compiled instead of two, 33 bytes read instead of 104, and the report says so. But the saving is in the reading and the checking, not in the whole build: the optimizer still runs on the whole joined form, the tests still run, and the file is still written. On a program of two small files, the saving is small. On a program of fifty files, where one line changed in one of them, the saving is the difference between a build that takes a moment and a build that takes a chore. The tool does not claim a time saving it has not measured. It claims a work saving it can count, and the count is above.

One more honest note about the worked example. The program folds completely, twice, because it has no inputs: every value is known at build time, so the optimizer can prove the whole program is just printing a number. A real program reads files, takes input, and loops on data, and most of it does not fold. The example shows the build tool's mechanism, the checking, the skipping, the joining, the reporting, not a promise that every program shrinks to 3 operations. The hello loop is the other side of the same point: the optimizer changes nothing in it, and the file is the 72 bytes section 9.2 counted.

The build tool is designed now. The next section designs the package: the file that carries a built program from one machine to another.

## 14.2 Package format and versions

A package is a program plus its description. The program is the file Chapter 9 defined, and Appendix B holds its full formal spec. The description names it, versions it, and lists what it needs. The two travel together in one file, so a program carried from one machine to another arrives with everything the other machine needs to know, and nothing it has to guess.

The package file has three parts, in this order: a magic, the description, and the program file.

The magic is 4 bytes, the same shape as the program magic in section 9.1: the letters R I Z P, stored as four bytes, `52 49 5A 50` in hexadecimal, which is R is 82, I is 73, Z is 90, P is 80 in ordinary counting, from the ASCII table Chapter 5 introduced. The P says package. The tool reads the first four bytes of a file it is asked to install, and if they are not R I Z P, it knows at once that this is not a package, and it stops. A package is not a program to run: the loader expects R I Z A and would refuse it. The install tool is the one that reads a package.

The description has three fields, in this order: the name, the version, and the dependencies.

The name is the package's name, stored the way a name table entry is stored in section 4.3: one byte saying how many characters, followed by the characters. The name is the identity: the name the install tool records, the name the user asks for, and the name other packages list when they need it. Two packages with the same name are two versions of the same package, and the version says which is which.

The version is three numbers: major, minor, patch, one byte each, 3 bytes in total. Each part counts from 0 to 255, because one byte holds 256 values, from section 5.1. That is the limit, stated here: a version part that reaches 255 has no room to grow, and the format would change in the open, the way section 9.2 said the name table limit would be handled. The three parts mean different things, and the meaning is the rule for compatibility:

Major changes when the package breaks something that used it. A function removed, a function's promise changed, a name that used to work and no longer does. A major change is a breaking change, and a program built against the old major version is not promised to work with the new one.

Minor changes when new work is added that breaks nothing. A new function, a new name, a new capability, with everything that already worked still working. A minor change is safe to take.

Patch changes when a mistake is fixed and nothing else changes. The program does the same work, the same way, with a bug removed.

The dependencies are the packages this package needs, and the versions it works with. The description holds a count byte, saying how many dependencies there are, then one entry per dependency. Each entry is a name, stored the way the package's own name is stored, and a version, three bytes, the earliest version the package works with. The rule for whether an installed version satisfies a dependency is one sentence: the installed version must have the same major number as the named version, and it must be the named version or newer. Same major, because a different major breaks things. Named version or newer, because a newer minor or patch breaks nothing.

Work through the rule on one dependency, because the rule is only worth something if the reader can apply it. A package needs `greet` version 1.2.0.

| Installed version | Does it work? | Why |
| --- | --- | --- |
| 1.2.0 | Yes | The named version itself |
| 1.2.1 | Yes | Same major, a fix on top of the named version |
| 1.3.0 | Yes | Same major, new work added, nothing broken |
| 1.1.9 | No | Older than the named version |
| 2.0.0 | No | Different major, breaking changes |

The comparison is part by part: major first, and it must be equal; then minor, and the installed minor must be at least the named minor; then patch, and if the minors are equal, the installed patch must be at least the named patch. A version of 1.3.0 passes a need for 1.2.5, because the minor is higher, and the patch does not matter once the minor is higher.

Now work one package through, the way sections 4.4 and 9.2 worked their examples, with every byte counted. The program is the hello loop from section 4.4, as a package named `hello`, version 1.0.0, with no dependencies, because the loop needs nothing but the core library, which is part of the system, not a package.

| Part | Bytes |
| --- | --- |
| Magic, R I Z P | 4 |
| Description: name `hello`, length 5 plus 5 characters | 6 |
| Description: version 1.0.0, three bytes | 3 |
| Description: dependencies, count 0 | 1 |
| Program file, the hello loop from section 9.2 | 72 |
| Total | 86 |

Check the total: 4 plus 6 is 10, plus 3 is 13, plus 1 is 14, plus 72 is 86. The package is 86 bytes. The description is 10 bytes: 6 for the name, 3 for the version, 1 for the dependency count saying 0. The program file is the 72 bytes section 9.2 counted, unchanged, with its own R I Z A magic at its start. The package adds 14 bytes to the program.

A package with a dependency adds one entry per dependency. A dependency on `greet` version 1.2.0 costs 9 bytes: the name, 1 byte for the length plus 5 characters, is 6 bytes, and the version is 3 bytes. 6 plus 3 is 9. The count byte says how many entries follow, so the reader knows where the description ends and the program begins.

Installing a package is the tool's job, and it is careful on purpose, because installing is the act of trusting a file enough to put it on the machine. The steps are in order.

First, read the package magic. If the first four bytes are not R I Z P, the file is not a package, and the tool stops and says so.

Second, read the description: the name, the version, and the dependencies. The tool checks the dependencies against what is already installed: for each one, it asks whether the installed version satisfies the rule, same major and at least the named version. If a dependency is missing, or the installed version is too old, or the major number differs, the tool stops, says so plainly, naming the missing version, and installs nothing. It does not guess, and it does not fetch without being asked. A package that needs `greet` 1.2.0, on a machine that has `greet` 1.1.0, gets this report: this package needs greet 1.2.0 or later in the 1 series, and the machine has greet 1.1.0. Nothing is installed.

Third, check the program file inside the package, with the six validity checks from section 9.4, in the same order: magic, version, sizes, bounds, references, value shapes. A package whose program fails any check is a broken package, and the tool installs nothing, reporting the failure in plain words, the way section 9.4 designed it. The checks run before anything is placed on the machine, so a broken program never reaches the disk.

Fourth, place the program file on the machine, as the plain Chapter 9 file, 72 bytes for the hello loop, and record the description: the name, the version, and where the file lives. The record is the list of installed packages, one entry per package, and it is what the tool consults when another package lists a dependency.

No network is needed to install a package the user already has. The package file is on the machine, the checks run on the machine, and the program is placed on the machine. Section 3.2 says the system works with no network at all, and installing is part of the system.

There is no package registry that needs the network. A registry is a central list of packages, on a server, that the tool must ask before it can install anything. Riza has none. Packages travel as files, copied from one machine to another, the way the system itself travels: on a disk, on a stick, over a shared folder, or over the network if the user asks. Section 3.2 lists fetching a package as a use the network can serve, when the user asks for it, and that is the rule here: the tool fetches a package only when the user asks, and only from a place the user names. It never fetches on its own, it never checks for updates on its own schedule, and it never phones home, because section 2.4 forbids hidden work and a background update check is work on a schedule the user cannot see. The user says where to get a package, or carries the file over by hand, and the tool installs it the same way, checked first.

Updating a package follows the same careful order. A newer version of an installed package is a new package file, with the same name and a higher version. The tool checks the new package the same way: dependencies, then the six validity checks, and only when the new package passes does the old version move aside, so a failed update leaves the old version running, not a half-installed new one. That is the same clean-failure rule as the build tool's. Chapter 33 designs updates that cannot brick the machine, and this chapter's rule is the package-level part of it: check first, then replace, never the other way around.

The chapter counts as done when the build tool builds the three counted programs, `print("hello")`, the hello loop, and the branch, and the package format holds each of them, on the target PC. That is part 5 of section 3.3: anything the book says runs on the target machine has been run there, or is marked plainly as waiting for the real-PC run. Until then, the build tool and the package format are designs waiting for their proof, and the book says so.

Part 3 ends here. Chapter 9 defined the file. Chapter 10 defined the form. Chapter 11 built the compiler. Chapter 12 optimized what it writes. Chapter 13 took the ladder down. Chapter 14 built the tool that runs them all with one command, and the package that carries a program from one machine to another. The binary and the compiler are designed, from the file format to the package. Chapter 30 builds the full system image from programs installed as packages, so the package format is the unit the system is built from. Part 4 builds the runtime and the core library, starting with Chapter 15, where the file the build tool writes is started, and what runs before the program's first act is designed.

---

The build tool and the packages close Part 3. One command builds: the tool checks what changed against its record, compiles only the changed files and anything that depends on them, joins the middle forms, runs the optimizer, writes the new file under a temporary name, runs the tests, and moves the file into place only when everything passes, reporting what it built and what it skipped in plain words. A program of two files worked through by hand shows the saving: the second build reads 33 bytes of text instead of 104, compiles one file instead of two, and the new file differs from the last build in exactly one byte, the constant. A package is the program file plus a small description, name, version, and dependencies, under its own magic R I Z P: the hello loop as a package is 86 bytes, 14 bytes of package around a 72 byte program. Versions are three plain numbers, major, minor, patch, and a dependency is satisfied by the named version or newer in the same major series. Installing checks the program with the six validity checks before placing it, needs no network, and installs nothing when a dependency is missing. The binary and the compiler are designed. Part 4 builds the runtime.
