# Appendix F: Hardware notes for the Mint PC

These notes gather the target machine in one place: the machine section 3.1 fixed, part by part, and every hardware fact the later chapters leaned on, each with the design decisions that fact drove. A reader who wants to check a decision against the machine it was made for can do it here, without hunting through thirty four chapters for the one sentence where a cache size or a drive habit did its work.

The honesty rule for this appendix is the book's rule, applied to the machine itself. Every fact here is a fact a chapter stated, and the chapter is named beside it. A fact no chapter stated is not added, even if it is true of the real machine sitting on the desk, because a note invented here would quietly become a design input nobody argued for. These notes describe the book's machine: the machine the chapters described, no more and no less. Chapter 1's numbers are not here, because they were measured on the machine used to write this manual, a different machine, and rule 1 of section 1.5 forbids mixing the two. This appendix is about the target alone.

## The machine as it stands

Chapter 3, section 3.1, fixed the machine in one table, and the whole book is built inside it:

| Part | What it is |
| --- | --- |
| Machine | HP All-in-One |
| Processor | AMD Ryzen 3 3250U, 2 cores, 4 threads |
| Graphics | Radeon Vega 3, integrated |
| Memory | 8 GB DDR4 |
| Storage | 1 TB spinning hard drive |
| Screen | 1366 by 768 |
| Operating system | Linux Mint |
| Python | 3.12.3 |
| GPU | None |

Linux Mint and Python 3.12.3 are the starting ground, not part of the finished system: the machine as it stands today is where the build begins, and the bootstrap compiler of Chapter 11 is written in the Python already installed, on the machine as it is, because the build has to start from what is already there (Chapter 3). The rest of these notes take the parts one at a time.

## The processor and its 2 cores

The processor is an AMD Ryzen 3 3250U with 2 cores and 4 threads (Chapter 3). A core is a worker inside the processor, and 2 cores means the machine can truly do two things at once, and no more. The 4 threads mean each worker can juggle two lines of work, switching between them quickly, but switching is not doing: four lines of work on two cores still take the time of two workers. So the ceiling on doing many things at the same time is two, and Chapter 3 states the consequence plainly: a design that needs eight cores to be fast is not a design for this machine, and a design that is fast on two cores is fast everywhere.

That ceiling shaped the design in named places. Chapter 24's scheduling is fair sharing on exactly 2 cores, not a scheduler written for a server: two cores run two programs at once, the rest wait their turn, and every waiting program gets a turn. The build itself lives under the same ceiling: the compiler runs on this machine, on two cores, and a compile that takes an hour is a compile nobody runs twice, so the tools are built to be fast on this machine rather than on a machine with more cores (Chapter 3, and Chapter 11 after it). And the ceiling reaches down into the language: the dictionary's codes are measured in bytes and not in bits because a processor reads bytes, not bits, and unpacking a bit packed code on every operation would cost time on every operation, the wrong trade on a machine with two cores and no GPU (Chapter 4).

## The caches

A cache is a small, very fast memory built into the processor, holding the data and instructions the program is using right now, so the processor reaches them without waiting on the main memory. Section 5.3 gives the target machine's caches from AMD's own spec page for the Ryzen 3 3250U:

| Cache | Size | Shared or per core |
| --- | --- | --- |
| L1 | 192 KB in all, 96 KB per core: 32 KB for data, 64 KB for instructions | Per core |
| L2 | 1 MB in all, 512 KB per core | Per core |
| L3 | 4 MB | Shared by both cores |

The cache moves data in runs called lines, and on this chip a line is 64 bytes, from CPU-World's spec listing for the 3250U (Chapter 5). One line holds sixteen 4 byte numbers, or eight 8 byte numbers, so a single load can bring sixteen useful values, or one useful value and sixty bytes of dead space, depending entirely on how the data is laid out. Read against the main memory, the fast memory is tiny: the L3 cache is 4 MB out of 8 GB, one part in 2,048, and one core's L1 data cache is 32 KB out of 8 GB, one part in 262,144 (Chapter 5).

The decisions those sizes drove:

- Data the program uses together sits together in memory, so one load brings a whole run of useful values instead of one value and a lot of dead space. That is section 5.3's design rule, and the layouts of Chapter 5 are built to it.
- The kernel's code on the common path, the code that runs on every kernel call, is sized to fit the 64 KB instruction cache on each core, 65,536 bytes, so the path a call takes stays in the fastest cache the chip has (Chapter 22). Chapter 31 tests the fit on the real PC, and a common path that does not fit is a fail of Chapter 22's design, not a reason to want a bigger cache (Chapter 31).
- The kernel's memory record is sized to live in the 32 KB data cache on each core: at 21 bytes an entry, 1,560 entries fit inside 32,768 bytes, so the hand out walk reads the fastest cache the chip has long past any scene the book works through (Chapter 25).
- A program that keeps its core keeps its cache warm, which is why Chapter 24's sharing prefers to give a waiting program the core it ran on last, when that core is free: a program that moves cores starts cold (Chapter 24).
- Loop unrolling is skipped on purpose, and the cache is part of the reason: writing a loop out four times makes the code bigger, and bigger code takes more room in the 64 KB instruction cache, pushing other code out, certain bytes paid for unmeasured time (Chapter 12).

## The memory: 8 GB

The machine holds 8 GB of memory, which is 8,589,934,592 bytes (Chapters 3 and 25). Every megabyte of it has an owner, because the whole system shares it: the operating system, the programs, and the graphics, since integrated graphics takes its share from the same memory instead of having its own (Chapter 3). On this machine memory is the scarcest thing there is, and Chapter 3 names what that forbids: nothing in Riza is allowed to treat memory as free. One fresh editor window on today's stack can take about a twentieth of the whole machine's memory before a single file is open, and on 8 GB that kind of appetite is the difference between a system that runs and a system that crawls (Chapters 1 and 3).

The decisions the 8 GB drove:

- Chapter 6 builds memory management that accounts for every byte, and Chapter 25 carries the same rule into the kernel: the kernel owns the machine's memory, all 8 GB of it, and hands it out in runs, keeps the record of who has what, and takes it back when a program is done (Chapter 25).
- The record's shape is set by the size itself: a run's start and its length are 8 bytes each, because 4 bytes count to 4,294,967,295, which names only the first 4 GB of the machine, and the machine holds twice that. Eight bytes name every byte in the machine with room to spare (Chapter 25).
- Running out of memory is a designed case, not a surprise: the graphics takes its share, the system takes its share, and what is left is the programs', and a program that asks for more than what is left gets a clean no it can see and handle (Chapters 6 and 25).

## The spinning drive

The storage is a spinning hard drive holding 1 TB, and 1 TB the way drives are sold is 1,000,000,000,000 bytes; the book uses the sold number, stated plainly (Chapter 26). The drive is read and written in blocks of 4,096 bytes, a size Chapter 26 fixed as a design choice, and counted in blocks the disk is 244,140,625 blocks exactly, numbered from 0 to 244,140,624.

The drive's one habit drives more of this design than any other fact in these notes. A spinning drive reads fastest in long sequential runs, one stretch of data after another, and slowest when it must jump between files scattered across the disk. Each jump costs a seek, and seeks on a spinning drive take milliseconds each (Chapters 1 and 3). Room is not the problem: the drive holds a full terabyte. Speed is the problem, and the shape of the reading is what sets the speed.

The decisions that habit drove:

- Files in Riza are laid out to be read in long runs, few files instead of many, and a program starts by reading, not by hunting (Chapter 3). Chapter 26's layout carries it onto the disk: a file is one long run of blocks where it can be, and a second run is the exception, not the shape.
- The binary format of Chapter 9 is built for that shape, and Chapter 15 designs startup around it, because an interpreter startup is a sequence of small reads scattered across the disk, each one paying a seek, which is why cold starts hurt most on this machine (Chapters 1, 3, and 15).
- Inside a file, the same shape holds: the name table is one run, read from start to finish with no jumps, and the code section is another run, so the loader reads a file in a small number of long reads (Chapter 4).
- Cold and warm runs are always reported separately, never mixed, because the disk cache hides the drive's cost on the second run: a warm number presented as cold would be a lie about this drive (Chapters 1 and 32).

## The screen: 1366 by 768

The screen is 1366 pixels wide and 768 tall, a small screen by today's standards, and Chapter 3 makes it a rule: every window in Riza must fit and be usable at 1366 by 768, with text readable at that size, or it does not ship. A window designed for a bigger screen is a window the user cannot use, and if a window cannot fit on the screen it was built for, the window is wrong, not the screen (Chapter 3).

The decisions the screen drove:

- The editor earns its space: what it shows must be worth the room it takes, and it does not fill the screen with panels the programmer did not ask for, because the text is the work and the work gets the screen (Chapter 8).
- The text viewer's page is 40 lines, a number chosen to fit this screen with room for the prompt, and the user can raise or lower it (Chapter 19).
- The screen driver holds the display as a grid of character cells, 8 pixels wide and 16 tall: 170 columns by 48 rows, 8,160 cells, and 8,162 bytes of driver state with the cursor (Chapter 27).
- The benchmark scoreboard is sized to the screen: its 19 rows fit on one screen at 1366 by 768 and can be read in one sitting, because a board too big to read is a board nobody checks (Chapter 32).

## The clock

The machine always has a clock, and the design leans on that plain fact. The core library's `now` routine hands back the current time as a number of seconds since the start of 1970, and it does not fail, because the system always has a clock (Chapter 16).

The decisions the clock drove:

- Scheduling is counted with it: a program's turn on a core is counted with `now`, so the turn's length is a clock count, stated in the open, and the count is set so a turn always lasts at least one full second (Chapter 24).
- The clock device itself calls the kernel's attention once per second, as each second passes, and that signal is how the kernel runs its turn check on the second, every second, even for a program that calls nothing and waits for nothing (Chapter 27, settling the question Chapter 24 left open).
- The clock is set up early: it is the kernel's second setup act at boot, before the drivers, because the kernel needs the count from the first moment (Chapter 27, following Chapter 23).
- The time service keeps the count set to the world's time, asking the kernel to write the count to the clock device when the user asks it to check (Chapter 28).
- The clock is one of the machine's noise sources: the secret bytes the kernel gathers for a program are mixed from the timings of the disk, the keys, and the clock, the timings no program can predict or replay (Chapter 29).

## The devices

The machine's hardware is reached through five drivers, one per device: the screen, the keyboard, the disk, the network, and the clock (Chapter 27). The drivers run as part of the kernel, and a program reaches its device only through the kernel and the driver, never directly. Under them sits the firmware, the program built into the machine's hardware by its maker, which wakes the machine when the power comes on and runs the bootloader (Chapter 23).

What the chapters fixed about the devices themselves:

- A program starts with the screen and the keyboard. Under the default permission entry, a program may print to the screen and read keys from the keyboard when the user runs it in the open, and every device beyond those two is a guarded act (Chapter 29).
- The disk and the network hardware get no direct word from a program at all: they are reached, when they are reached at all, through the services and the filesystem, under their own guards (Chapter 29).
- The keyboard hands back one key per ask, and the disk knows blocks and nothing else, moving whole blocks in the order it is asked (Chapter 27).
- The network driver knows bytes and nothing else: no names, no addresses, no other machines. What a connection is belongs to the network service above it. Arriving bytes are held in a buffer of 4,096 bytes, the disk's block size, and when the buffer is full, arriving bytes are not kept, stated plainly as the limit (Chapter 27).
- Finding no network is a normal answer, not a failure. The driver starts, finds nothing to reach, the kernel logs it in plain words, and the system goes on without the network, because nothing at the prompt needs it and section 3.2 fixed that the system works with no network at all (Chapter 27).

## The energy counters question

Power is one of the book's five measures, and how it can be measured on this machine is a hardware question the book deliberately leaves open until the real PC answers it. Section 1.5 named the two methods. The practical one is a plug in power meter on the wall socket, measuring the whole machine over a fixed task, minus the idle baseline measured the same way. The finer one is the chip's own energy counters: Intel calls them RAPL, AMD provides an equivalent on Ryzen, readable under `/sys/class/powercap` on Linux, giving detail per part of the chip (Chapter 1).

Whether the Mint PC exposes those counters is checked in Chapter 31 and recorded, so Chapter 32 knows its method before it needs it (Chapter 31). Chapter 32's power method follows that check, in that order: the counters if they are there, read the same way on both sides; if they are not, the wall meter minus the idle baseline; and if neither is at hand on the day, the power rows read not measured, with the reason. No meter is bought to fill the row, because the budget is zero and stays zero (Chapters 32 and 3). The two methods measure different wholes, the meter the whole machine, screen and disk included, the counters the chip alone, so a number from one is never compared with a number from the other (Chapter 1).

## What the design does not use

The machine's graphics hardware plays no part in the design. The table in section 3.1 lists the Radeon Vega 3, integrated, and GPU: none. There is no separate graphics chip to hand work to, no graphics compute to lean on, and no hardware acceleration to assume. The integrated Vega 3 draws the screen, and that is its job. Every calculation in the system, from compiling to searching to drawing a window, runs on the two cores (Chapter 3). Chapter 4's byte codes lean on the same fact: the design trades a little size for simplicity on a machine with two cores and no GPU. Chapter 3 calls the limit a simplification as well as a limit: one processor to understand, one thing to measure, and no second chip with its own memory, its own drivers, and its own failure modes.

Two more things the design leaves aside, both stated in the chapters. The 4 threads are not 4 workers: the design counts the machine's workers as 2, because switching between lines of work is not doing them (Chapter 3). And the network is never required: the system works with no network at all, and a network that is there is a bonus the user can ask to use, never a requirement (Chapters 3 and 27).

## Closing

These notes add no fact and no requirement: they gather the machine the book was designed for, and if a note here and a chapter ever disagree, the chapter wins. They add no tests and no routines: the test suite stays at 217 tests, and the core library stays at 39 routines, the closing counts Chapter 34 left as the first version's state.

With this appendix, the manual is complete: the roadmap, thirty four chapters, and six appendices.
