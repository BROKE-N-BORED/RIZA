# Chapter 15: Startup and shutdown

Part 1 named the waste, the idea, and the rules. Part 2 built the language. Part 3 built the binary and the compiler: Chapter 9 defined the file, Chapter 10 defined the form, Chapter 11 built the compiler, Chapter 12 optimized what it writes, Chapter 13 took the ladder down, and Chapter 14 built the tool that runs them all with one command, and the package that carries a program from one machine to another. Part 4 begins here, with the runtime and the core library, and this chapter is the first piece of the runtime: what happens when the file the build tool wrote is started, and what happens when it ends.

A program on disk is a file. A program running is a file that has been loaded, checked, placed, translated, and started. The gap between the two is startup, and startup is the cost the user pays before the program does anything useful. Section 1.4 counted that cost for today's programs, and the numbers were the worst kind of waste: a Python program that prints one line pays 28 milliseconds and 10 MB before the line prints, and a Node program that does nothing at all pays 114 milliseconds and 61 MB. The work in both cases is trivial. The startup is the program. This chapter designs startup so that a program pays for what it uses, when it uses it, the way section 1.4 said it should, and designs shutdown so that every program ends cleanly, every time, whether it ends well or ends on an error.

There are two sections. The first names what runs before main: the loader's work, in order, and what the loader does not do. The second names the finish routine: the five steps that end every program, and what the finish routine cannot do.

One warning, stated up front the way Chapters 4 through 14 stated their own. This chapter designs the startup and the shutdown. It does not build either one. No loader exists yet, no finish routine exists yet, and no claim in this chapter says either does. The chapter also claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own examples, counted by hand: reads, checks, operations, steps. No time is claimed anywhere in this chapter, because no startup or shutdown has been measured on the target machine, and Chapter 3 forbids numbers without a method. The measured proof comes in Chapter 31, on the real PC.

## 15.1 What runs before main

Start with the job, in one sentence. Before the program's first act, the loader runs, and the loader's work is the startup cost.

The program's first act is main: the first operation at the entry point, the position the header names. But main does not run first. The loader runs first. The loader reads the file, checks it, places it, translates it, sets up the memory it needs, and only then jumps to the entry point. Everything before that jump is startup, and everything before that jump is the cost this section counts.

Section 1.4 is the baseline this section designs against, so read it again before going further. Here is its table, the time and memory for a program to run and exit, measured on the machine used to write this manual, median of 5 runs with the disk cache warm:

| Program | Time to run and exit | Peak memory |
| --- | --- | --- |
| `/bin/true` (does nothing, exits) | 0.9 ms | 1.3 MB |
| C hello world (prints one line) | 1.3 ms | 1.3 MB |
| `ls --version` | 2.1 ms | 2.1 MB |
| `python3 -c "print('hi')"` | 28 ms | 10 MB |
| `python3 -c "import json"` | 35 ms | 11 MB |
| `node -e ""` (does nothing) | 114 ms | 61 MB |

Read the table the way section 1.4 read it. A program that does nothing takes about a millisecond to start, because the operating system still has to load it and hand it a process. A C program that prints one line takes 1.3 milliseconds and 1.3 MB. Python doing the same job takes about 28 milliseconds and 10 MB. Node doing literally nothing takes 114 milliseconds and 61 MB. The work in all six rows is trivial. The difference is what loads before the work begins. Python loads an 8 MB interpreter binary, then its startup files, then the parts of its 55 MB standard library that startup requires. Node loads a 121 MB binary. The principle is the one section 1.4 stated: a program should pay for what it uses, when it uses it. Today's programs pay for the whole runtime up front, before they know what the user wants. The design in this section pays for one file, and nothing else.

The loader's work is the design, so it is stated in full, in order.

First, read the header. One read, 33 bytes, the fixed size from section 9.1, and ask the filesystem for the file's size. From those 33 bytes, the loader knows the version, the entry point, and where each section sits and how big it is. It knows the file's shape before it reads the rest of it.

Second, run the first four validity checks, the ones that need only the header and the file's size. The magic, four bytes compared with R I Z A. The version, one byte compared with the dictionaries the loader knows. The sizes, one addition: the header plus the three sections must add up to the file's size exactly. The bounds, each section's offset plus its size checked against the file's size, and no two sections overlapping. Section 9.4 defined these checks and their order, cheapest first, and the loader runs them here, before it reads anything else, because a file that fails one of them is not worth reading further.

Third, read the sections. One read per section that has anything in it, in file order: code, data, dictionary. The data section is empty for many programs, and an empty section costs no read. The reads move forward through the file, the way section 9.3 described, so the disk head moves forward with them, never jumping back. This is the shape section 3.1 said the spinning drive reads fastest: long runs, few files, no hunting.

Fourth, run the last two validity checks, the ones that need the sections. The references: one forward walk of the code, operation by operation, checking every name number against the name table, every jump target against the code section and the operation boundaries, every function number against the function mapping, and the entry point against the code section. The value shapes: one forward walk of the data section, reading each type byte and checking the value that follows it fits the kind it names. Both checks walk data the loader has already read. Nothing is read twice.

Fifth, place the sections, and only now, because section 9.4 said a file that fails a check gets no memory placed. The code section goes where the program runs from. The data section goes into static memory, the place section 6.3 named for values that live for the whole run. The name table and the function mapping are held for the run, because the program looks names up by number and calls functions by number while it runs. The comments are left on disk, because they are for the text view, not for the run, and the loader does not pay for what the run does not use.

Sixth, translate the middle form to native code, inside the loader, the way section 10.3 designed it. Two passes. The first pass walks the middle form and counts how many native bytes each step becomes, using the encodings Appendix C fixes, so the translator knows where each middle-form position lands in the native code. The second pass writes the native instructions, and every jump target is looked up in the table the first pass built. The translation is the last short step, done once per load, and its cost is part of the load.

Seventh, set up the memory the program needs. The pile is set up on the machine's own stack, the way section 10.3 described, empty, ready for the program's values. The static memory is set up with the slots for the named values, one slot per name, sized to the name's type, in the static memory from section 6.3. The data section's values are already in static memory from the fifth step. The program's memory is ready before its first operation runs.

Eighth, jump to the entry point. The loader jumps to the position in the code the header named, and the first operation runs. There is no warm up, no first run that pays a parse the later runs avoid. The first run is the same as every run, because the work of understanding the program was done once, when it was written, and never repeated.

Say plainly what the loader does not do, because the list is as much a part of the design as the eight steps.

It does not parse. Section 2.2 promised that loading is reading, not re-translating, and the promise holds: the parse happened once, when the program was written, and the loader never parses. The parser lives in the editor, where a human is writing and needs it, and not in the loader, which runs every time a program starts.

It does not load anything the program does not need. It reads the program's file and nothing else. It does not read the comments. It does not load a runtime, because there is no runtime to load: the program is the program, and the library routines it calls are Chapter 16's work, reached by a call, not loaded as a lump before the program starts. It does not fetch anything, from the disk or from the network, that the file does not name.

It does no hidden work. Section 2.4 forbids it, and the loader is held to the same standard as everything else: it runs when the user starts a program, and only then. Nothing runs on a schedule the user cannot see. Nothing the user turns off comes back on its own. The loader's work is the eight steps above, visible in the open, and nothing else.

It does not check a cache. Section 9.3 said the file is the truth and there is nothing to go stale, so there is no cache to check for freshness, no timestamp to compare, no first run that pays a cost the later runs avoid. The file is read, and the file is run.

It does not need the network. Section 3.2 says the system works with no network at all, and starting a program is part of the system. Pull the network cable out, and a program still starts, because starting it needs nothing from anywhere but the disk.

Now work one startup through, step by step, the way sections 4.4 and 9.2 worked their examples, with every piece of work counted. The program is the hello loop from section 4.4, as a complete file of 72 bytes, the way section 9.2 counted it: 33 bytes of header, 24 of code, 0 of data, and a dictionary section of 15 bytes.

First, the header. One read, 33 bytes. The magic is R I Z A. The version is 1. The entry point is 0. The code is at offset 33 with size 24. The data is at offset 57 with size 0. The dictionary is at offset 57 with size 15.

Second, the first four checks. The magic matches, one comparison. The version is known, one comparison. The sizes add up: 33 plus 24 is 57, plus 0 is 57, plus 15 is 72, and the file is 72 bytes, one addition and one comparison. The bounds check: the code ends at 57, the data ends at 57, the dictionary ends at 72, and none of them runs past the end of the file or overlaps another. All four checks pass.

Third, the sections. Two reads: the code section, 24 bytes, and the dictionary section, 15 bytes. The data section is empty, so the loader reads nothing for it. Three reads in total so far, 72 bytes, the whole file, each read a long run, each moving forward.

Fourth, the last two checks. The references: one forward walk of the 14 operations. The name numbers in the code are 0 and 1, and the table has 2 entries, so both are inside. The jump targets are 23 and 4, both inside the 24 byte code section, and both on the start of an operation: 23 is END, 4 is the loop test. The entry point is 0, the first operation. The value shapes: the data section is empty, so there is nothing to check, and the check runs and finds nothing. Both checks pass.

Fifth, the placing. The code section goes where the program runs from, 24 bytes. The data section is empty, so nothing goes into static memory from it. The name table, 13 bytes, and the function mapping, 1 byte, the count saying 0, are held for the run. The comments, 1 byte, the count saying 0, stay on disk.

Sixth, the translation. The first pass walks the 14 operations and counts the native bytes each step becomes. The second pass writes the native instructions: 23 of them, the count section 10.3 reached by hand, nine of the fourteen steps translating one for one, the most any one step needing five. The optimizer in Chapter 12 can shrink the multi-instruction steps, and the fused count is 19, but the translation this section counts is the plain one, 23, because the fusion is the optimizer's work and this section counts the loader's.

Seventh, the setup. The pile is set up on the machine's own stack, empty. One slot is set up in static memory, for the name `count`, sized to the name's type. The data section is empty, so no static values are placed beyond the slot.

Eighth, the jump. The loader jumps to the entry point, position 0, and the first operation runs: LOAD_CONST 0, put the number 0 on the pile. Main has started.

Now the totals, the startup work for the hello loop:

| Work | Count |
| --- | --- |
| Reads | 3: header 33 bytes, code 24 bytes, dictionary 15 bytes |
| Bytes read | 72, the whole file |
| Validity checks | 6, all passed |
| Code walks | 2: one for the references, one for the translation's first pass |
| Operations walked | 14, each walk |
| Native instructions written | 23 |
| Named slots set up | 1, for `count` |
| Parse steps | 0 |
| Other files loaded | 0 |
| Network calls | 0 |

Read the table honestly, because a count of work is not a measurement of time, and the chapter does not pretend it is. The table counts what the loader does, not how long it takes. How long 3 reads of 72 bytes take on the spinning drive, and how long the translation of 14 operations takes on the two cores, are measurements, and they belong to Chapter 31, on the real PC. What the table shows is enough for the design: the startup is a few long reads, six checks, one translation, and a jump, and nothing else. Compare it with the shape section 1.4 counted for Python: an 8 MB interpreter binary, startup files, and parts of a 55 MB standard library, read from scattered places on the disk. The Riza startup reads one file, 72 bytes, in 3 reads, and the comparison is in the shape of the work, not in a time this chapter has not measured.

One more honest note about the translation, because section 10.3 already raised it and this section is where the cost lands. The translation is computing, and this chapter says so. Section 2.2 promised that loading is reading, not re-translating, and the promise was about the parse: the parse never happens at load, because the parse happened once, when the program was written. The translation is not a parse. It does no analysis, no tree building, no error recovery. It reads one step and writes its native instructions. It is mechanical and short, and its cost is part of the load, counted in the table above as 23 native instructions written, and measured in Chapter 31. The book keeps the complicated step out of the load while admitting the mechanical one in, and it says so in the open, in both chapters.

The startup is designed now: eight steps, in order, from the first byte of the header to the first operation of main, with nothing hidden and nothing extra. The next section designs the other end: what happens when the program ends, and why it ends cleanly, every time.

## 15.2 Clean shutdown, every time

Start with the job, in one sentence. When the program ends, the finish routine runs, and it leaves nothing half done.

The program ends at END, and END becomes `call finish`, the way section 10.3 designed it. The finish routine is the loader's own routine, and it ends the program cleanly. Why the loader's routine, and not the program's own code? Because the loader started the program, jumping to the entry point the way section 9.3 described, and there is no return address waiting, so the program cannot return. It asks the loader to finish it, and the loader does. The program's last act is a call, and the call is to the routine that started it.

The finish routine has five steps, in order, and the order is the design, so it is stated in full.

First, flush the output. For each file the program opened, write any bytes still waiting in memory to the disk, so nothing the program wrote is lost. A program that writes to a file holds the written bytes in memory until they are flushed, and the flush is what puts them on the disk. The finish routine flushes every open file, so the disk holds everything the program wrote.

Second, close the files. Release each file the program opened, in the reverse order they were opened, so the last file opened is the first closed, the way a pile unwinds. Closing a file tells the system the program is done with it, and the file is free for the next program that wants it.

Third, free what the program owned. The heap values and the arenas go back, under the ownership rule from section 6.1: every value has one owner, and the owner frees it when it is done, and the program ending is the moment every value it owned is done. The pile is released whole, and the stack memory goes back with it, the way section 6.3 described: stack memory frees itself when a function returns, with no decisions to make, and at the end of the program there is no function left to return to, so the whole pile goes back at once.

Fourth, release the static memory. The data section, the name table, the function mapping, and the slots for the named values go back, the way section 6.3 promised: static memory is freed as part of clean shutdown, which this chapter designs, and when the program ends, its static memory goes back with everything else.

Fifth, report the exit, in plain words, to whoever started the program. The report says the program ended, and whether it ended well or on an error. If it ended on an error, the report names the error, with the error's message, the way section 7.3 described: an error is a value with a code and a message, and the message says what happened in plain words. The report goes to whoever started the program: the build tool, when it runs the program's tests, the way section 14.1 described, or the program that asked for this one to run. The report is the last thing the finish routine does, because it describes the whole shutdown, and it cannot be written until the shutdown is done.

The finish routine is small, the way section 2.4 requires. Five steps, no hidden work, nothing on a schedule the user cannot see. It runs when the program ends, and only then. It does not run in the background. It does not phone home. It does not check for updates. It finishes the program and reports, and that is all it does.

Every program shuts down cleanly, every time, and that includes a program that ends on an error. An error is a value, the way section 7.3 designed it, not a crash, and a program that ends on an error shuts down the same way as a program that ends well: the same five steps, in the same order, with the report naming the error. Section 7.3 named the three ways a program deals with an error, and the second way is to give up cleanly: save its work, close its files, and stop, with the error stated in plain words, the way this chapter designs it. Giving up is not a failure of the program. It is the program doing its job when the job cannot be done.

Now work one shutdown through, step by step, with the work counted. The program opens a file, writes one line to it, and ends. Here is the text, in the form Chapter 8 designed:

| Line | Text | Bytes |
| --- | --- | --- |
| 1 | `notes = open_file("notes.txt")` | 30 characters + 1 newline = 31 |
| 2 | `if notes is error:` | 18 characters + 1 newline = 19 |
| 3 | `    print(notes)` | 16 characters + 1 newline = 17 |
| 4 | `write(notes, "hello")` | 21 characters + 1 newline = 22 |
| Total | | 89 bytes |

The program is 89 bytes of text. The `open_file` call is the one section 8.1 used, and the error check is the branch section 8.1 showed: if the answer is an error, the program prints it and ends, jumping to END after the print, so the write never runs with an error value. The file routines are Chapter 16's work, and the names here show the shape, the way section 8.1 showed `open_file` before Chapter 16 names the routines. The program's middle form is not counted here, because the file routines belong to Chapter 16. What this section counts is the finish routine's work, which is this chapter's subject.

Run the program with the file present. `open_file` returns the file. The check finds no error, so the print is skipped. The program writes `hello` to the file: 5 characters, 5 bytes, held in memory, waiting to be flushed. The program reaches END, and END becomes `call finish`. The finish routine runs.

First, flush. The file has 5 bytes waiting in memory, the word `hello`, and the finish routine writes them to the disk. The file on the disk holds 5 bytes.

Second, close. One file is open, `notes.txt`, and the finish routine closes it. The file is free for the next program.

Third, free. The program owned one value, the file value, and the finish routine frees its memory. The pile is released whole.

Fourth, release the static memory. The name table, the function mapping, and the slots go back.

Fifth, report. The program ended well, and the report says so, in plain words, to whoever started the program.

| Step | Work, success run |
| --- | --- |
| 1. Flush | 5 bytes written to the disk |
| 2. Close | 1 file closed |
| 3. Free | 1 value freed, pile released |
| 4. Release static | Name table, function mapping, slots |
| 5. Report | The program ended well |

Now run the same program with the file missing. `open_file` returns an error, because the file does not exist. The check finds the error, so the program prints it and ends, giving up cleanly, the way section 7.3 described. The program reaches END, and END becomes `call finish`. The finish routine runs, the same five steps, in the same order.

First, flush. No files are open, so there is nothing to flush: 0 bytes.

Second, close. No files are open, so there is nothing to close: 0 files.

Third, free. The program owned one value, the error value, and the finish routine frees its memory. The pile is released whole.

Fourth, release the static memory. The name table, the function mapping, and the slots go back.

Fifth, report. The program ended on an error, and the report names it, in plain words: the file notes.txt does not exist. The report goes to whoever started the program, and the user sees what happened, instead of a frozen screen or a vanished program.

| Step | Work, error run |
| --- | --- |
| 1. Flush | 0 bytes, no files open |
| 2. Close | 0 files, no files open |
| 3. Free | 1 value freed, the error value, pile released |
| 4. Release static | Name table, function mapping, slots |
| 5. Report | The program ended on an error: the file notes.txt does not exist |

Read the two runs together, because the sameness is the point. The same five steps, in the same order, whether the program ends well or ends on an error. The only difference is in the work each step finds to do, and in what the report says. A shutdown that changes shape when the program fails is a shutdown the programmer cannot trust, because the failure is the moment the shutdown matters most. A shutdown that is the same every time is a shutdown the programmer can count on, and the count is above.

Say honestly what the finish routine cannot do, because the limits are as much a part of the design as the five steps.

When the program is killed from outside, there is no clean shutdown. A program the user stops, or the system stops for using more than its share, ends without reaching END, and the finish routine never runs. The files it had open are not flushed by the finish routine, and bytes still waiting in memory are lost. Files already closed are safe, because closing put their bytes on the disk. Files open at the kill are not safe, and the book says so plainly instead of pretending otherwise.

When the machine loses power, there is no clean shutdown either. The finish routine is a program, and a program cannot run without power. Bytes still waiting in memory are lost, and a file written halfway through a flush may hold part of what the program wrote. The safety of the file on the disk, what survives a power cut and what does not, is the filesystem's design problem, and it is separate from this chapter's: this chapter is the program's shutdown, and the file's safety on disk is the filesystem's work.

And a program that never reaches END never shuts down. Section 7.2 named the loop that never ends: a loop whose test never fails runs forever, and the program never moves past it. The finish routine runs when the program ends, and a program that does not end does not shut down. Stopping it from outside is the unclean case, the way this section described it. The design answer is the same as everywhere in this book: the test is visible in the code, the programmer can read it and see whether it can fail, and the system does not guess whether a loop will end, because guessing would be hidden work, and section 2.4 forbids hidden work.

The debug log layer in Chapter 17 records what the startup and the shutdown did, so the record is there when something goes wrong: which file was started, which checks ran, how the program ended, and what the report said. The log is a plain record, not a hidden trace, and the programmer reads it the way the user reads the error itself: in plain words, in the open. Section 2.4 promised that every program carries a debug log layer, and section 7.3 said the layer records errors as they happen. The startup and the shutdown write to it too, because the beginning and the end of a program are the two moments the record matters most.

The chapter counts as done when the startup and the shutdown run on the target PC: the loader starts the three counted programs, `print("hello")`, the hello loop, and the branch, and shuts each one down cleanly, with the times measured in Chapter 31. That is part 5 of section 3.3: anything the book says runs on the target machine has been run there, or is marked plainly as waiting for the real-PC run. Until then, the startup and the shutdown are designs waiting for their proof, and the book says so.

Part 4 has begun. Chapter 16 builds the core library, the routines the startup uses and the file routines the finish routine closes. The kernel's own startup is the same shape at a bigger scale, the way Chapter 22 designs it: the kernel is a program, and starting it is loading, checking, placing, translating, and jumping to its entry point. Chapter 23 designs boot, from power on to prompt, where the first program is started. Chapter 30 builds the full system image, and Chapter 31 measures the startup and the shutdown on the real PC: the time from the first read to main's first operation, and the time from END to the exit report.

---

Startup and shutdown are the two ends of a program's life, and both are designed to be small, visible, and counted. Startup is the loader's eight steps, in order: read the header, run the first four checks, read the sections, run the last two checks, place the sections, translate the middle form, set up the pile and the static memory, and jump to the entry point. The hello loop's startup is 3 reads of 72 bytes, 6 checks, 14 operations walked, 23 native instructions written, 1 slot set up, and no parse, no libraries loaded, and no network. Shutdown is the finish routine's five steps, in order: flush the output, close the files, free what the program owned, release the static memory, and report the exit in plain words. The same five steps run whether the program ends well or ends on an error, and a program that writes 5 bytes to a file flushes all 5 before it ends, while a program that ends on a missing file reports the error and shuts down cleanly, with nothing half done. What the finish routine cannot do is stated too: a program killed from outside, or a machine that loses power, gets no clean shutdown, and the book says so. The runtime has begun. Chapter 16 builds the core library.
