# Chapter 22: Kernel design: small on purpose

Part 1 named the waste, the idea, and the rules. Part 2 built the language. Part 3 built the binary and the compiler. Part 4 built the runtime and the core library: Chapter 15 designed the startup and the shutdown, Chapter 16 designed the routines every program calls, and Chapter 17 designed the debugging that is built into every program from the start. Part 5 built the tools and the shell: Chapters 18, 19, and 20 built the file tools, the text tools, and the developer tools, and Chapter 21 built the shell that runs them. Part 6 begins here, with the operating system they all run on, and this chapter is the first of it: the kernel, the small core at the center of the system.

The kernel is the part of the system that owns the hardware and hands out memory, files, and time, the way section 1.2 defined it. Everything in the system depends on it, so its size and its behavior set the floor for the whole machine. Section 2.4 said the core in Riza is small on purpose, and does no hidden work. This chapter designs the kernel on that rule: it does only what must be in the kernel, and nothing else, with no layers for the sake of layers.

There are two sections. The first names what the kernel does and what it does not do, each item with its reason, and works one kernel act through: a program asks the kernel to start another program. The second names the layers the kernel does not have, and works one call through: a program asks for the time.

One warning, stated up front the way Chapters 4 through 21 stated their own. This chapter designs the kernel. It does not build it. No kernel exists yet, and no claim in this chapter says one does. The chapter also claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own examples, counted by hand. No time is claimed anywhere in this chapter, because no kernel has been measured on the target machine, and Chapter 3 forbids numbers without a method. The measured proof comes in Chapter 31, on the real PC.

## 22.1 What the kernel does, what it does not do

Start with the job, in one sentence. The kernel does only what must be in the kernel, and nothing else, because every job inside the kernel is a job every program pays for, and every job outside it is a job only the programs that need it pay for.

The test for what belongs inside is one question: can this job live outside the kernel? If it can, it lives outside, where it can be added, removed, or replaced without touching the kernel, and a program that does not need it does not pay for it, the way section 2.4 said. The kernel holds only the jobs that cannot live anywhere else: the jobs that need the hardware itself, or the trust to hand out what programs share.

The kernel does five things.

First, it starts and stops programs. Starting a program is the loader's work from section 15.1, run by the kernel: read the file, run the checks, place the sections, translate the middle form, set up the memory, and jump to the entry point. Stopping a program is the finish routine's work from section 15.2, run by the kernel when the program ends: flush, close, free, release, report. The kernel runs the loader because starting a program is the act of handing out the machine's shared goods, memory and time, and only the kernel hands those out.

Second, it shares the two cores fairly between programs. The target machine has two cores, from section 3.1, so two programs run at once, and a third waits its turn. Fair sharing means no program takes a core and keeps it while another waits. How the sharing is decided is Chapter 24's design.

Third, it owns the memory and hands it out. The machine has 8 GB, from section 3.1, and every program's memory comes from that one store. The kernel keeps the record of who has what, hands out a run when a program asks, and takes it back when the program is done with it. Ownership runs through it the way section 6.1 requires: the kernel owns the record, and a program owns what it was handed until it gives it back. When a program asks for more than what is free, the program's side of the failure is section 6.4's four steps: the ask, the clean no, the error the program can see, and the program's choice. The kernel's side, what happens when the whole machine is out, is Chapter 25's design, the way section 6.4 said.

Fourth, it guards the machine's devices, so only one program at a time talks to each device. The screen, the keyboard, the disk, and the network are single things, and two programs using one at once would each get the other's answer. The kernel stands between the programs and the devices, and a program reaches a device only through the kernel. The drivers, the small modules that talk to one device each, are Chapter 27's design.

Fifth, it keeps the time. The clock is hardware, and the kernel is the only part of the system that talks to hardware, so the time lives with the kernel. A program that wants the time asks the kernel, the way section 16.2 said the time routines ask the system. Chapter 28 designs the time service, the part that keeps the clock set and serves the time to programs.

The kernel does not do five things, and each one has its reason and its home.

First, it does not read files. Reading a file means knowing how files are laid out on the disk, and that knowledge is large and changes with the disk, so it lives outside the small core, in the filesystem, which is a service Chapter 26 designs. Say plainly how that fits with the loader, because the loader reads the program's file. The kernel runs the loader's steps, and when a step needs bytes from the file, the kernel asks the filesystem service for them. The kernel never learns the disk's layout. The reading is the service's work. The starting is the kernel's.

Second, it does not draw on the screen. Drawing means knowing the screen's hardware, and that knowledge lives in the screen driver, a small module Chapter 27 designs. The kernel calls the driver, and the driver draws.

Third, it does not run the shell. The shell is a program, the way Chapter 21 designed it: it reads a line, splits it, finds the routine, calls it, and reports the result. It runs like any program, and the kernel starts it, as the first program, the way Chapter 23 designs it. But the kernel does not contain the shell: a shell that lived in the kernel would make the kernel as large as the shell, and every program would pay for a shell whether it used one or not.

Fourth, it does not check for updates. Checking for updates means reaching for the network, asking what is new, and deciding about versions, and none of that belongs in the small core. The update service is a program, Chapter 28 designs it, and it runs when the user asks it to. Nothing in the kernel reaches for the network on its own.

Fifth, it does not decide what the user is allowed to do. The rules about who may do what belong to the user, in data the user can read and change, and Chapter 29 designs them. The kernel reads the permission data and follows it: it starts a program the data allows, and it refuses a start the data forbids. It does not invent rules, and it does not change them.

The kernel's own code is written in Riza, in the same language, and translated the same way, the way section 10.3 described: middle form in, native instructions out, one translator for every program in the system, from the smallest script to the kernel itself. There is no second translator for the kernel, no special form for the operating system. The kernel's own startup is the same shape at a bigger scale, the way Chapter 15 closed: loading, checking, placing, translating, and jumping to the entry point. The loading is the bootloader's work, and Chapter 23 designs it.

The kernel has no garbage collector. Section 1.3 counted what a collector costs: pauses that arrive without warning, the worst measured 2.2 seconds on Python 3.12, 320 milliseconds on Python 3.14, and 33 to 50 milliseconds in Go, on a test built to stress the collector. A pause in a program stops that program. A pause in the kernel would stop every program on the machine at once, because every program waits on the kernel. So the kernel follows the same memory rules as every program, from Chapter 6: every value has exactly one owner, the owner frees it when done, values that live and die together share an arena and are freed in one act, and there is no background detective finding memory to reclaim. The kernel's memory is freed by its own structure, and no pause arrives unannounced.

The kernel starts once, at boot, the way Chapter 23 designs it. Section 1.4 counted the startup cost every program pays today, and the kernel pays its own startup once per boot, not once per program. The programs it starts pay the loader's eight steps each time they start. The kernel pays its startup once, and then it runs until the machine is turned off.

The kernel is sized for the machine from section 3.1: two cores, 8 GB of memory, and the cache from section 5.3. The design rule for the kernel's code is the cache rule: the code on the common path, the code that runs on every kernel call, is sized to fit the 64 KB instruction cache on each core, 65,536 bytes, so the path a call takes stays in the fastest cache the chip has. The kernel's data follows the same rule in Chapter 25. Whether the code fits is measured in Chapter 31, on the real PC, and this chapter claims no size for code that is not written yet.

Now work one kernel act through, step by step, with the steps counted. A program asks the kernel to start another program. The asking program is the tester from section 20.2, and the program it asks for is the hello loop from section 4.4, as a complete file of 72 bytes, the way section 9.2 counted it. Section 15.2 said the finish routine's report goes to whoever started the program, and the build tool running the program's tests is the example it gave.

The request is one call. The tester calls the kernel's start routine by number, the way a program calls a core-library routine by number from section 16.1, and names the program file: `hello`. One call, one name.

The kernel runs two checks before the loader runs, in order. First, the file exists. The kernel asks the filesystem service for the file, and the service says it is there. A file that is not there ends here, with an error, code 1, file not found, and nothing starts. Second, the permission data allows the start. The kernel reads the data Chapter 29 designs, and the data says the tester may start this program. A start the data forbids ends here, with the refusal reported in plain words, and the error code for it is Chapter 29's to fix. Existence comes first, because there is nothing to allow if the file is not there. Both checks pass, and the loader runs.

The loader's eight steps run, by the kernel, the way section 15.1 counted them for the hello loop.

First, the header. One read, 33 bytes. The magic is R I Z A. The version is 1. The entry point is 0. The code is at offset 33 with size 24. The data is at offset 57 with size 0. The dictionary is at offset 57 with size 15.

Second, the first four checks. The magic matches. The version is known. The sizes add up: 33 plus 24 is 57, plus 0 is 57, plus 15 is 72, and the file is 72 bytes. The bounds hold: the code ends at 57, the data ends at 57, the dictionary ends at 72, and none of them runs past the end of the file or overlaps another. All four checks pass.

Third, the sections. Two reads: the code section, 24 bytes, and the dictionary section, 15 bytes. The data section is empty, so nothing is read for it. Three reads in total, 72 bytes, the whole file.

Fourth, the last two checks. The references: one forward walk of the 14 operations. The name numbers are 0 and 1, and the table has 2 entries, so both are inside. The jump targets are 23 and 4, both inside the 24 byte code section, and both on the start of an operation: 23 is END, 4 is the loop test. The entry point is 0, the first operation. The value shapes: the data section is empty, so the check runs and finds nothing. Both checks pass.

Fifth, the placing. The code section goes where the program runs from, 24 bytes. The name table, 13 bytes, and the function mapping, 1 byte, the count saying 0, are held for the run. The comments, 1 byte, the count saying 0, stay on disk.

Sixth, the translation. The first pass walks the 14 operations and counts the native bytes each step becomes. The second pass writes the native instructions: 23 of them, the count section 10.3 reached by hand.

Seventh, the setup. The pile is set up on the machine's own stack, empty. One slot is set up in static memory, for the name `count`, sized to the name's type.

Eighth, the jump. The kernel jumps to the entry point, position 0, and the first operation runs: LOAD_CONST 0, put the number 0 on the pile. The hello loop is running.

The program runs. It prints `hello` three times, and it reaches END. END becomes `call finish`, the way section 10.3 designed it, and the finish routine's five steps run, by the kernel, the way section 15.2 counted them.

First, flush the output. The hello loop opened no files, so there is nothing to flush. Second, close the files. None are open, so there is nothing to close. Third, free what the program owned. The pile is released whole. Fourth, release the static memory. The name table, the function mapping, and the slot for `count` go back. Fifth, report the exit, in plain words, to the tester that asked for the program: the program ended well.

Now the totals, the whole act, from the request to the report.

| Work | Count |
| --- | --- |
| Kernel calls | 1, the start call naming `hello` |
| Kernel checks | 2, the file exists and the permission data allows it |
| Loader steps | 8, the loader's eight steps from section 15.1 |
| Finish steps | 5, the finish routine's five steps from section 15.2 |
| Steps in all | 16: 1 plus 2 is 3, plus 8 is 11, plus 5 is 16 |
| Reads | 3, the header, the code, the dictionary |
| Bytes read | 72, the whole file |
| Checks in all | 8: the kernel's 2, and the validity checks' 6 |
| Native instructions written | 23, from 14 middle-form operations |
| Static slots set up | 1, for `count` |

Read the total honestly, because rule 7 of section 1.5 says to report the number that loses too. Starting a program through the kernel costs 2 checks more than the loader's eight steps alone. The checks are the price of the guard: the kernel checks that the file is there and that the start is allowed, because it is the kernel's job to guard what is shared, and the guard costs two checks. A loader that ran without the kernel would skip them, and a loader that skipped them would start any file any program named, allowed or not.

Now the failure runs, because a start that only works when the file is good is a start the user cannot trust. The tester asks for a program that does not exist, `missing`. The kernel asks the filesystem service, the service says there is no such file, and the start call hands back an error, code 1, file not found, with the message naming the file. Nothing starts, nothing is read, and the loader never runs. The tester asks for a file that fails the validity checks, a file whose magic is not R I Z A. The header is read, the first check fails, and the start call hands back an error, code 8, wrong kind, with the message naming the check that failed, the code section 16.3 kept for the ones only the run can see. Nothing runs, and no memory is placed, the way section 9.4 requires: no instruction executes, no memory is placed, no program starts.

One thing this chapter does not fix, stated plainly. How the kernel names a running program, so the program that asked for it can tell one running program from another, is Chapter 24's design, with the processes and the scheduling. The start call hands back the answer that the program started, and the finish routine's report names the program when it ends. The naming of what runs between those two moments belongs to the chapter that designs running.

## 22.2 No layers for the sake of layers

Start with the job, in one sentence. The kernel has no layers it does not need: a program talks to the kernel directly, with a call, the kernel talks to a driver directly, with a call, and the driver talks to the device.

Section 1.2 counted the layers a keystroke travels through today, and the count is the reason for this section. Follow it again, and count the handoffs. The keyboard hardware signals the driver. The driver hands the event to the kernel. The kernel hands it to the display server. The display server hands it to the GUI toolkit. The toolkit hands it to the runtime. The runtime hands it to the app. That is five handoffs on the way up. The answer travels back down: app to toolkit, toolkit to display server, display server to graphics driver, graphics driver to screen. That is four handoffs on the way down. Nine handoffs in all, five up and four down, and each crossing copies data, checks types, and translates between one layer's idea of an event and the next layer's, for every key, every mouse move, every frame.

The string is the same lesson in a different shape. A string read from disk can be copied four or five times before it reaches the screen: from the disk cache to a kernel buffer, to a runtime buffer, to an app string object, to a GUI text object, to a texture on the graphics card. Six stops, each holding its own copy, because each layer keeps its own private copy of the world in its own private format, and every layer re-does work the layer below already did.

The kernel in Riza has three links in its chain, and no more. A program talks to the kernel directly, with a call, the way a program calls the core library. The kernel talks to a driver directly, with a call. The driver talks to the device. There is no layer between the program and the kernel, and no layer between the kernel and the driver.

Say how a call reaches the kernel. A program calls a kernel routine by number, the way a program calls a core-library routine by number from section 16.1. The numbers are assigned with the library's numbers, the way section 16.3 left the routine numbers to be fixed, in Appendix A, and this chapter names the calls it uses and leaves the numbering to the appendix. Say how the kernel answers. The routine returns a value or an error, the way section 7.3 designed errors: a value the caller holds, with a code and a message, from the standard codes section 16.3 fixed. A kernel call that fails hands back an error, and the program deals with it the way section 7.3 said: check it, pass it up, or give up cleanly.

Work the chain through on a call the reader already knows: printing. A program calls print, the library routine from section 16.1. The print routine touches the world, and a routine that touches the world asks the kernel, because the screen is a device and the kernel guards the devices. The kernel calls the screen driver, the small module Chapter 27 designs. The driver writes to the screen. Four acts: one library call, one kernel call, one driver call, one device write. The text is handed along, not copied into a fresh buffer at each step, because there is one form from top to bottom, the way section 2.3 designed it, and no layer keeps a private copy of the text in a private format.

The keystroke takes the same short road. The keyboard driver reads the key, hands it to the kernel, and the kernel hands it to the program that is waiting for it, the shell, when the user is typing at the prompt. Two handoffs, driver to kernel and kernel to program, against the nine section 1.2 counted. The display server, the toolkit, and the runtime are not in the road, because no program in Riza sits between the kernel and the program that typed the key.

Now work one call through, step by step, with the steps counted. A program asks for the time, by calling `now`, the routine from section 16.2, and `now` asks the kernel, because the clock is hardware and only the kernel talks to hardware.

First, the call. The program calls the kernel's time routine by number, with no values to hand over, because asking the time needs no argument. One call.

Second, the clock. The kernel reads the machine's clock, the way a driver reads a device: one read, and the clock answers with the count. One read.

Third, the answer. The kernel hands back the count of seconds since the start of 1970, the standard count from section 16.2, as an 8-byte number. At the start of 15 November 2026 the count was 1,794,700,800, the number section 16.2 gave, and any reader can check it against a clock. The program holds the number, and the call is done. One return.

Now the totals, the whole call.

| Work | Count |
| --- | --- |
| Kernel calls | 1, the time call |
| Clock reads | 1 |
| Returns | 1, the count, 8 bytes |
| Steps in all | 3: 1 plus 1 plus 1 |
| Crossings | 2, program to kernel and kernel to program |
| Copies of the value | 0, the number is handed back, not copied through buffers |

Set the counts beside today's, and read them honestly. The time call crosses 2 boundaries and copies the value 0 times. The keystroke crosses 9 handoffs. The string is copied 4 or 5 times. The jobs differ, and the chapter says so: asking the time is not typing a key, and typing a key is not reading a string. The comparison is about layers, not about identical work. Today, every crossing copies, checks, and translates, because every layer keeps its own copy in its own form. In Riza, the call crosses one boundary each way, and the value is handed back, because there is one form and no private copies. The boundary that remains is needed: the program cannot read the clock itself, because the clock is hardware, and only the kernel talks to hardware. What is gone is the layers that added nothing.

Say what the kernel does not hide, because section 2.4's test applies to the kernel as much as to any program. Every call is named: the kernel's calls are named in this chapter and in the chapters that design each job, and a call the book does not name does not exist. Every cost is visible: the steps are counted here, 16 for a start, 3 for the time, and Chapter 31 measures the time they take on the real PC. And the user can see what the kernel did in the debug log, the way section 17.1 designed it.

The kernel writes to the log the way any program does. The kernel is a program, so it keeps its log in a `logs` folder beside its file, named for the kernel, one line per entry, in plain text, each line starting with the time from `now`. The log holds what the kernel did at the level of acts: which program it started, which call failed, and why. It does not hold every call. Logging every time call would flood the log with lines nobody reads, so the kernel logs the acts, not the calls. The log follows the same rules as any program's: a buffer of 1,024 bytes, a limit of 64 KB, the oldest lines dropped whole when the limit is reached, and the rule from section 16.3, log the act, never the secret. The user can read the kernel's log, measure what it costs, and turn logging off, because section 2.4's test asks whether the user can turn it off, and the kernel is held to it like everything else.

Work one kernel log line through, so the shape is concrete. The tester asks the kernel to start the hello loop, and the kernel writes one line when the program starts. The line is the time, a space, and the act, in plain words: `1794700800 started hello`. The time is 10 digits, the count section 16.2 gave. The line is 24 characters plus the newline, 25 bytes, the same count as the started line in the hello loop's own log from section 17.1, because the kernel's log and the program's log are the same shape, written by the same layer. When the program ends well, the kernel writes one more line: `1794700800 ended hello well`, 27 characters plus the newline, 28 bytes. Two lines, 53 bytes, for the whole act, from the start to the report. A start that fails writes one line naming the failure: `1794700800 start failed missing code 1`, 38 characters plus the newline, 39 bytes, the file named and the code given, so the user can see what happened without asking.

The test suite grows with this chapter. Eight new tests cover the kernel. Five cover starting and stopping: the kernel starts the hello loop, which prints `hello` three times and ends well; starting a program that does not exist gives error code 1, and nothing runs; starting a file that fails the validity checks gives code 8, and no memory is placed; the finish routine's five steps run when the program ends, and the report reaches the program that asked for it; and the kernel starts two programs, and both run to the end. Three cover calls and the log: the time call returns the count of seconds since the start of 1970, as an 8-byte number; a kernel call that fails returns an error value, and the calling program goes on; and the kernel writes one line to its log when it starts a program, and one line when a start fails. That is 8 new tests, and the suite grows from 112 to 120: 112 plus 8 is 120. Each test runs on the target PC, the way section 2.5 demands.

This chapter adds no new routines to the core library. The kernel's calls are the kernel's own, numbered the same way the library's routines are numbered, and the library stays at 34 routines. The suite grows to 120 tests.

The chapter counts as done when the kernel runs on the target PC from section 3.1: the kernel starts each of the three counted programs, the hello loop and the two from section 13.2, and shuts each one down cleanly, the time call returns the count, the log holds the start and the end, and the times are measured in Chapter 31. Until that run happens, the chapter's claims are marked plainly as waiting for the real-PC run, the way part 5 of the done test in section 3.3 requires.

Chapter 23 designs boot, from power on to the prompt, where the prompt is the shell, and the shell is the first program the kernel starts. Chapter 24 designs processes and scheduling, fair sharing on 2 cores, and names the running programs. Chapter 25 designs memory in the kernel, who owns what, and protection without waste. Chapter 26 designs the filesystem, layout on disk and crash safety, and the service the kernel asks for a file's bytes. Chapter 27 designs the drivers, screen, keyboard, disk, and network, small modules in the same language. Chapter 28 designs the services, time, network, and updates. Chapter 29 designs the security model, permissions in plain words, and fixes the code for a refused start. Chapter 30 builds the full system image, and the kernel is part of it. Chapter 31 measures the kernel on the real PC, on the standard tasks from section 1.5, and Chapter 32 benchmarks it against today's stack.

---

The kernel is small on purpose: it starts and stops programs, shares the two cores fairly, owns the memory and hands it out, guards the devices, and keeps the time, and it does nothing else. It does not read files, draw on the screen, run the shell, check for updates, or decide what the user is allowed to do, because each of those jobs lives somewhere else, in a service, a driver, a program, or in data the user owns. Its own code is written in Riza and translated the same way as every program, with one translator from the smallest script to the kernel itself. It has no garbage collector, because a pause in the kernel would stop every program at once. It starts once, at boot, and its code on the common path is sized to fit the 64 KB instruction cache on each core. One act worked through by hand shows the proof: starting the hello loop is 1 call, 2 checks, 8 loader steps, and 5 finish steps, 16 steps in all, 3 reads, 72 bytes, and 23 native instructions, with the failures reported as error values and nothing half run. And it has no layers for the sake of layers: a program talks to the kernel directly, the kernel talks to a driver directly, and the driver talks to the device. One call worked through by hand shows the proof: asking the time is 3 steps, 2 crossings, and 0 copies, beside the keystroke's 9 handoffs and the string's 4 or 5 copies today. Every call is named, every cost is visible, and the kernel writes what it did in the debug log. No new routines join the library, and the suite grows from 112 tests to 120.
