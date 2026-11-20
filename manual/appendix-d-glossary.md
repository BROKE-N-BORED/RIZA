# Appendix D: Glossary in plain words

This glossary holds the book's own words, defined the way the book uses them, in plain words. Every word here is a word some chapter gave a precise meaning of its own, and the definition below is that chapter's meaning restated, nothing more. The entries stand in alphabetical order, so a reader who meets a hard word anywhere in the book can find it here without knowing where it was defined.

This is not a general computing dictionary. A word the book uses in its ordinary sense is not here, and no definition here adds anything to the design: a glossary entry explains the chapters, it never extends them. Where a chapter pointer is given, it names the chapter that fixed the meaning.

**arena.** One stretch of heap memory set aside for a group of values that live and die together. The program asks for the arena once, hands out pieces of it to the group's values, and frees the whole stretch in a single act when the group is done. Chapter 6.

**block.** The unit the disk is read and written in. A read takes whole blocks and a write writes whole blocks, never anything smaller. In this book a block is 4,096 bytes, and the whole disk is counted in blocks. Chapter 26.

**bootloader.** The small program that starts the kernel. The firmware runs it from its fixed place at the very start of the disk, it reads the kernel's file at that file's fixed place, and its job ends when the kernel starts. It is the one program stored as native code, because nothing is running yet that could translate it. Chapter 23.

**bootstrap compiler.** The first compiler, written in an existing language (Python, already on the target machine) so the build can start from the machine as it stands. It exists to produce the first Riza compiler, and it comes down once the compiler builds itself. Chapters 11 and 13.

**cache.** A small, fast store inside the processor that holds copies of data and instructions the processor has just used, so reaching them again is quick. The book designs data to sit together so that one trip to memory fills the cache with values that will be used together. Chapter 5.

**code.** In this book, usually short for error code: the number inside an error value that says which kind of failure happened. Code 0 means no error, codes 1 to 127 are the system's own (nine standard ones are fixed in Chapter 16, among them 1 for a file that does not exist, and Chapter 29 added 10 for an act permission forbids), and codes 128 to 255 are left for programs to name their own failures. The word also appears inside operation code; see opcode.

**cold start.** Starting a program when none of its files have been read yet, so everything must come from the disk. On the target machine's spinning drive this is the slow case, and the book always reports cold and warm runs separately so the two are never mixed. Chapter 1.

**compiler.** The tool that turns what the programmer writes into the binary file the machine runs. In Riza it reads the text form once, checks it, and writes the middle form into the file, so the machine that runs the program never has to do that work again. Chapter 11.

**core.** A worker inside the processor: the part that actually runs instructions. The target machine has 2 cores, so it can truly do two things at once, and the whole design is built inside that ceiling. The book also uses "the core" for the small center of the system that everything depends on, the kernel and the few services every program needs, which Chapter 2 keeps small on purpose.

**core library.** The shared routines every program in the system can call, written once and kept as part of the system itself, not as a package a program installs. A program pays only for the routines it actually calls, at the moment it calls them. The library holds 39 routines. Chapter 16.

**dense form.** The stored form of a program, where operations are short codes and names are numbers pointing into the name table. It is the form on the disk and the form the machine loads, and it is far smaller than the same program written out as text. Chapter 4.

**dictionary.** Two things, and the book uses the word for both. First, the language's shared codebook: the numbered list of operations, with the shortest codes given to the most used operations, shared by every program and changed only in the open, one version at a time (Chapter 4, Appendix A). Second, the dictionary section inside each binary file, which holds that file's name table, its function mapping, and its comments (Chapter 9, Appendix B).

**done test.** The five-part test a chapter must pass before it counts as done: plain words, every number measured or sourced, every claim checkable, the scoreboard filled in where a chapter claims an improvement, and anything said to run on the target machine either run there or marked plainly as waiting. Chapter 3. The checklists in Appendix E are this test written out for use.

**driver.** A small module that talks to one piece of hardware, a screen, a keyboard, a disk, the network, the clock, so the rest of the system never has to know how that device works. A driver runs as part of the kernel, and a program reaches its device only through the kernel and the driver, never directly. Chapter 27.

**entry point.** The position in a file's code section where the program starts. It is written in the file's header, and it is where the loader jumps when the program begins. Appendix B.

**error value.** How this book reports failure: not a crash, but an ordinary value the program receives and must deal with. It holds a code saying which kind of failure it is and a message in plain words saying what happened, and the program chooses to handle it, pass it up to whoever called it, or give up cleanly. Chapter 7.

**firmware.** The program built into the machine's hardware by its maker. When the power comes on, the firmware wakes the machine, checks the hardware is there, and runs the bootloader. It is not part of Riza, and Riza does not control it. Chapter 23.

**fixed place.** A counted stretch of blocks at the start of the disk, reserved for one system file: the bootloader, the kernel, the shell, and each of the four services has one, and the top folder's entry holds a single block among them. A fixed place is found by its position alone, with no filesystem needed, which is why the machine can start before any filesystem exists. Chapters 26 and 30.

**folder record.** A folder's own record of what it holds: for each entry, the name, the kind, the size, the date, and where the file's blocks sit. The folder records are the truth about which files exist; everything else the filesystem keeps is rebuilt from them. Chapter 26.

**free record.** The filesystem service's list of which blocks in the general space are free, kept as runs with a start and a length. It is not stored on the disk as a second copy of the truth: the service rebuilds it from the folder records every time it starts. Chapter 26.

**function mapping.** The part of a file's dictionary section that says where each function's code starts, one entry per function, holding the function's name number and its starting position in the code section. It is how a call by number finds the code it calls. Appendix B.

**garbage collector.** Today's usual answer to freeing memory: a background worker inside a runtime that searches memory for what nothing uses any more and reclaims it. It costs pauses, processor time, and extra memory, and Riza has none: memory here is freed by the program's own structure, under ownership. Chapters 1 and 6.

**general space.** The rest of the disk after the fixed places, running from block 1921 to the end, where ordinary files and folders live, named by folder records. Chapter 30.

**guarded act.** An act a program may not do unless its entry in the permission record allows it. The list is fixed: starting a program, stopping one, reading or writing outside its own folder, using a device beyond the screen and keyboard it started with, opening a network talk, setting the time, and running an update. Chapter 29.

**heap.** The large region of memory where values live that must outlive one function call: lists, file contents, records a program is working on. Heap memory is freed by its owner, at the moment the owner is done with it, with no scan and no delay. Arenas live on the heap. Chapter 6.

**hidden work.** Anything the machine does that the user did not ask for, cannot see, and cannot turn off: background indexing, telemetry, an update check on a schedule the user cannot see. Riza forbids it. If the machine is busy, the user can ask why and get an answer in plain words. Chapter 2.

**image.** The full system image: one file holding the whole system, the fixed places and the general space files together, counted block by block. Installing Riza means writing this image to the disk. Chapter 30.

**instruction cache.** The part of a core's cache that holds instructions rather than data. On the target machine each core holds 64 KB of it, and code small enough to fit in it runs without waiting on memory, which is one reason the book keeps code small. Chapters 5 and 24.

**interning.** Storing a name once, in the name table, and using its short number everywhere else. A name used fifty times is spelled out once; every use after that costs one byte. Chapter 4.

**kernel.** The small program at the center of the system. It owns the hardware, hands out memory and takes it back, starts and stops programs, guards the devices so only one program uses each at a time, and keeps the record of what is running. It does only what must live in the center; everything else is a module or a service outside it. Chapter 22.

**kernel call.** A call a running program makes to the kernel, naming what it wants by number: start a program, stop one, ask for memory, give memory back, read the clock, get secret bytes, and the rest. There are ten, numbered in Appendix C, and a refused or failed call hands back an error value like any other failure.

**layer.** A program that provides services to the program above it and uses services from the one below it. Today's stack has many, app, runtime, operating system, driver, and each one copies and re-translates the work passing through it. Riza's answer is one shared form instead of many layers re-doing each other's work. Chapter 1.

**library call.** The middle form's one way of calling the core library: a single step that names a routine by its number, with the values it needs placed on the pile first. It has no operation code of its own in the dictionary; its numbers are the routine numbers in Appendix C.

**loader.** The part of the system that starts a program. It reads the file's header, runs the six validity checks, reads the sections, places them in memory, translates the middle form, sets up the program's memory, and jumps to the entry point: eight steps, and loading is reading, never re-parsing. Chapter 15.

**log.** The plain record every program keeps of what it did and when, written as it runs into a file the user can find and read, and small by design. Its rule is: log the act, never the secret. Chapters 16 and 17.

**long tail.** The rare operations at the far end of the dictionary, used so seldom that they are reached through marker codes for the price of extra bytes. The common operations get the one byte codes; the long tail is where the rest live without crowding them. Chapter 4 and Appendix A.

**middle form.** The list of simple steps that sits between what the programmer writes and what the machine runs: loads, stores, arithmetic, jumps, calls, each step doing one small thing. Riza has one middle form shared by the language, the tools, and the operating system, so work done once is done once and no boundary re-translates it. Chapter 10 and Appendix C.

**module.** A self-contained piece of the system with one job, living outside the core. A module can be added, removed, or replaced without touching the core, and a program that does not need it does not pay for it. The drivers are modules. Chapter 2.

**name table.** The part of a file's dictionary section that holds the program's names and strings, each spelled out exactly once, as a length byte followed by the characters. Everywhere else the program refers to a name by its entry number. A file holds at most 255 entries. Chapters 4 and 9.

**native code.** The processor's own instructions, the form the hardware actually runs. Programs in Riza are stored as middle form and translated to native code when they load; the bootloader is the one exception, stored as native code because it runs before any translator exists. Chapter 10.

**offline.** Short for offline first, one of the build's hard rules: the system works with no network at all. No download to start, no license check, no activation. If a network is there, the system can use it for things the user asks for; it is a bonus, never a requirement. Chapter 3.

**opcode.** Short for operation code: the number that stands for one operation in the dictionary, written in the code section where the operation happens. The most used operations get the shortest codes, most of them one byte. The book usually just says operation; the final numbers are in Appendix A.

**operand.** The number that rides right after an operation code and says what the operation works on: which name, which constant, which position to jump to. An operation and its operands together make one step of the middle form. Chapter 10 and Appendix A.

**optimizer.** The stage of the compiler that improves the middle form, but only where the saving is proven by the code itself: removing steps that never run, computing constants at build time, and writing a small function into its one caller. Everything it cannot prove, it skips on purpose. Chapter 12.

**ownership.** The rule that replaces the garbage collector: every value has exactly one owner, the part of the program responsible for its memory, and when the owner is done with the value, the owner frees its memory right then. Ownership can be handed over to a new owner, or lent out as a loan that ends, but there is always exactly one owner. Chapter 6.

**package.** A program plus its description, its name, its version, and what it needs to run, carried together in one file so a program can travel from one machine to another with nothing left to guess. A package is installed by a tool, never run directly. Chapter 14.

**parse.** To read text, split it into meaningful pieces, and arrange those pieces into a structure the machine can follow. Today's systems parse the same text on every run and throw the result away; in Riza the parse happens once, in the editor, when the program is written. Chapter 1.

**permission record.** The file named `permissions` in the top folder, holding one entry per program: the program's name and one byte of acts saying which guarded acts it is allowed. Only the user changes it, through the kernel; no program may read it or write it, and the kernel reads the entry fresh at each check so a change takes effect at once. Chapter 29.

**pile.** The middle form's stack of working values. A load puts a value on the pile, an operation takes values off the top and puts its result back, and a store takes the top value off into a slot. When the form is translated, the pile lives on the machine's own stack. Chapters 4 and 10.

**program number.** The number the kernel gives a running program so programs can be told apart. Numbers are handed out in order starting from 1, the shell is number 1, 0 means no program, and a number is never handed to a second program while the machine runs. Chapter 24.

**record.** A value made of named fields sitting at fixed positions, like a name, an age, and a score in one small block whose size is the sum of its fields. Chapter 5. The book also uses the word for kept lists of entries, the kernel's record of running programs, the free record, the permission record, and folder records; each of those has its own entry here or in its chapter.

**round trip.** The guarantee about the text form: text turned into binary and back produces the same text, byte for byte, with nothing lost and nothing silently changed. Chapter 8.

**routine.** One named piece of shared work inside the core library. Each routine says what it takes, what it hands back, and what it does when it fails, and a program calls it by its number. Appendix C lists all 39 with their numbers.

**run.** A stretch with no break in it. In memory, a run is one stretch with one owner, one start, and one length, and the kernel's memory record is a list of runs (Chapter 25). On the disk, a file's blocks sit in runs, one long run where they can (Chapter 26). The same shape serves both, on purpose.

**scoreboard.** The book's table for reporting a measured comparison: the task, today's number, Riza's number, the machine, and the method, with losses shown as losses. Chapter 1 sets the form and Chapter 32 collects the tables. There is no single total score; each row stands on its own.

**secret bytes.** Bytes a program asks the kernel for when it needs numbers no one can guess, for secrets. The kernel gathers them fresh at the moment of the ask from the machine's own noise, the timings of the disk, the keys, and the clock, mixed so no one source decides any byte. There is no seed, nothing is stored, and the bytes are never logged. Chapter 29.

**service.** A program the system keeps for one shared job: the filesystem, time, network, and updates are the four. A service is started by the kernel when its job is first needed, owns the records of its job, ends when it is done, and is reached by other programs only through the kernel. A service works when it is asked; it does not run on its own schedule in the background. Chapter 28.

**shell.** The first program the kernel starts, and the program the user types commands into. In Riza a command is not a separate program: the shell holds its tools as routines and calls them the way a program calls the library, so a command does not pay a fresh startup cost. Scripts in the shell are small programs in the same language. Chapter 21.

**size record.** The first 8 bytes of a fixed place: one number holding the size of the file that lives there, with the file's bytes right after it. A size of 0 means no file is there. When a file at a fixed place is replaced, its bytes are written first and the size record last, so the place never names a file that is not there whole. Chapter 26.

**slot.** The fixed place in static memory where one named value lives. Each name in a program has one slot, sized to the name's type, and a store puts the name's current value into it. Chapter 10.

**stack (today's stack).** All the layers of software a program sits on in an ordinary system today: the app, its runtime, the operating system, the drivers, each with its own formats and its own copies. It is the thing Riza is measured against in Chapter 32. This is not the memory stack; for that, see pile.

**static memory.** The memory that holds a program's values for as long as the program runs: its named slots and the values from its file's data section, placed once when the program loads. Chapter 6.

**talk.** One conversation between a program and another machine, held through the network service. A program opens a talk by naming the machine, sends and receives on it, and closes it, and the service keeps every talk's bytes apart from every other talk's, in the order they arrived. Chapter 28.

**text form.** The human readable view of a program: the names, lines, and words the programmer writes and reads. It is generated from the binary form whenever it is needed, and the binary form is generated from it when the program is saved, with an exact round trip between them. The programmer's experience is text; the machine's truth is binary. Chapter 8.

**trace.** A deeper record than the log: which operations the program ran, in what order, with what values, kept in the binary form the program runs in and mapped back to the text the programmer wrote, so a failure can be followed step by step. Chapter 17.

**translator.** The part of the loader that turns the middle form into native code, one step at a time, mechanically, when a program loads. It does not optimize, does not check types, and does not manage memory; each of those is someone else's job, done once, before it runs. Chapter 10.

**turn.** One program's stretch of running on a core while others wait. Programs waiting for a core stand in a list in the order they asked, and when a turn ends the program at the front of the list runs next. The turn's length is fixed at 1 second, counted with the clock, so a turn always lasts at least one full second. Chapter 24.

**type tag.** The one byte at the start of each value in a file's data section, saying what kind of value follows and how many bytes it takes, so the loader can walk the section value by value. The tag numbers are assigned in Appendix B.

**version byte.** The byte in a file's header that says which dictionary the file was written against. It is also the file format version, because the dictionary and the format move together. A loader that meets a version it does not know does not guess: it says so in plain words and stops. Chapter 9.

**waiting list.** The kernel's list of programs waiting for a core, kept in the order they asked. The program that has waited longest runs next, and no program jumps the list. Chapter 24.

---

This glossary defines nothing new. Every entry restates a chapter, and if a definition here and a chapter ever disagree, the chapter wins. It adds no tests and no routines: the test suite stays at 217 tests, and the core library stays at 39 routines, the closing counts Chapter 34 left as the first version's state.
