# Riza: A Better Way to Compute

Riza is a better way to compute, way more efficient than today's standard, built from the root. Not a patch on top of what exists. A compressed language, a binary format, a compiler, tools, utilities, and an operating system, all sharing one form from top to bottom.

Target machine: 8GB Ryzen 3 Mint PC, CPU only. Hard $0, offline first.

This document is the roadmap for the full manual on building Riza from scratch. 7 parts, 34 chapters, plus appendices. The chapters are not written yet. This roadmap is the reference the manual gets built from.

## Part 1: Foundations

1. Where today's computing wastes effort
   - 1.1 Text in, parse, throw away, parse again
   - 1.2 Layers on layers: app, runtime, OS, driver
   - 1.3 Memory: garbage collection pauses and copies
   - 1.4 Startup cost: what loads before anything runs
   - 1.5 What we measure: size, RAM, speed, power
2. The core idea
   - 2.1 Prediction is compression
   - 2.2 Binary first, text second
   - 2.3 One shared form from top to bottom
   - 2.4 Small core, no hidden work
   - 2.5 Prove it on real hardware or it does not count
3. Rules and limits for this build
   - 3.1 Target machine: 8GB Ryzen 3 Mint PC, CPU only
   - 3.2 Hard $0, offline first
   - 3.3 How a chapter counts as done

## Part 2: The Compressed Language

4. The dictionary: short codes for common work
   - 4.1 Frequency ranking: most used gets shortest code
   - 4.2 One byte ops, two byte ops, the long tail
   - 4.3 Names and symbols: intern once, reuse
   - 4.4 Worked example: same program, text vs dense form
5. Types and data
   - 5.1 Numbers, text, true/false
   - 5.2 Lists, records, files
   - 5.3 How data sits in memory for cache
   - 5.4 What the language refuses to hide
6. Memory without a garbage collector
   - 6.1 Ownership in plain words
   - 6.2 Arenas: allocate together, free together
   - 6.3 Stack, heap, and static
   - 6.4 Out of memory: what happens, in what order
7. Functions, flow, and errors
   - 7.1 Calling and returning
   - 7.2 Branches and loops
   - 7.3 Errors as values, not crashes
8. The human view: readable text form
   - 8.1 Text syntax
   - 8.2 Text to binary and back, exact round trip
   - 8.3 What the editor shows, what it hides

## Part 3: The Binary and the Compiler

9. The binary format
   - 9.1 File header and version
   - 9.2 Code section, data section, dictionary section
   - 9.3 Loading: no re-parse, straight to run
   - 9.4 Checking a file is valid before running it
10. The shared middle form
    - 10.1 Why one middle form for language, tools, and OS
    - 10.2 Middle form ops and layout
    - 10.3 Middle form to native, step by step
11. The bootstrap compiler
    - 11.1 The smallest compiler that works
    - 11.2 Built first in an existing language
    - 11.3 Tests for every opcode
12. The optimizer: only what pays
    - 12.1 Dead code, constants, inlining
    - 12.2 What we skip on purpose
13. Self hosting: the compiler builds itself
    - 13.1 Porting the compiler into the new language
    - 13.2 Proving both compilers produce the same output
14. Build tool and packages
    - 14.1 One command builds
    - 14.2 Package format and versions

## Part 4: Runtime and Core Library

15. Startup and shutdown
    - 15.1 What runs before main
    - 15.2 Clean shutdown, every time
16. Core library
    - 16.1 Text, lists, files
    - 16.2 Time, math, random
    - 16.3 Errors and logging
17. Debugging from the start
    - 17.1 Debug log layer in every program
    - 17.2 Traces that fit the binary form

## Part 5: Tools and Utilities

18. File tools
    - 18.1 List, copy, move, delete
    - 18.2 Search inside files
19. Text tools
    - 19.1 View, edit, compare
    - 19.2 Dense text handling
20. Developer tools
    - 20.1 Formatter and checker
    - 20.2 Tester: run only what changed
21. The shell
    - 21.1 Commands as library calls, not separate programs
    - 21.2 Scripts in the same language

## Part 6: The Operating System

22. Kernel design: small on purpose
    - 22.1 What the kernel does, what it does not do
    - 22.2 No layers for the sake of layers
23. Boot: from power on to prompt
    - 23.1 Bootloader
    - 23.2 First process
24. Processes and scheduling
    - 24.1 Starting and stopping
    - 24.2 Fair sharing on 2 cores
25. Memory in the kernel
    - 25.1 Who owns what
    - 25.2 Protection without waste
26. Filesystem
    - 26.1 Layout on disk
    - 26.2 Crash safety in plain words
27. Drivers: screen, keyboard, disk, network
    - 27.1 Driver shape: small modules, same language
    - 27.2 One driver at a time, tested on real PC
28. Services: time, network, updates
29. Security model
    - 29.1 Permissions in plain words
    - 29.2 What untrusted code can and cannot do

## Part 7: Proving It, Shipping It, Growing It

30. The full system image
    - 30.1 What goes in, what stays out
    - 30.2 Building the image from scratch
31. Tests on the real PC
    - 31.1 Boot test, tools test, daily use test
    - 31.2 What counts as a fail
32. Benchmarks vs today's stack
    - 32.1 Size, boot time, RAM, speed, power
    - 32.2 Score before, score after, every time
33. Install, update, recovery
    - 33.1 Install on the Mint PC
    - 33.2 Updates that cannot brick the machine
    - 33.3 Recovery when something breaks
34. What comes after v1
    - 34.1 What to add next, what to never add
    - 34.2 Keeping the core small as it grows

## Appendices

- A. Opcode dictionary v0.1
- B. Binary file format spec
- C. Middle form spec
- D. Glossary in plain words
- E. Checklists: chapter done, system done
- F. Hardware notes for the Mint PC
