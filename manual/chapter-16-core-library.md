# Chapter 16: Core library

Chapter 15 designed the startup and the shutdown: the loader's eight steps, from the first byte of the header to the first operation of main, and the finish routine's five steps, from the flush to the exit report. This chapter designs what the program uses between those two ends: the core library, the routines every program calls. Text, lists, and files. Time, math, and random. Errors and logging. The routines are the shared work of the system, written once, called by every program, and this chapter names them, one by one, with what each one takes, what it hands back, and what it does when it fails.

A library is a set of routines a program calls instead of writing itself. Every program needs to work with text, to count, to read a file, to know the time. Without a library, every program would write those routines itself, and the same work would be written a hundred times, each copy a place for a bug to hide. With a library, the work is written once, tested once, and called by every program that needs it. One copy, shared by all, the way section 2.3 shares one form and section 4.3 stores one name.

The core library is part of the system, not a package, the way section 14.2 said: the hello loop needs nothing but the core library, and the core library is not something a program installs. The routines are reached by a call, not loaded as a lump before the program starts, the way section 15.1 designed it: the loader reads the program's file and nothing else, and a routine runs when the program calls it, and only then. A program that never reads a file never pays for the file routines. The library is small, the way section 2.4 requires, and a program pays for what it uses, when it uses it, the way section 1.4 said it should.

The program's file holds the calls, not the routines. A call is short. The middle form has one way to call the library, the way section 10.2 said, and the call names the routine by number, the way CALL names a function by number. The routine itself is native code, in the system, shared by every program. That is the opposite of the shape section 1.2 counted, where every app ships its own copy of the world.

There are three sections. The first names the routines every program uses most: text, lists, files. The second names time, math, and random: the routines that reach outside the program, for the time, for numbers the compiler cannot fold, and for chance. The third names errors and logging: the routines that make and check errors, and the routines that write the record of what the program did.

One warning, stated up front the way Chapters 4 through 15 stated their own. This chapter designs the core library. It does not build it. No routine exists yet, and no claim in this chapter says one does. The chapter also claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own examples, counted by hand. No time is claimed anywhere in this chapter, because no routine has been measured on the target machine, and Chapter 3 forbids numbers without a method. The measured proof comes in Chapter 31, on the real PC.

## 16.1 Text, lists, files

Start with the job, in one sentence. The routines in this section do the work every program does most: shape text, hold lists, and read and write files.

The text routines work on the text Chapter 5 defined: characters, stored once in the name table and called by number. Each routine hands a result back, or an error value if it cannot do its work, the way section 7.3 designed errors. The error codes are the standard codes section 16.3 names.

| Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- |
| print | A value | Nothing, it prints the value | An error, if there is nowhere to print to |
| join | Two pieces of text | One piece, the two joined | An error, code 4, if the joined text runs out of memory |
| split | Text and a mark | A list of the pieces | An error, code 4, if the list runs out of memory |
| compare | Two pieces of text | 0 if the same, below 0 if the first comes first, above 0 if it comes second | Does not fail: two pieces of text can always be compared |
| find | Text and a piece | The position of the piece, counting from 0 | An error, code 7, if the piece is not in the text |
| length | Text or a list | The number of characters, or the number of items | Does not fail |
| text_of | A number | The text of the number | An error, code 4, if the text runs out of memory |

Work through the text routines on one short example. Take the text `hello world`, 11 characters. `join("hello", " world")` gives `hello world`: 5 characters plus 6 is 11. `find("hello world", "world")` gives 6, because `world` starts at the seventh character, and positions count from 0. `length("hello world")` gives 11. And `print` is the routine the PRINT operation calls, the way section 10.3 designed it.

Say plainly how the routines handle wider text, because section 5.1 left the full text story to this chapter. Text in other languages needs more than one byte per character, because one byte holds only 256 values. The standard way to store wider text, called UTF-8, gives each character one to four bytes: plain letters take one byte, the way they always did, and wider characters take more. The name table stores bytes, the way section 4.3 defined it, so wider text works the same way, with more bytes per character. The routines work on characters, not bytes: `length` counts characters, so a wider character counts as one, and `find` gives a position counted in characters. The byte count is the size, stated in the value, the way section 5.1 requires.

The list routines work on the lists Chapter 5 defined: one run, one value after another, with a length stored with it. A list is made by writing its values between brackets, `[1, 2, 3]`, the way section 8.1 showed, and an empty list is `[]`.

| Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- |
| append | A list and a value | The list, with the value added at the end | An error, code 4, if the grown list runs out of memory |
| get | A list and a position | The value at that position, counting from 0 | An error, code 6, if the position is past the end |
| length | A list | The number of items | Does not fail |

`append([1, 2], 3)` gives `[1, 2, 3]`. `get([10, 20, 30], 1)` gives 20, because positions count from 0. `get([10, 20, 30], 5)` gives an error, code 6, because there is no sixth item. `length` is one routine that works on text and on lists alike, because both are the same idea, how many, and the type of the value says what is counted.

Growing a list is visible work, the way section 5.4 demands: when `append` finds the run full, it makes a bigger run, copies the old contents into it, and frees the old one, the way section 5.2 described. The copy is the cost of growing, and the line that grows the list says so, because the line names `append`.

The file routines work on the files Chapter 5 defined: a run of bytes with a name, treated as a value the program can read and write. `open_file` is the call section 8.1 showed, and `write`, `flush`, and `close` are the ones section 15.2 used.

| Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- |
| open_file | A name | The file | An error, code 1, if the file does not exist |
| read | An open file | The file's remaining text | An error, code 2, if the file cannot be read |
| read_lines | A name | A list of the file's lines | An error, code 1, if the file does not exist; code 2, if it cannot be read |
| write | A file and text | The number of bytes written | An error, code 3, if the disk is full |
| write_line | A file and text | The number of bytes written, text plus newline | An error, code 3, if the disk is full |
| flush | A file | Nothing; the waiting bytes go to the disk | An error, code 3, if the disk is full |
| close | A file | Nothing; the file is released | An error, code 3, if the final flush fails |

`read_lines` opens the file, reads its lines into a list, and closes it, in one call. `read` reads from a file the program already opened. `write` writes text and hands back the count: `write(out, "hello")` hands back 5. `write_line` writes the text and a newline: `write_line(out, "hello")` hands back 6, five for the word and one for the newline.

Ownership runs through all three groups, the way section 6.1 requires. A routine that makes a value hands ownership to the caller: `read_lines` makes the list of lines, and the caller owns it; `open_file` makes the file value, and the caller owns it, and closing it is the caller's job, or the finish routine's when the program ends. A routine that takes a value and gives it back hands ownership back: `append` takes the list and hands back the grown list. A routine that only reads a value borrows it: `length`, `find`, and `compare` read the text and give it back untouched. The compiler checks it before the program runs, the way section 11.1 described.

Now work one small program through, step by step, the way sections 4.4 and 7.2 worked their examples, with every byte counted. The program opens a file, reads its lines into a list, counts the words in each line, and writes the count to another file.

First, the text, with the byte count of each line, newlines included the way section 4.4 counted them.

| Line | Text | Bytes |
| --- | --- | --- |
| 1 | `lines = read_lines("notes.txt")` | 31 characters + 1 newline = 32 |
| 2 | `if lines is error:` | 18 characters + 1 newline = 19 |
| 3 | `    print(lines)` | 16 characters + 1 newline = 17 |
| 4 | `out = open_file("counts.txt")` | 29 characters + 1 newline = 30 |
| 5 | `if out is error:` | 16 characters + 1 newline = 17 |
| 6 | `    print(out)` | 14 characters + 1 newline = 15 |
| 7 | `index = 0` | 9 characters + 1 newline = 10 |
| 8 | `while index < length(lines):` | 28 characters + 1 newline = 29 |
| 9 | `    line = get(lines, index)` | 28 characters + 1 newline = 29 |
| 10 | `    words = split(line, " ")` | 28 characters + 1 newline = 29 |
| 11 | `    write_line(out, text_of(length(words)))` | 43 characters + 1 newline = 44 |
| 12 | `    index = index + 1` | 21 characters + 1 newline = 22 |
| 13 | `close(out)` | 10 characters + 1 newline = 11 |
| Total | | 304 bytes |

The program is 304 bytes of text. The two error checks are the branches section 8.1 showed: if the answer is an error, the program prints it and ends, jumping to END after the print, the way section 15.2 described, so the rest never runs with an error value.

Now the input file, line by line, with the word count of each line.

| Line | Text | Words | Bytes |
| --- | --- | --- | --- |
| 1 | `hello world` | 2 | 11 characters + 1 newline = 12 |
| 2 | `one two three` | 3 | 13 characters + 1 newline = 14 |
| 3 | `four` | 1 | 4 characters + 1 newline = 5 |
| Total | | 6 | 31 bytes |

The input file is 31 bytes: 28 characters of words and 3 newlines. It holds 6 words in all: 2 plus 3 plus 1.

Run the program. `read_lines` opens `notes.txt`, reads its lines into a list of 3 strings, and closes it. The check finds no error, so the print is skipped. `open_file` opens `counts.txt` for writing. The check finds no error. The loop runs 3 times, once per line.

On the first pass, `get` gives `hello world`, `split` gives 2 pieces, `length` gives 2, `text_of` gives `2`, and `write_line` writes `2` and a newline, 2 bytes. The second pass gives 3 pieces and `3`; the third gives 1 piece and `1`. The loop test runs a fourth time: 3 is not less than 3, so the loop ends. `close` releases `counts.txt`, and the finish routine flushes anything still waiting, the way section 15.2 designed it. The output file holds `2`, `3`, `1`, each on its own line: 2 bytes plus 2 plus 2 is 6 bytes.

Now the routine calls, counted.

| Routine | Calls | Why |
| --- | --- | --- |
| read_lines | 1 | Once, for the input file |
| open_file | 1 | Once, for the output file |
| length | 7 | 4 on `lines`, the loop test runs 4 times, and 3 on `words`, once per line |
| get | 3 | Once per line |
| split | 3 | Once per line |
| text_of | 3 | Once per line |
| write_line | 3 | Once per line |
| close | 1 | Once, for the output file |
| Total | 22 | |

The program makes 22 library calls: 1 plus 1 is 2, plus 7 is 9, plus 3 is 12, plus 3 is 15, plus 3 is 18, plus 3 is 21, plus 1 is 22. It reads 31 bytes and writes 6. Every call is named, every byte is counted.

Run the same program with the input file missing. `read_lines` gives an error, code 1, with the message naming the file. The check finds the error, so the program prints it and ends, giving up cleanly, the way section 7.3 described. No lines are read, no counts are written, and the output file is never opened. The error run makes 2 calls, `read_lines` and `print`.

Read the worked example honestly, because it shows the mechanism, not a promise. A real file has more lines than three, and the counts grow with them: the calls grow with the lines, the bytes grow with the words. The example shows how the routines fit together, the reading, the counting, the writing, and the error checks at each step, and the shape holds for a file of any size, because each line costs the same calls as the last one.

## 16.2 Time, math, random

Start with the job, in one sentence. The routines in this section reach outside the program: for the time, for numbers the compiler cannot fold, and for chance.

Time is the one the system must provide, because the program cannot know the time on its own. The time of day lives outside the program, in the machine's clock, and the only way to learn it is to ask. So the time routines ask the system, and the system answers.

| Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- |
| now | Nothing | The current time, as a number of seconds since the start of 1970 | Does not fail: the system always has a clock |
| wait | A number of seconds | Nothing; it waits that long, then goes on | An error, code 6, if asked to wait less than 0 seconds |
| elapsed | A time, from `now` | The seconds since that time | An error, code 6, if the time is after now |

`now` gives an 8-byte number: the count of seconds since the first day of 1970. The starting point is the standard one, the same count every modern machine uses. A 4-byte number counts to about 4.3 billion, from the table in section 5.1, and 4.3 billion seconds is about 136 years, so a 4-byte count of seconds runs out about 136 years after 1970, in the year 2106. An 8-byte number counts to about 18.4 quintillion seconds, which is about 584 billion years, and the count will never run out. At the start of 15 November 2026, the count was 1,794,700,800 seconds, about 1.79 billion, and any reader can check that number against a clock.

`elapsed` is how a program measures how long a piece of work took: `start = now()`, the work, then `taken = elapsed(start)`. If `start` was 1,794,700,800 and the time now is 1,794,700,805, `elapsed` gives 5, five seconds of work. Chapter 31 uses this shape to measure the routines themselves: read the time, run the routine, read the time, subtract.

The optimizer never guesses the time, the way section 12.1 said: a value read from outside is not known at build time, and the optimizer leaves it alone. `now` is read from the clock every time it is called.

The math routines start where the middle form stops. The middle form already has ADD, SUBTRACT, MULTIPLY, and DIVIDE, from section 10.2, and the same operations work on fractional numbers, because the operation carries the type, the way section 10.3 said the comparison follows the type the operation carries. The library adds the three the middle form does not have.

| Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- |
| remainder | Two numbers | What is left after dividing the first by the second | An error, code 9, if the second number is 0 |
| square_root | A number | The number that multiplied by itself gives the number | An error, code 6, if the number is below 0 |
| power | A base and a count | The base raised to the count | An error, code 5, if the answer is too big for its type |

7 divided by 3 is 2, with a remainder of 1, because 2 times 3 is 6 and 7 minus 6 is 1. The square root of 9 is 3, because 3 times 3 is 9. The square root of 2 is about 1.4142135623730951, and the word about is the point, because the exactness limit below says how close about is. 2 to the power of 8 is 256, from the table in section 5.1. 3 to the power of 4 is 81.

Dividing by zero gives an error value, code 9. This chapter fixes the code for the middle form's DIVIDE operation as well as for the library's remainder routine: a number divided by zero has no answer, and the language does not invent one, so it gives an error the program can see and handle, the way section 5.4 refuses silent overflow. The square root of a negative number gives an error, code 6, because no number multiplied by itself gives a negative answer, and this language has no imaginary numbers.

Now the fractional numbers, because section 5.1 left their exact layout to this chapter. A fractional number is stored in the standard layout, called IEEE 754, the same layout every modern machine uses. The layout holds a sign, a size, and the digits, and the number of bytes says how many digits are exact. An 8-byte fractional number holds about 15 digits exactly. A 4-byte fractional number holds about 7 digits exactly.

The limit section 5.1 named is still the limit. One tenth has no exact end in binary, the way one third has no exact end in decimal. So a program that needs exact fractions must count in whole numbers of the smallest unit: cents, not dollars. A price of 12 dollars and 99 cents is stored as 1299 cents, a whole number, exact, and the program divides by 100 only when it prints.

The random routines give the program chance: a number it cannot predict, and a choice it cannot foresee.

| Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- |
| random_between | Two numbers, a low and a high | A number between them, both ends included | An error, code 6, if the low is above the high |
| random_choice | A list | One item from the list, picked at random | An error, code 7, if the list is empty |

`random_between(1, 6)` gives 1, 2, 3, 4, 5, or 6, each as likely as the others, the way a die gives one of six faces. `random_choice(["red", "green", "blue"])` gives one of the three colours, each as likely as the others. The routines do not favour any outcome.

The seed comes from two sources. The first is the time, from `now`: the count of seconds, which is different every second, so two runs a second apart start from different seeds. The second is the machine's own noise: the exact timing of disk reads, key presses, and other events the program cannot predict, gathered by the system as the machine runs. What this chapter fixes is the rule: the seed comes from the time and from the machine, never from a fixed number, so the same program does not roll the same numbers every time it runs.

Say honestly what this randomness is for, and what it is not for. It is good enough for games, for shuffles, for sampling, and for tests that need a different input every run. A program that needs true randomness for security, for secrets and keys, is a different problem, and Chapter 29 designs it. A key made from a seed a program could guess is a key that is not a secret, and the book says so instead of letting a programmer trust a shuffle with a secret.

## 16.3 Errors and logging

Start with the job, in one sentence. The routines in this section make errors, check for them, and write the record of what the program did.

The error routines work on the errors section 7.3 designed: an error is a value with a code and a message. The routines are three, and the third is not a routine at all, which is the point.

| Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- |
| make_error | A code and a message | An error value | Does not fail: a code and a message always make an error |
| is_error | A value | True if the value is an error, false if it is not | Does not fail |
| Pass up | An error | The error, handed to the caller | Not a routine: passing an error up is returning it |

`make_error` builds the error value: 2 bytes, the code in 1 byte and the message's table number in 1 byte. The message text is stored once in the name table, the way section 4.3 stores every string, so the message costs bytes once no matter how often the error is raised. The error from section 15.2, `make_error(1, "the file notes.txt does not exist")`, holds a message of 33 characters, stored once in the table as 1 length byte plus 33 characters, 34 bytes in the table, and the error value itself is 2 bytes.

`is_error` is the check the text form writes as a branch: `if answer is error:`, from section 8.1, is this routine. The compiler refuses a program that uses a result without the check, the way section 11.1 described, so the check is not optional.

Passing an error up needs no routine, because an error is a value, and passing a value up is returning it. Section 7.3 said the error travels up the chain of calls through the same return path as any value, unchanged, until it reaches a function that can deal with it. A routine for passing up would be a name for a return, and the library does not need a name for something the language already does.

The standard error codes live here, the way section 7.3 promised: one byte covers 256 kinds of error, and the standard codes live in the core library, where Chapter 16 keeps them. Code 0 is not an error code: it means no error. Codes 1 to 127 are the system's, the failures the library itself names. Codes 128 to 255 are for programs to define their own, so a program can name its own failures without colliding with the system's. The system's codes, the ones this chapter fixes, are nine:

| Code | Meaning | Where the book named it |
| --- | --- | --- |
| 1 | File not found | Sections 7.3 and 15.2 |
| 2 | File not readable | This chapter, the file routines |
| 3 | Disk full | Section 7.3 |
| 4 | Out of memory | Section 6.4 |
| 5 | Number too big | Section 5.4 |
| 6 | Value out of range | This chapter, the list and math routines |
| 7 | Not found | This chapter, the find and choice routines |
| 8 | Wrong kind | Section 11.1, for the ones only the run can see |
| 9 | Divided by zero | This chapter, the math routines |

Each code's message is plain words naming what happened: `open_file` gives code 1 with the message naming the file that does not exist, `read` gives code 2 with the message naming the file that cannot be read. The code says which kind of failure it is. The message says what happened, and the user reads the message, not the code.

Code 8 needs one honest note, because the compiler catches most wrong-kind mistakes at build time. Section 11.1 said a call that hands a function the wrong kind of value is refused before the program runs, so most wrong kinds never reach the run. Code 8 is for the ones only the run can see: a value read from a file that turns out to be the wrong kind, a number from the user that is not a number. The compiler catches what it can see. The code covers what it cannot.

The logging routines write to the debug log, the layer section 2.4 promised and Chapter 17 builds: every program carries a debug log layer, a plain record of what it did and when.

| Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- |
| log_write | Text, one line | Nothing; the line goes on the log | An error, code 3, if the disk is full |
| log_read | Nothing | The log's lines, as a list of text, in the order they were written | Does not fail |

Each line on the log starts with the time it was written, from `now`, then the message, in plain words. The startup and the shutdown write to the log too, the way section 15.2 said: which file was started, which checks ran, how the program ended, and what the report said.

Say plainly what the log holds, and what it does not hold. It holds what the program did and when, the errors it hit, and the choices it made. It does not hold secrets, passwords, or anything the user would not want written down. The log holds what happened, not what the user typed in secret, and the rule is the programmer's, stated here so it cannot be missed: log the act, never the secret.

The log is a plain record, not a hidden trace, the way section 7.3 said: the programmer reads it the way the user reads the error itself, in plain words, in the open. The user can read the log, with `log_read` or with any tool that reads text, because the log is text, one line per entry. And the user can turn logging off, because section 2.4's test asks whether the user can turn it off. A program run with logging off writes nothing.

The layer itself is Chapter 17's work: where the log file lives, how it is kept from growing without bound, and how the traces fit the binary form. Chapter 17 builds the layer the routines write to.

The chapter counts as done when the routines run on the target PC: each routine called by a small test program, with the results checked by hand, and the times measured in Chapter 31. That is part 5 of section 3.3: anything the book says runs on the target machine has been run there, or is marked plainly as waiting for the real-PC run. Until then, the routines are designs waiting for their proof, and the book says so.

The suite grows with the routines. Section 11.3 said the library call gets its test when Chapter 16 names the routines it calls, and this chapter names 28 of them: 7 for text, 2 for lists, 7 for files, 3 for time, 3 for math, 2 for random, 2 for errors, and 2 for logging. One test per routine, plus the worked example from section 16.1 as an end-to-end test, makes 29 new tests. The suite goes from 34 tests, from section 13.2, to 63: 34 plus 29 is 63. Each test runs on the target PC, the way section 2.5 demands, and a routine that fails its test goes back, the way section 3.3 said a chapter that fails the test goes back.

Part 4 continues. Chapter 17 builds the debug log layer the logging routines write to, and the traces that fit the binary form. Chapter 18 builds the file tools on the file routines: list, copy, move, delete, search. Chapter 19 builds the text tools on the text routines: view, edit, compare. Chapter 20 builds the developer tools, and the tester runs the suite this chapter grew. Chapter 21 builds the shell, where commands are library calls, not separate programs. Chapter 29 designs the security model. Chapter 30 builds the full system image. Chapter 31 measures the routines on the real PC.

---

The core library is the shared work of the system, written once and called by every program: 28 routines, each one named, each one saying what it takes, what it hands back, and what it does when it fails. Text, lists, and files do the work every program does most, and one program worked through by hand shows them fitting together: a word counter that reads 31 bytes, makes 22 library calls, and writes 6 bytes, with an error check at each step. Time, math, and random reach outside the program, for the clock, for numbers the compiler cannot fold, and for chance, with the fractional layout fixed and the seeding rule stated. Errors and logging make the failures visible and the record plain: 9 standard error codes, a log that holds what happened and never a secret, and a layer Chapter 17 builds. The routines are designs waiting for their proof, and Chapter 31 measures them on the real PC.
