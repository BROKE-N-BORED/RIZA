# Chapter 1: Where today's computing wastes effort

A modern computer is thousands of times faster than the machines of thirty years ago, and it carries thousands of times more memory. Yet a text editor can take seconds to open. A chat app can eat half a gigabyte of RAM. A one line script can take longer to start than to run. The hardware won, and the software gave the winnings back.

This chapter is the diagnosis. It names four kinds of waste in the way computers work today, shows each one with real numbers, and then sets down the ruler the rest of this book is measured against. Nothing here is a guess about the future. Every number in this chapter was either measured while writing it, or taken from a named published source. Where a number varies by machine, version, or workload, the text says so.

A note on the measurements. The timings and memory figures marked as measured in this chapter were taken on the Linux machine used to write this manual: a virtual machine with 4 cores and 8 GB of RAM, running Python 3.12.3 and Node 24.22.1. Timings are the median of 5 runs with the disk cache warm, meaning the files were already read once, so the numbers show the cost of the work itself rather than the cost of a cold disk. Memory figures are peak resident set size, the most physical memory the program held at any point, measured with a small harness built for this chapter and cross checked against the operating system's own accounting. Your numbers on the target PC, an HP All-in-One with a Ryzen 3 and a spinning hard drive, will be slower in absolute terms, especially on cold starts. The ratios are the point, and the ratios hold. Chapter 31 repeats these measurements on the real target machine.

## 1.1 Text in, parse, throw away, parse again

A program is stored as text. Text is characters a human can read: letters, numbers, punctuation. A computer cannot run text directly. Before anything happens, the text has to be parsed. To parse means to read the characters, split them into meaningful pieces called tokens (a word, a number, a bracket), and arrange those pieces into a structure the machine can follow, called a tree. Only then can the machine do the work the program describes.

Here is the waste: this conversion is done over and over, and its result is thrown away every time.

Take Python, the language this manual's own tooling is written in. When you run a Python program, the interpreter reads each source file, parses it, and turns it into an intermediate form called bytecode, which is a compact list of instructions for a small virtual machine inside the interpreter. Python caches that bytecode on disk in `__pycache__` folders as `.pyc` files, precisely because parsing is expensive enough to be worth avoiding. Even with the cache, every import still checks the source file's timestamp and size to make sure the cached bytecode is fresh. And the cache only helps for files that were run before. The first run of anything pays the full parse.

The same pattern repeats everywhere text is the storage format:

- A web server receives a request as JSON, which is text shaped into a standard form for data. It parses that text into objects, does its work, then turns its answer back into JSON text to send. The next request parses the same shapes all over again. The parsing produces nothing the user asked for. It is the price of using a human readable form as a machine form.
- A web page is HTML, CSS, and JavaScript, all text. A browser parses megabytes of it on every page load, builds its internal structures, draws the page, and throws the structures away when the tab closes. Browsers now cache compiled bytecode for scripts they have seen before, which is an admission of the same problem Python's `.pyc` files admit: parsing the same text twice is work nobody wants to pay for.
- A configuration file is read and parsed on every launch of the program that uses it. A build tool reads and parses its project files on every run. A shell reads and parses its startup files on every login.

None of this parsing is huge on its own. Parsing one small file takes a fraction of a millisecond. But it is paid on every run, every request, every page, on every machine, and it forces the whole parsing toolchain, the lexer that splits text into tokens, the parser that builds the tree, the error recovery that copes with malformed input, to be shipped inside every runtime and every browser. The work is also fragile: a single misplaced character in the text breaks the whole parse, so every parser carries error handling for mistakes a binary format would make impossible by construction.

The deeper point is about what text is for. Text is for humans. Machines work in binary, in numbers and fixed size records. Storing a program as text and converting it on every use is like keeping a book in a foreign language and re-translating it every time you open it. The translation is pure overhead. It adds nothing to what the program does. Chapter 2 names the alternative: binary first, text second. Chapter 9 defines the binary format that makes loading a matter of reading, not re-translating.

## 1.2 Layers on layers: app, runtime, OS, driver

A layer is a program that provides services to the layer above it and uses services from the layer below it. A typical desktop program today sits on four or more of them: the app itself, a runtime (the language implementation and libraries it runs on), the operating system (the kernel, the part of the system that owns the hardware and hands out memory, files, and time), and the drivers (small programs that talk to one specific piece of hardware, such as the screen or the disk).

Follow one keystroke through that stack. The keyboard hardware signals the driver. The driver hands the event to the kernel. The kernel hands it to the display server, the program that owns the screen and the keyboard. The display server hands it to the GUI toolkit, the library that draws windows and buttons. The toolkit hands it to the runtime. The runtime hands it to the app. The app decides a character was typed, and the answer travels all the way back down: app to toolkit to display server to graphics driver to screen. Each crossing copies data, checks types, and translates between one layer's idea of an event and the next layer's. No single crossing is slow. There are just a lot of them, and they happen for every key, every mouse move, every frame.

The clearest example of layer cost is Electron, the framework behind VS Code, Slack, Discord, and many other desktop apps. Electron lets developers write a desktop app with web technology, HTML, CSS, and JavaScript, by shipping a whole web browser, Chromium, plus the Node.js runtime, inside every app. The numbers show what that costs:

- An Electron hello world program, an empty window that does nothing, starts at around 130 MB on disk. Chromium alone accounts for roughly 120 to 150 MB of disk before a single line of the app's own code.
- VS Code, a text editor, is around 300 to 500 MB installed depending on version and platform. Microsoft's own documentation puts its disk footprint under 500 MB.
- One 2026 compilation of installed sizes put Slack near 650 MB and Discord near 360 MB.
- A freshly opened Electron window with nothing in it uses roughly 80 to 120 MB of RAM. A fresh VS Code window with no extensions uses roughly 300 to 400 MB in published measurements. One comparison put Sublime Text, a native editor written without a bundled browser, at about 22 MB of RAM at idle.

Install VS Code, Slack, and Discord, and you now have three complete copies of Chromium on your machine. Each is frozen at the version that was current when that app was built, each occupies its own disk space, each loads its own processes into memory, and each keeps its own cache. The three apps cannot share one browser engine because each ships its own private copy.

This is not a complaint about Electron's authors. Each layer in the stack solved a real problem when it was added: portability across operating systems, safety, developer speed. The waste is not that layers exist. It is that nobody removes them, and that every layer re-does work the layer below already did. Data gets buffered, copied, translated, and checked at each boundary. A string read from disk can be copied four or five times before it reaches the screen: from the disk cache to a kernel buffer, to a runtime buffer, to an app string object, to a GUI text object, to a texture on the graphics card. Each copy costs time and memory bandwidth, and none of the copies is visible to the user. They exist because each layer keeps its own private copy of the world in its own private format.

The layer problem is also a duplication problem. Every Electron app ships its own JavaScript engine. Every Python program ships its own interpreter. Every app bundles its own copy of common libraries rather than sharing one system copy, because sharing one copy means depending on a version the developer does not control. The result is a machine full of near identical code, each copy loaded separately, each copy maintained separately, each copy a separate thing to update when a bug is found.

Chapter 10 answers this with one shared middle form for the language, the tools, and the operating system, so that work done once is done once. Chapter 22 designs a kernel that is small on purpose, with no layers for the sake of layers.

## 1.3 Memory: garbage collection pauses and copies

A running program keeps its data in memory, and memory is managed in two regions with different rules. The stack is a small, fast region where the program keeps short lived values: the arguments to the function that is running right now, its local variables. When a function returns, its stack space is freed at once, with no decisions to make. The heap is the large region where the program keeps everything that must outlive one function call: objects, lists, strings, buffers. Heap memory is allocated when the program asks for it, and the hard question is when to free it. Free it too early and the program crashes. Never free it and the program grows until the machine runs out of memory.

A garbage collector is the standard answer to that question. It is a part of the runtime that finds heap memory nothing can reach any more and reclaims it, so the programmer never has to free anything by hand. It does this in two ways, often combined. Reference counting keeps a count on each object of how many other things point at it, and frees the object the moment the count hits zero. Tracing walks from the roots, the global variables and the live stack, follows every pointer it finds, marks everything it can reach as alive, and reclaims the rest. Python uses reference counting as its main method with a tracing collector as backup for cycles, groups of objects that point at each other and keep each other's counts above zero forever. Go and Java use tracing collectors.

The deal looks free. It is not. The collector must do the finding, and finding takes work the program would not otherwise do. That work shows up in three ways: pauses, CPU, and copies.

A pause happens because the collector needs a consistent picture of memory. If the program keeps changing pointers while the collector is tracing, the collector can free something that is still in use. The classic solution is to stop the world: freeze every thread of the program, do the collection, then let the program continue. Modern collectors work hard to keep those freezes short, but every tracing collector still stops the world at least briefly, and some stop it for a long time.

The best case today is Go. Go's collector runs mostly alongside the program, marking and sweeping on spare CPU capacity, and stops the world only twice per cycle. Published GC traces show those stops measured in tens of microseconds: one trace records stop times of 0.042 milliseconds and 0.038 milliseconds, with the concurrent marking work taking 1.12 milliseconds in between while the program kept running. That is genuinely good engineering. But the cost does not vanish. It moves. Go pays in CPU, because marking runs on cores the program could otherwise use, and it pays in memory, because with the default setting the heap is allowed to grow to about twice the live data before a cycle finishes. Memory is spent to buy time.

The rough end is Python. In a test published in February 2026 by Daniel Lemire, a small program that built a 1000 element linked list in a loop, deliberately stressing the cyclic collector, saw a worst case pause of 2.2 seconds on Python 3.12 on Linux, and 320 milliseconds on Python 3.14 on a Mac. The same algorithm in Go peaked at 33 to 50 milliseconds. Read the test fairly: it was built to stress the collector, and ordinary programs see smaller pauses. But the pause is unbounded by design, it arrives without warning, and it arrives exactly when the program is busy allocating, which is when it can least afford to stop. A third of a second is long enough for a human to notice. Two seconds is long enough for a user to decide the program has frozen.

History shows the same pattern at larger scale. In the 1990s, a full collection on a multi gigabyte Java heap could freeze a program for seconds. Modern Java collectors such as ZGC and Shenandoah aim for pauses under a millisecond even on very large heaps, and they largely achieve it, but they pay for it with extra CPU, extra memory, and a small check, called a barrier, on pointer operations throughout the program. Every generation of collector is cleverer than the last, and every generation pays a tax the program would not pay if memory were freed by the program's own structure instead of by a background detective.

Copies are the second cost. In most garbage collected languages, strings cannot be changed in place. They are immutable. Editing a string means allocating a new one and copying the old contents into it. Building a long string by adding one piece at a time, in the naive way, copies the whole string on every addition: adding 1,000 pieces one at a time copies on the order of 500,000 characters in total, because the first copy moves 1 character, the second moves 2, and so on up to 1,000. Real implementations add special cases to dodge the worst of this, and programmers learn to collect pieces in a list and join them once. Both facts are telling: the language creates a cost, and then the runtime and the programmer both work around it. The same copying shows up at every boundary where data changes form: turning objects into JSON text to send them somewhere, then parsing the text back into objects on the other side, is two conversions for one crossing, and section 1.1 already counted the parse.

Put together, the garbage collector trades programmer effort for machine effort. The programmer writes less bookkeeping code, and the machine pays in pauses, CPU, and memory. On a machine with 8 GB of RAM, the machine's share is the one that runs out. Chapter 6 builds memory management on a different deal: the program's own structure says when memory is done, in plain words the reader can follow, with no background collector and no pause that arrives unannounced.

## 1.4 Startup cost: what loads before anything runs

Startup is the time from the moment you ask for a program to the moment it can do useful work. Everything before that moment is overhead, and in many programs the overhead is larger than the work.

Here is what startup looks like on the machine used to write this manual, measured as the median of 5 runs with the disk cache warm:

| Program | Time to run and exit | Peak memory |
| --- | --- | --- |
| `/bin/true` (does nothing, exits) | 0.9 ms | 1.3 MB |
| C hello world (prints one line) | 1.3 ms | 1.3 MB |
| `ls --version` | 2.1 ms | 2.1 MB |
| `python3 -c "print('hi')"` | 28 ms | 10 MB |
| `python3 -c "import json"` | 35 ms | 11 MB |
| `node -e ""` (does nothing) | 114 ms | 61 MB |

Read the table from the top down. A program that does nothing takes about a millisecond to start, because the operating system still has to load it and hand it a process. A C program that prints one line takes 1.3 milliseconds and 1.3 MB of memory, and its binary is 16 KB on disk. Python doing the same job takes about 28 milliseconds and 10 MB. Node doing literally nothing, an empty program, takes 114 milliseconds and 61 MB. The work in all six rows is trivial. The difference is what loads before the work begins.

What loads is the runtime. A C program loads the C library, a few hundred kilobytes of shared code that nearly every program uses, and runs. Python loads an 8 MB interpreter binary, then its startup files, then the parts of its 55 MB standard library that startup requires. Node loads a 121 MB binary, because Node is a whole JavaScript engine, V8, plus its library, plus the runtime, all in one file. Importing one more module in Python, `json` in the table above, adds a few milliseconds as the loader finds the file, checks the bytecode cache, and parses or loads it.

The cost multiplies. A script that runs 100 times a day pays its startup cost 100 times. A build tool that shells out to a short script for each file pays the startup cost per file. A test suite that starts a fresh interpreter per test file pays it per file. Each payment is small. The sum is not.

Cold starts are worse, and the target PC makes them worse still. The numbers above were measured warm, with the files already in the disk cache. On a cold start, every file must be read from disk. The target PC has a spinning hard drive, and a spinning drive reads fastest in long sequential runs and slowest when it must jump between files. An interpreter startup is a sequence of small reads scattered across the disk: the binary, the loader, the libraries, the startup modules, the configuration. Each jump costs a seek, and seeks on a spinning drive take milliseconds each. A startup that takes 30 milliseconds warm can take several times longer cold, and the exact multiple depends on where the files happen to sit on the disk. This book does not quote a cold start number here, because it was not measured on the target machine. Chapter 31 measures it there.

Big applications pay the same tax at larger scale. Electron apps, from section 1.2, take about 2 to 4 seconds to cold start in published figures: the browser engine, the runtime, the app code, the extensions, and the cached bytecode all load before the first window can draw. A user who opens an editor ten times a day waits the better part of a minute a day for startup alone.

The principle is simple: a program should pay for what it uses, when it uses it. Today's programs pay for the whole runtime up front, before they know what the user wants. Chapter 15 designs startup so that a program does the least work possible before its first useful act, and Chapter 9 defines a binary format that loads with no re-parse, so that starting a program is mostly reading, not computing.

## 1.5 What we measure: size, RAM, speed, power

Everything before this section was a claim that today's computing wastes effort. Claims are cheap. This section defines the ruler the rest of the book is measured against, so that every later claim can be checked. Four numbers: size, RAM, speed, power. If a design in this book does not improve them, it does not ship. If it improves three and hurts the fourth, the book says so plainly.

### Size: installed bytes on disk

What is measured: the total bytes a user must download and keep to have the thing installed and working. That includes the program's own code, its runtime, its bundled libraries, and its data files. It does not include caches the program builds while running, but caches are reported separately where they matter, because a program that needs a gigabyte of cache to run fast is not a small program.

How it is measured: `du -sh` on the installed tree for the installed size, and the size of the downloaded package for the download size. Both numbers are reported, because a download is compressed and an install is not, and the two answer different questions: how much must come over the network, and how much must live on the disk.

The trap: counting only the main binary while the runtime hides elsewhere. A Python program's binary may be a few kilobytes of script, but it needs an 8 MB interpreter and a 55 MB standard library to run. An Electron app's own code may be a few megabytes, but it ships a browser engine with it. Size is always the whole thing, or the number is a lie.

### RAM: peak resident set size

What is measured: the physical memory the program actually occupies while running, at its highest point, and at steady state while idle with a typical document open. The technical name for physical memory occupied is resident set size, shortened to RSS. It is the number that matters on a machine with 8 GB of RAM, because physical memory is what runs out.

How it is measured: run the program under a harness that records peak RSS. The harness used in this chapter is a small C program that starts the target, waits for it, and reads the peak RSS the operating system recorded for the child. On Linux the same information is available from `/usr/bin/time -v`, which prints maximum resident set size, and from `/proc/<pid>/status`, where the field `VmHWM` is the peak RSS of a running process. Peak and idle are both reported, because a program can spike at startup and settle, or start small and grow as it works.

Two things RSS is not. It is not virtual memory, which is address space the program has claimed but may never touch. A program can claim gigabytes of virtual memory and use almost none of it, so virtual memory numbers are not reported in this book. And RSS is not additive across processes: a shared library loaded by ten processes is counted in all ten, so adding ten RSS figures double counts the shared parts. Where a total across processes is needed, the book says which measure was used and how shared pages were handled.

The trap: caches make the second run look cheaper. A program that loads its files once and keeps them in memory will show a higher RSS on a long run than on a short one, and a warm run will start faster than a cold one. Cold and warm are therefore reported separately, and the state of the machine, what else was running, is stated alongside the number.

### Speed: wall clock time to finish a defined task

What is measured: the time a real task takes from start to finish, as a person waiting for it would feel it. That is wall clock time, the time on the clock on the wall, as opposed to CPU time, which is the total time the processor spent working. The two differ when a program uses several cores at once: four cores working for one second is four seconds of CPU time and one second of wall time.

How it is measured: time the whole job from launch to result, median of at least 5 runs, on the target machine, with nothing else running. The tasks are defined per tool and fixed before measuring, so that the measurement cannot be tuned to flatter the result. The standard tasks used through this book are: cold start to first output, warm start to first output, and one real task per tool, for example listing 10,000 files, formatting a 1,000 line file, or searching a 10 MB text file for a word. Chapter 32 fixes the exact task list and the exact inputs, so that any reader can repeat the measurements.

Both wall time and CPU time are reported, because the difference between them is information. A program that finishes in one second of wall time by burning four seconds of CPU time has used more of the machine, and on a two core machine it has used cores another program needed. A program that finishes in one second of wall time on one second of CPU time did the same job with a quarter of the work.

The trap: one fast run proves nothing. Machines vary from run to run as other work, temperature, and disk state change. This book reports the median of repeated runs and also the slowest run, because the slowest run is what the user feels on a bad day. A design that is fast on average and slow one time in five is not a fast design.

### Power: energy used to finish the task

What is measured: the energy the machine uses to finish a defined task, in joules, or in watt hours for longer tasks. Energy is not the same as power. Power, in watts, is a rate, how fast energy is being used right now. Energy, in joules, is a total, how much was used altogether. A fast program can draw more watts and still use less energy, because it finishes sooner. A slow program can draw few watts and use more energy, because it runs longer. What the user pays for, in electricity and in battery life, is energy. So energy is the number.

How it is measured on the target PC: the practical method is a plug in power meter on the wall socket, measuring the whole machine over a fixed task, then subtracting the idle baseline measured the same way over the same length of time. The idle baseline matters, because the machine draws power even when doing nothing, and the task's cost is the extra energy above idle. Where the machine exposes energy counters, Intel calls them RAPL and AMD provides an equivalent on Ryzen, readable under `/sys/class/powercap` on Linux, those give finer detail per part of the chip. Whether the Mint PC exposes them is checked in Chapter 31; if it does, the book uses them, and if it does not, the wall meter is the method.

Be honest about the limits. Power is the hardest of the four to measure at home. A wall meter measures the whole machine, screen and disk included, so the number includes things the program did not cause. RAPL counters measure the chip, not the screen or the disk, so they miss part of the picture. The two methods give different numbers for the same task, and numbers taken by different methods must never be compared. Every power figure in this book therefore states its method alongside the number, and comparisons are only made between figures taken the same way on the same machine.

### The scoreboard

Every later chapter that claims an improvement reports a small table in this form:

| Task | Today's number | Riza's number | Machine | Method |
| --- | --- | --- | --- | --- |
| Cold start to first output | | | | |
| Peak RAM, same task | | | | |
| Installed size | | | | |
| Energy for the task | | | | |

Chapter 32 collects every table into one place, so the whole book's claims can be read at a glance and checked one by one.

### The rules for honest numbers

1. Same machine. A number measured on one machine is never compared with a number measured on another. The target machine is the HP All-in-One described in Chapter 3.
2. Same task. Both sides do the same work on the same input. A faster time on an easier task is not a win.
3. Median of repeated runs, with the slowest run reported too.
4. Cold and warm stated. A warm number is never presented as if it were cold.
5. Method stated. Every number carries the tool and the procedure that produced it, so a reader can repeat it.
6. The whole job measured. Startup counts. Shutdown counts. The work in between counts. Measuring only the fast middle of a job is cheating.
7. A loss reported as a loss. If a design is smaller but slower, the table shows both, and the text says which trade was made and why.

"Better" in this book means better on the four numbers together. A design that trades a small size win for a large speed loss is a loss, and the book will say so. A design that wins on all four is the goal, and the goal is stated in advance so that nobody, including the author, can move it after the fact.

---

The waste is real, it is measured, and it is the same waste in every layer: work repeated instead of done once, data copied instead of shared, memory scanned instead of freed by structure, and whole runtimes loaded before the first useful act. The next chapter names the idea that answers it.
