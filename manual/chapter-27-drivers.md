# Chapter 27: Drivers: screen, keyboard, disk, network

Part 6 began with the kernel: Chapter 22 designed the small core at the center of the system, Chapter 23 designed the boot, Chapter 24 designed processes and scheduling, Chapter 25 designed memory in the kernel, and Chapter 26 designed the filesystem. This chapter designs the last piece of the kernel's side of the system: the drivers, the small modules that stand between the kernel and the hardware. Every other part of the system talks to a device through a driver.


Six promises from earlier chapters meet here. Section 21.1 named no core-library routine for reading from the keyboard, and said devices are the drivers' work, this chapter's work. Section 22.1 fixed that the kernel guards the devices, one program at a time per device, and that the drivers are this chapter's design. Section 23.2 started four drivers one at a time at boot, and left the one-driver-at-a-time rule to section 27.2. Section 24.2 left how the clock's second changing wakes the kernel to the clock driver. Chapter 25's closing fixed that a driver's memory is the kernel's own, kind 7, because the driver runs as part of the kernel. And Chapter 26's closing left how the service and the disk driver divide the work to this chapter.

There are two sections. The first designs the driver shape and each of the five drivers in turn: the screen, the keyboard, the disk, the network, and the clock. The second designs the order the drivers are built and started in, one at a time, and the test each one must pass on the real PC before the next begins.

One warning, stated up front the way Chapters 4 through 26 stated their own. This chapter designs the drivers. It does not build any of them. The chapter also claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own examples, counted by hand: cells, bytes, blocks, handoffs. No time is claimed anywhere in this chapter; Chapter 3 forbids numbers without a method. The measured proof comes in Chapter 31, on the real PC.

## 27.1 Driver shape: small modules, same language

Start with the job, in one sentence. A driver is the one module that talks to one device, so the rest of the system never has to know how the device works.

Each device's hardware works in its own way, with its own signals and its own timing, and that knowledge lives in exactly one place per device: the driver. The kernel does not know how the disk moves its head, and a program does not know how the screen turns text into light. Each of them knows one driver's calls, and the driver knows the device.

The shape is the same for all five drivers, and it is fixed here. Each driver is a small Riza module, written in the same language as every program in the system, in the same middle form, translated by the same translator. Section 10.3 fixed that: one translator, from the smallest script to the kernel itself, and the drivers are part of the kernel's side of the system. There is no second language for drivers. A driver is loaded by the kernel through the loader's eight steps from section 15.1, the way section 23.2 designed it. A driver runs as part of the kernel, not as a program of its own: it has no number in the running-program record from section 24.1, and its memory is the kernel's own, kind 7 in the record from section 25.1, owner 0, the way Chapter 25's closing fixed it.

Say what a driver holds, because a driver holds exactly three things.

First, the calls it answers, named in this section, one small set per driver. The calls are made by number, the way section 22.2 calls the kernel, and the numbers live in Appendix A. A call hands values over and hands back a value or an error, the way section 7.3 designed errors.

Second, the state it keeps: small, stated in this section for each driver, counted in bytes, and owned by the driver, the way section 6.1 requires. No driver's state is a second copy of anything another part of the system holds. Where a driver's state looks like a copy, this section says what it actually is: the device's own current state, held in the one place that tends the device.

Third, the device itself, which only the driver touches. A program cannot touch a device except through the kernel and the driver, and section 25.2's protection is the wall behind that rule: a program that reaches for the kernel's memory and calls itself is stopped the way a program that reaches for another program's memory is stopped. The bootloader follows the same rule: section 23.1 fixed that it carries the disk driver and the screen driver in native form, so both drivers live twice, one source in two stored forms.

Now the guard, from section 22.1, because the guard lives in the driver and this chapter fixes how it works. Two programs using one device at once would each get the other's answer, so the kernel lets one program use a device at a time, and the kernel names the asking program's number, from section 24.1, with every call. The rule for a second program asking for a busy device is fixed here: it waits. It is not refused, because a busy device is not a failure. It is a queue, served in the order they asked, the way section 24.2 serves the waiting list for the cores. The waiting program gives up its core, and the kernel wakes it when the device reaches it. Each driver keeps the line of waiting programs as program numbers, 4 bytes each, in its own state. The device is held for one answered call at a time, so two programs' calls never mix inside one act on the device. A program that ends while it waits leaves the line, removed by the kernel when its record entry is removed. The keyboard has one addition to this rule, stated in its part below, because a line of text is a run of calls and not one call.

Say what a driver does not do, because the list is part of the shape. A driver does not decide policy: what a program may ask a device to do is permission, data the user owns, Chapter 29's design, and the kernel reads that data before the call reaches the driver, the way section 22.1 designed it. A driver does not hold a second copy of anything. A driver does not run in the background: it answers when it is called and does nothing when it is not, because section 2.4 forbids hidden work. The one signal in this chapter that arrives unasked is the clock device's, stated plainly in the clock's part below. And a driver for a device the machine does not have is not loaded: nothing pays for a device that is not there.

The drivers' code is kernel code, so the cache rule from section 22.1 covers it: the code on the common path is part of the kernel's common path, sized to fit the 64 KB instruction cache on each core from section 5.3. The drivers' acts are logged in the kernel's log, the way section 22.2 designed it: a driver starting, a device found, a device not found, a device failing are acts, one line each. The blocks a disk driver moves are calls, and they are not logged one by one: the log would flood.

Now the five drivers in turn, each stated the same way: the calls it answers, the state it keeps, one call worked through where a count helps, and what its failure looks like.

### The screen driver

The screen is the machine's display, 1366 pixels wide and 768 tall, from section 3.1. The screen driver answers three calls.

| Call | Takes | Hands back |
| --- | --- | --- |
| Write text at a place | A line number, a column number, and text | Nothing, or an error |
| Write text | Text, written at the cursor, moving the cursor as it goes | Nothing, or an error |
| Clear | Nothing | Nothing; the screen is blank and the cursor is at the first line, first column |

Write text is the call the print chain ends in, worked through below. Write text at a place is the call the failure reports use: Chapter 23's plain-words lines are written at a named line and column, so a failure line lands where the reader's eye starts.

Say what the driver holds the screen as, because one shape is fixed here, stated as a design choice. The driver holds the screen as a grid of character cells: a fixed box of pixels holding one character. The cell size is fixed here as a design choice: 8 pixels wide and 16 pixels tall. Count the grid by hand. Across: 1366 divided by 8 is 170, with 6 pixels left over, because 170 times 8 is 1360 and 1366 minus 1360 is 6. The grid is 170 columns wide, and the 6 leftover pixels at the right edge are not used. Down: 768 divided by 16 is 48, exactly. The grid holds 170 times 48 cells: 170 times 50 is 8500, minus 170 times 2, which is 340, gives 8160 cells. Each cell holds one character, 1 byte, so the grid is 8,160 bytes. The driver also holds the cursor: its line, 1 byte, because 48 fits in one byte, and its column, 1 byte, because 170 fits in one byte. The screen driver's state is 8,160 plus 2, which is 8,162 bytes, in the kernel's own memory, kind 7.

The grid is the screen's own state, not a copy of anything a program holds: it is what is on the screen, kept by the one module that tends the screen, so the driver can scroll it and write a failure line at a place without asking any program what it printed earlier. When text at the bottom line moves the cursor past the last line, the grid scrolls: every line moves up one, the top line goes, and the bottom line is blank.

Now work the print chain through, the chain section 22.2 counted. A program prints the word `hello`, 5 characters. First, the library call: the program calls print, from section 16.1. Second, the kernel call: print touches the world, so it asks the kernel, and the kernel checks the guard, the screen free or the asker waiting. Third, the driver call: the kernel calls write text, handing the text along, not copied. The driver writes the 5 characters into the 5 cells at the cursor and moves the cursor 5 columns. Fourth, the device write: the driver writes the 5 changed cells to the screen hardware, and the word shows. Four acts in all, the count section 22.2 reached, now with the driver's side designed.

The screen's failure is the one failure in this chapter that cannot always be reported. If the screen will not answer at boot, there is nowhere to write the plain-words report. The kernel starts the screen driver first, so every later failure has somewhere to be written, and a dead screen at boot is the one failure the machine cannot describe: the boot stops.

### The keyboard driver

The keyboard driver answers one call.

| Call | Takes | Hands back |
| --- | --- | --- |
| Next key | Nothing | One key: a character for a character key, the newline character for the Enter key |

One call, one key, in the order the keys were pressed. Keys pressed before any program asks are held in a small buffer, not lost. Its size is fixed here as a design choice: 32 keys, 1 byte each, 32 bytes. When the buffer is full, a new press is not kept, stated plainly as the limit. The driver's state is the buffer plus the waiting line every driver keeps, in the kernel's own memory, kind 7.

This call settles the gap Chapter 21 left open. Section 21.1 designed the shell reading a line from the keyboard, up to the newline, and named no core-library routine for the read, because the keyboard is a device. Here is the read in full. The shell asks the kernel, by a kernel call, for the next key. The kernel checks the guard and calls the keyboard driver. The driver hands back one key, or the shell waits, core given up, if no key has arrived yet. The shell collects the keys into the line it is building, one key per ask, until a key is the newline character, and then the line is complete and the shell goes on to split it. No core-library routine is named for this, because the read is the kernel's call, the way section 21.1's counts left it.

The keyboard's addition to the guard rule belongs here. A line of text is a run of next key calls, and serving it one call at a time under the plain guard would let a second program's ask slip into the middle of the first program's line. So the keyboard driver serves one reader at a time until the newline: the program being served keeps the keyboard until the driver hands it the newline character, and only then does the next waiting program's first ask begin. The line is the unit for the keyboard, the way one call is the unit for the other drivers.

Now work one keystroke through, the road section 22.2 counted against today's. The user presses one key while the shell waits at the prompt. The hardware signals the driver, and the driver reads the key. The driver hands the key to the kernel: handoff one. The kernel hands it to the shell: handoff two. Two handoffs in all, against the nine section 1.2 counted for a keystroke today, five up and four down, and the key is handed along, not copied the way section 1.2 counted the string copied four or five times.

### The disk driver

The disk is the machine's spinning drive from section 3.1, holding the filesystem Chapter 26 laid out in blocks of 4,096 bytes. The disk driver answers three calls.

| Call | Takes | Hands back |
| --- | --- | --- |
| Read blocks | A first block number, a count of blocks, and the memory to fill | The blocks' bytes in that memory, or an error |
| Write blocks | A first block number, a count of blocks, and the memory to take from | Nothing, or an error |
| Size at a place | A fixed place's first block number | The 8-byte size record at that place, or 0 for no file there |

The third call is the ask Chapters 23 and 26 fixed. The bootloader asks it for the kernel's place, block 64. The kernel asks it for the shell's place, block 1088, the way section 23.2 designed the shell's start: a size above 0 is a file there, a size of 0 is no file. The driver reads the first 8 bytes of the place's first block and hands the number back. It does not read the file; that is the loader's work, through read blocks.

This driver settles Chapter 26's closing: how the filesystem service and the disk driver divide the work. The division is one sentence. The service owns the records and the layout, and the driver owns the moving. The service knows everything Chapter 26 designed: the fixed places, the folder entries, the runs, the free record, and the write order from section 26.2. The driver knows blocks and nothing else: the block size, 4,096 bytes, block numbers from 0 to 244,140,624, and how to move bytes between a block and a run of memory the caller names. It does not know what a file is, or what any byte it moves means. A read of a folder's entries block and a read of a file's data block are the same call to the driver, differing only in the block number.

Two rules follow, and both are fixed here. The driver moves whole blocks, always, the way section 26.1 fixed the disk's own rule: a caller that needs part of a block reads the block and takes its part from memory. And the driver moves blocks in the order it is asked, and never reorders asks. Section 26.2's crash safety is an order, new blocks first and the entry last, and a driver that reordered a write could land the entry first and break the rule. The driver's state is the smallest of the five: no buffer and no table, nothing held between calls but the waiting line every driver keeps.

Now work one block read through. The filesystem service needs the top folder's entry, and section 26.1 fixed its place: block 1344. First, the service's ask reaches the kernel, and the kernel calls read blocks with first block 1344 and count 1, naming the memory the bytes are to land in, a heap run the service asked for, the way Chapter 25 designed. Second, the driver reads block 1344, 4,096 bytes, one whole block, into the named memory: one device read, 4,096 bytes moved. Third, the service reads the entry from the start of the block: 35 bytes, the count section 26.1 reached by hand, naming where the top folder's entries file sits. The rest of the block, 4,096 minus 35, which is 4,061 bytes, is unused, because the disk moves in blocks.

A disk that will not answer hands back an error value, code 2, cannot be read, from section 16.3, with a message in plain words naming the block that could not be read or written. The service that asked deals with it the way section 7.3 designed: a read for the loader becomes a failed start, a read for a program becomes that program's error. At boot, a disk failure stops the boot, per Chapter 23: the bootloader writes `Error 1: the kernel file was not found.` on the screen through the screen driver, and nothing runs.

### The network driver

The network driver answers two calls.

| Call | Takes | Hands back |
| --- | --- | --- |
| Send bytes | Bytes to send | The count of bytes taken by the device, or an error |
| Bytes arrived | Nothing | The bytes that have arrived since the last ask, or nothing arrived |

The driver knows bytes and nothing else: no names, no addresses, no other machines. What a connection is and what the bytes mean belong to the network service, Chapter 28's design, which runs over this driver the way the filesystem service runs over the disk driver. Bytes that arrive before any program asks are held in a buffer, the way the keyboard driver holds keys. Its size is fixed here as a design choice: 4,096 bytes, the block size from section 26.1. When the buffer is full, arriving bytes are not kept, stated plainly as the limit. An ask that finds nothing arrived hands back nothing arrived: a normal answer, not an error.

Section 3.2 fixed that the system works with no network at all, and this driver is where that rule lands. A driver that starts and finds no network to reach has found a normal answer, not a failure. The kernel logs it as an act, in plain words, and the system goes on without the network, because nothing at the prompt needs it. A send asked for with no network there hands back an error value in plain words, per section 7.3, naming the network, and the program that asked decides what to do. The driver never waits for a network to appear and never retries on its own, because section 2.4 forbids background work.

### The clock driver

The clock is a driver too, the fifth. Section 23.2 set the clock up before the four drivers it started, and section 22.1 fixed that the kernel keeps the time. The clock driver answers one call.

| Call | Takes | Hands back |
| --- | --- | --- |
| Count | Nothing | The count of seconds since the start of 1970, as an 8-byte number |

That count is what section 16.2's `now` hands a program: `now` asks the kernel, the way section 22.2 worked the call through, and the kernel's answer comes from this driver reading the clock device. The driver keeps no state but the waiting line every driver keeps. Keeping the clock set is the time service's job, Chapter 28's design. The driver only reads what the device says.

This driver settles section 24.2's deferral: how the clock's second changing wakes the kernel, so the turn check happens on the second. The shape is fixed here, stated plainly. The clock device itself calls the kernel's attention, once per second, as each second passes. Calling the kernel's attention means the device raises a signal, and the kernel runs to answer it: whichever program is running on a core gives the core up to the kernel for that moment, and takes it back when the kernel is done, unless the turn check says its turn is over. The hardware holds the signal until the kernel takes it, so no second's call is lost because the kernel was busy. When the signal comes, the clock driver reads the count, and the kernel runs the check section 24.2 designed: has any running program's turn ended, with a program waiting for a core? If yes, the running program goes to the back of the waiting list and the program at the front takes its core. If no program waits, the running program keeps its core. This is the one signal that arrives unasked, and it is the device calling, not the driver running in the background: the driver does nothing between signals and polls nothing. The signal covers the case the other two checks from section 24.2 miss: a program that calls nothing and waits for nothing, running out its turn in silence. Without it, fair sharing would rest on luck. With it, the check happens on the second, every second.

The failure pattern across all five drivers is one pattern, stated once here. The driver reports an error value in plain words, per section 7.3, with a code from section 16.3 where one fits. The system goes on without the device where it can, the network above all, and stops where it cannot: the disk at boot, per Chapter 23, and the screen at boot, because there is no report without the screen. A keyboard failure is reported on the screen, part of why the screen starts first. No driver hides a failure, retries in secret, or invents an answer the device never gave.

## 27.2 One driver at a time, tested on real PC

The title of this section means two things, and both are the design. One driver at a time is the build order and the start order. Tested on the real PC is the done condition each driver must meet. Take them in turn.

First, one driver at a time. The drivers are built one at a time: one driver is written, loaded, and tested until it passes, and only then does the next begin. The reason is the same reason section 13.1 ported the compiler one piece at a time: only one thing is new, so if a test fails, the new driver is the suspect, not the whole system. The start order at boot is the same shape, already on the record: section 23.2 starts the drivers one at a time, each finished before the next starts, because a driver started on top of a half-started driver inherits whatever is wrong underneath it.

The order is fixed here, following the boot's own reasons from section 23.2. The clock comes with the kernel from the start: section 23.2 set the clock up as the kernel's second setup act, before the drivers, because the kernel needs the count from the first moment. Then the four, in the boot's order. Screen first, so failures can be reported. Disk second, so files can be read: the shell's file, and every program after it, comes off the disk. Keyboard third, because the user types only after the prompt appears, and the prompt waits on the shell. Network last, because nothing at the prompt needs it. Clock, screen, disk, keyboard, network: five drivers, one order, for building and for starting alike.

Second, tested on the real PC. Each driver counts as done only when it has run on the target machine from section 3.1, part 5 of the done test in section 3.3: anything said to run on the target machine has been run there, or is marked plainly as waiting. Section 2.5 said it in shorter words: prove it on real hardware or it does not count. A driver is the one part of the system that cannot be proven anywhere else, because it talks to this machine's screen, this machine's disk, this machine's keyboard. So each driver has one test, run on the real PC, stated here.

The clock's test: the count matches a known time. The driver's count is read at a moment whose count is known, the way section 16.2 gave 1,794,700,800 for the start of 15 November 2026, and the two counts agree, second for second. The signal is part of the same test: a program that calls nothing runs, and the turn check still happens on the second, because the device called.

The screen's test: the screen shows the boot failure lines from Chapter 23 and the shell's prompt. The three failure lines are written at a place, one at a time, `Error 8: the kernel file is not a Riza file.`, the unknown-version line, and `Error 1: the kernel file was not found.`, and each shows whole, at the place it was written to. Then a good boot's prompt, `> `, shows at the first line and first column after a clear, and text written past the last line scrolls.

The disk's test: the disk reads what the boot reads. The size at a place returns the kernel file's size from its record at block 64, and read blocks brings back the kernel's file, whose first four bytes are R I Z A, and block 1344, with the top folder's entry reading whole from its start, the 35 bytes section 26.1 counted. A write is part of the test too: blocks written to free space read back the same bytes, in the order they were asked.

The keyboard's test: the keyboard hands back the keys of one typed command. The user types `list docs` and the Enter key at the prompt, 10 keys in all, and the driver hands the shell those 10 keys, in that order, one per ask, the newline last, and the command runs. Keys typed before the shell's first ask wait in the buffer and are handed back in order, none lost inside the 32 the buffer holds.

The network's test has two passes, and both are passes, stated plainly. On the machine's own network, with another machine answering, the driver sends bytes and hands back the bytes that arrive in answer. With no network there to reach, the driver starts, finds nothing, reports it as the normal answer it is, and the system runs on without it.

Say what a fail means. The driver goes back, the way section 3.3 fixes, and the next driver does not start until this one passes, because a driver built on a broken driver inherits the break. A disk driver tested while the screen driver is broken has no way to show its failures, and a keyboard driver tested while the disk driver is broken cannot trust the shell it is typed into, because the shell came off that disk.

Say what this chapter does not fix, stated plainly as the limit of a design chapter. No driver's code size is claimed, because the code is unwritten; Chapter 30 counts the drivers in the full system image. The exact device registers and wire details belong to each device's own facts, learned on the real PC during each driver's test. And no timing is claimed for any driver: how the drivers bear on the startup cost section 1.4 counted and the standard tasks in section 1.5 is Chapter 31's measurement, with Chapter 32 comparing the whole stack against today's.

The test suite grows with this chapter. Eight new tests cover the drivers. Three cover the shape: a driver loads through the loader's eight steps and its memory is recorded as kind 7; a second program asking for a busy device waits and is served in the order it asked; and a disk that will not answer gives code 2 while a network driver finding nothing reports a normal answer, not a failure. Five cover the real-PC tests, one per driver, the five tests stated above. That is 8 new tests, and the suite grows from 153 to 161: 153 plus 8 is 161. Each test runs on the target PC, the way section 2.5 demands.

This chapter adds no new routines to the core library. The keyboard read is the kernel's call, the clock's count reaches programs through `now`, and every driver's calls are the kernel's own, numbered with the kernel's calls in Appendix A, the way section 22.2 designed it. The library stays at 35 routines, and the suite grows to 161 tests.

The chapter counts as done when the five drivers run on the target PC from section 3.1: the clock's count matches a known time, the screen shows the failure lines and the prompt, the disk reads the kernel's file at its fixed place and the top folder's entry block, the keyboard hands back one typed command, and the network sends and receives or reports none. Until that run happens, the chapter's claims are marked plainly as waiting for the real-PC run, the way part 5 of the done test in section 3.3 requires.

Chapter 28 designs the services: time, network, and updates. The network service runs over the network driver, the way the filesystem service runs over the disk driver, and the time service runs over the clock, keeping the count the driver reads set to the world's time. Chapter 29 designs the security model, permissions in plain words, including what a program may ask a device to do, the data the kernel reads before a call. Chapter 30 builds the full system image and counts the drivers in it. Chapter 31 runs the drivers' tests on the real PC and measures their work, and Chapter 32 benchmarks the whole stack against today's, drivers included.

---

The drivers are five small modules, one per device, written in the same language and middle form, translated by the same translator, loaded through the loader's eight steps, running as part of the kernel with the kernel's own memory, kind 7. Each holds only its calls, its small stated state, and its device, which nothing else touches. The screen holds the display as a grid of 170 by 48 cells, 8,162 bytes of state with its cursor, and the print chain reaches it in 4 acts. The keyboard hands back one key per ask, a keystroke crosses 2 handoffs against 9 today, and the shell's read that Chapter 21 left open is settled. The disk knows blocks and nothing else, moving whole blocks in the order it is asked, while the filesystem service owns the records and the layout. The network sends bytes and hands back what arrived, and finding no network is a normal answer, not a failure. The clock reads the count `now` serves, and its device calls the kernel's attention once per second, so the turn check from section 24.2 happens on the second. A second program asking for a busy device waits its turn, in order. The drivers are built and started one at a time, clock, screen, disk, keyboard, network, each proven on the real PC before the next begins. No new routines join the library, and the suite grows from 153 tests to 161.
