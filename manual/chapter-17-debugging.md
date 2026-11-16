# Chapter 17: Debugging from the start

Chapter 15 designed the startup and the shutdown: the loader's eight steps, from the first byte of the header to the first operation of main, and the finish routine's five steps, from the flush to the exit report. Chapter 16 designed the core library: the 28 routines every program calls, including log_write and log_read. This chapter designs what those routines write to, and what the programmer reads when something goes wrong: the debug log layer in every program, and the traces that fit the binary form. It is the last chapter of Part 4.

Debugging is usually bolted on after. A program breaks, and the programmer adds print statements, or attaches a debugger, or reads a crash report written by a system the programmer did not design. The evidence is reconstructed after the fact, and the reconstruction is guesswork. This chapter designs the other order: the debugging is built in from the start, part of the runtime, written as the program runs, so that when something goes wrong the evidence is already there.

Four promises from earlier chapters meet here. Section 2.4 promised that every program carries a debug log layer, a plain record of what it did and when. Section 7.3 said the layer records errors as they happen. Section 15.2 said the startup and the shutdown write to it too. Section 16.3 gave the layer its routines, log_write and log_read, and its rule: log the act, never the secret.

There are two sections. The first designs the debug log layer: where the log lives, how it is kept from growing without bound, how it is built into every program, and what it costs. The second designs the traces: records of what the program did, step by step, in the binary form the program runs in, mapped back to the text the programmer wrote.

One warning, stated up front the way Chapters 4 through 16 stated their own. This chapter designs the debug log layer and the traces. It does not build either one. No layer exists yet, no trace exists yet, and no claim in this chapter says one does. The chapter claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own examples, counted by hand. No time is claimed anywhere in this chapter, and the measured proof comes in Chapter 31, on the real PC.

## 17.1 Debug log layer in every program

Start with the job, in one sentence. The debug log layer is the plain record every program keeps of what it did and when, written as it runs, readable by the user, and small enough to cost almost nothing.

The layer is part of the runtime, not an add-on the programmer must remember. The programmer does not set it up, open its file, close it, or free its memory. The loader sets it up, the finish routine closes it, and the program's own code only writes to it, through log_write, when it has something worth recording. A program that never calls log_write still has a log, because the startup and the shutdown write to it, the way section 15.2 described.

Say exactly when the layer is set up and closed, because the order matters. The loader reads the header, runs the first four validity checks, and only then sets the layer up: a file that fails the checks is not a program, and gets no log, and the loader's plain-words report is the record, the way section 9.4 designed it. Once the first checks have passed, the loader sets up the buffer and writes the first line, which file was started; after the last two checks pass, it writes the second line, the checks and how many passed. The finish routine writes the ended line first, and then the five steps run: the flush carries the log's waiting lines to the disk, the close releases the log file, and the free releases the buffer.

Where the log file lives is part of the design, because a record the user cannot find is a hidden record, and section 2.4 forbids hidden work. The log file lives in a folder named `logs`, beside the program's file, and its name is the program's name with `.log` added: the program `hello` keeps its log in `logs/hello.log`. The user reads it with log_read or with any tool that reads text, because the log is text, one line per entry, the way section 16.3 said. If the folder cannot be written, the program runs with logging off and says so in the exit report, instead of failing.

Say plainly what the log holds, and what it does not hold. The log holds what the program did and when, the errors it hit, and the choices it made: which file was opened, which check failed, which branch was taken. It does not hold secrets, passwords, or anything the user would not want written down. A program that checks a password logs that the check happened, not the password. The rule is the programmer's, stated in section 16.3 so it cannot be missed: log the act, never the secret. And say honestly what the layer cannot do: it writes what it is given, and it cannot tell a secret from a plain word.

The log is kept small two ways: a buffer, and a size limit. The buffer first. The layer holds new lines in memory, in a buffer of 1,024 bytes, and writes them to the disk in one run, when the buffer fills or when the program ends. The buffer is one arena, the way section 6.2 designed arenas: the lines are allocated together and freed together, in one act, at shutdown. The layer owns the buffer, the way section 6.1 requires. The buffer is also why the log suits the spinning drive: section 3.1 said the drive reads and writes fastest in long runs, and the buffer turns many small lines into one long write.

One kind of line skips the buffer. Section 7.3 said the layer records errors as they happen, and as they happen means at once, not when the buffer fills. When the program hits an error, the error line goes straight to the disk, in its own write, before the program goes on. A program that ends on an error might never reach the flush, and a program killed from outside gets no clean shutdown at all, the way section 15.2 said, so an error line held in the buffer could be lost with the program. The cost is one small write per error, and errors are rare. Work one error line: the program from section 15.2 opens `notes.txt` and finds it missing, and the error line is `1794700800 error 1 the file notes.txt does not exist`, 52 characters plus the newline, 53 bytes: the time, 10 digits, a space, `error`, a space, `1`, a space, and the message, 33 characters, the message section 16.3 counted.

Now the size limit, which is what keeps the log from growing without bound. The limit is 64 KB, 65,536 bytes, and it is a limit on the file: the log file never holds more than 65,536 bytes. When a write would take the log past the limit, the oldest lines go, and the newest stay, so the log is always the recent past. The rule for how many go is fixed: the layer drops the oldest lines, whole lines, never half a line, until the log holds no more than half the limit, 32,768 bytes, and then the new lines are written. Dropping to half is what makes the limit cheap: the rewrite happens once per 32,768 bytes of new lines, not once per line. The user can raise or lower the limit, and can turn logging off entirely, because section 2.4's test asks whether the user can turn it off. Logging off costs zero: nothing is set up, nothing is written, nothing is freed.

Now the cost, and the three questions from section 2.4, because the layer is the no hidden work rule in practice. In memory, the layer costs the buffer, 1,024 bytes, freed at shutdown. On the disk, the log file costs at most 65,536 bytes, the limit. In writes, the cost is the lines the program writes, in as few runs as the buffer allows. In time, the chapter claims nothing, and Chapter 31 measures it on the real PC. Can the user see it? The log file sits beside the program, in plain text. Can the user measure what it costs? The file's size is the cost on disk, the buffer's 1,024 bytes is the cost in memory. Can the user turn it off? Yes, and off means nothing: no buffer, no file, no writes. The layer is the runtime's version of the rule section 5.4 stated for the language: nothing is hidden from the user.

Now work one log through, step by step, with every byte counted. The program is the hello loop from section 4.4, as a complete file of 72 bytes, the way section 9.2 counted it. The program never calls log_write, so its log holds only the lines the startup and the shutdown write: three lines. Each line starts with the time it was written, from `now`, the way section 16.3 designed it: the count of seconds since the start of 1970, and at the start of 15 November 2026 the count was 1,794,700,800, ten digits, the number section 16.2 gave. The program runs in well under a second, so all three lines carry the same time.

| Line | Text | Bytes |
| --- | --- | --- |
| 1 | `1794700800 started hello` | 24 characters + 1 newline = 25 |
| 2 | `1794700800 checks passed 6 of 6` | 31 characters + 1 newline = 32 |
| 3 | `1794700800 ended well` | 21 characters + 1 newline = 22 |
| Total | | 79 bytes |

Check the counts by hand. Line 1 is 10 plus 1 plus 7 plus 1 plus 5, 24, plus the newline, 25. Line 2 is the time, `checks`, `passed`, `6`, `of`, and `6`, with a space between each: 31, plus the newline, 32. Line 3 is 10 plus 1 plus 5 plus 1 plus 4, 21, plus the newline, 22. The total is 25 plus 32 plus 22, which is 79 bytes.

Read the total honestly, because section 1.5's rule 7 says to report the number that loses too. The log, 79 bytes, is bigger than the program file, 72 bytes. The reason is the fixed lines: every run writes a started line and an ended line, whatever the program does, and the hello loop does almost nothing. The whole log fits in the buffer, 79 bytes in 1,024, so it is written in one write, at shutdown: 1 write, 79 bytes, and nothing before it.

Now the size limit, worked through with the hello loop's log. The log file holds one run's lines after another. Each run is 79 bytes, and the limit is 65,536 bytes. 65,536 divided by 79 is 829 runs, with 45 bytes to spare: 829 times 79 is 65,491, and 65,536 minus 65,491 is 45. The 830th run would take the log past the limit, so the oldest lines go: 1,243 lines, 32,731 bytes, leaving 32,760 bytes, no more than half the limit. Then the new run's 3 lines are written, 79 bytes, and the log holds 32,839 bytes. The oldest runs are gone, the newest 415 runs are there, and the next cut comes after 413 more runs. Say honestly where the cut falls, because the rule drops lines, not runs: the cut falls between lines, never inside one, so the oldest run left in the log may be missing its first line. The newest runs are always whole.

The log layer is designed now: a plain record in a file the user can find, held in a buffer the layer owns, kept under a size limit by dropping the oldest lines, written by the startup and the shutdown as well as by the program, and turned off entirely when the user asks. The next section designs the record that goes deeper: the trace, which says which steps the program ran, in what order, with what values.

## 17.2 Traces that fit the binary form

Start with the job, in one sentence. A trace is a record of what the program did, step by step: which operations ran, in what order, with what values.

The trace goes deeper than the log. The log records what the program did, at the level of acts: it opened a file, it hit an error, it ended. The trace records which operations ran, at the level of the middle form: LOAD, ADD, STORE, JUMP, in the order they ran, with the values that changed. The log answers what happened. The trace answers how.

Say first what the debugger does not deal with. A broken file never reaches the debugger: section 9.4 catches it before it runs, with six validity checks, and the loader reports it in plain words. And a mistake the compiler can see never reaches the debugger either: section 11.1 catches the wrong kind, the unchecked error, the value with no owner, before the program runs, and refuses the program. The debugger deals with what the compiler could not see: the values at run time, the choices the program made with real data, the loop that ran one time too many.

The trace is made only when the programmer asks for it. Tracing every program on every run would be hidden work, and section 2.4 forbids hidden work: a record nobody asked for, written on every run, growing without bound. So the programmer asks, through the build tool or the debugger, and the run is traced; an untraced run writes nothing extra, and costs nothing extra.

The trace fits the binary form, and that is the whole design. It records middle-form operations by position, not by text line, because the program that runs is the binary, not the text. The positions are the ones section 4.4 counted: the hello loop's 14 operations sit at positions 0, 2, 4, 6, 8, 9, 11, 13, 14, 16, 18, 19, 21, and 23 in the code section. A position is two bytes, so the trace covers code sections up to 65,535 bytes, the range of two bytes from the table in section 5.1. A program with more code than that needs a wider position, and the trace format grows to meet it, in a new version, the way section 9.1's version byte lets the file format grow.

The trace file is binary, and it starts with a magic, the way section 9.1 and section 14.2 start their files: R I Z T, in bytes 52 49 5A 54. The T says trace, the way the P in section 14.2 said package. After the magic comes a version byte, the value 1. After the version come the entries, and there are two kinds.

A step entry says one operation ran. It holds a kind byte, the value 1, saying this entry is a step; the operation's position, two bytes; and the change, if the operation changed anything. Most operations change nothing the trace needs to record, because the debugger can work it out from the program file: a LOAD_CONST puts a number on the pile, and the number is in the code; an ADD takes two values off the pile, and the debugger knows what was on the pile, because it replays the steps in order. So a step that changes nothing visible is three bytes: the kind, and the position. Two kinds of step carry a value. A STORE carries the name's number, one byte, and the new value, sized to the name's type, because the value is new and the program file does not hold it. A library call that hands back a value from outside, the time from `now` or a line from a file, carries the value it handed back, because the debugger cannot replay what it cannot see. The trace records what changed, not everything.

A repeat entry says a pass ran again. It holds a kind byte, the value 2, saying this entry is a repeat; the position the pass starts from, two bytes; and the values the pass stored, one per STORE in the pass, in the order the stores happened, each as a name number and a new value. The debugger knows how many values to read, because it has the previous pass's steps. The rule is fixed: when the program jumps back to a position it has already traced, and the pass that follows runs the same steps, in the same order, as the pass before it, the trace writes one repeat entry, carrying only what changed. When a pass runs different steps, because a branch inside the loop went the other way, the trace writes the pass in full, and that pass becomes the shape the next pass is measured against. The trace never guesses that two passes match: it compares the steps, and it writes a repeat only when they match exactly.

The trace is read by the debugger, and the debugger shows the programmer what happened in the text view, the way section 8.3 designed the editor to show the program: the text the programmer wrote, with the values beside it. The trace maps back to the text through a line map, a file the compiler writes: one line number per operation, two bytes each, in code order. The debugger walks the code section, the way section 9.4 walks it, and pairs each operation with the next entry in the map. Line 0 means no text line: END belongs to no line, and neither does any step the compiler added that came from no line the programmer wrote.

The line map lives in its own file, next to the program file, named after the program with `.map` added: the program `hello` keeps its map in `hello.map`. It is a separate file, not a fourth section in the program file, for the reason section 2.4 gives: a program that will never be debugged should not carry the map. The map is written only when the program is built for debugging. The map file starts with a magic, R I Z M, in bytes 52 49 5A 4D, and a version byte, the value 1, and after them it holds the line entries and a removed list. The trace file lives with the log, in the `logs` folder beside the program file, named after the program with `.trace` added: `logs/hello.trace`. The debugger reads three files: the program file, for the code and the name table; the map file, for the positions to lines; and the trace file, for what ran.

Now the optimizer, because section 12.1 changes the middle form, and the trace must map back through those changes. The optimizer inlines, folds, and removes, so the steps that run are not the steps the compiler first wrote. The map is written after the optimizer runs, so it maps the steps that run to the text lines they came from, and the optimizer carries the line numbers with the operations as it works. The rules are fixed. A kept step carries the line it came from. A folded step keeps the line of the operation that computed the value: when `2 + 3` folds to `5`, the constant keeps the line of the ADD. An inlined step keeps the line in the function's body where it was written, not the line of the call. A removed step has no map entry, because it has no position in the code that runs, and the removed list names it instead: for each removed step, the text line it came from, two bytes, and the operation it was, one byte, the operation's code from Appendix A. The removed list starts with a count, two bytes, saying how many steps were removed.

One honest note about the round trip, because section 12.1 already raised it. Section 8.2 promised that text to binary and back produces the same text, byte for byte, and that promise is about the compiler's middle form, before the optimizer runs, tested in section 11.3. The map is about the optimized form, the form that runs, and it maps back to the text the programmer wrote, but it does not restore the removed steps to the text.

Now work one trace through, step by step, with every byte counted. The program is the hello loop, traced for two passes. The setup runs first: LOAD_CONST 0 at position 0, then STORE name 0 at position 2, putting 0 into `count`. Then the first pass: the test at positions 4, 6, 8, and 9, and the body at positions 11, 13, 14, 16, 18, 19, and 21. The second pass runs the same steps, in the same order, so the trace writes one repeat entry, carrying the one value that changed.

| Entry | Position | Operation | Value | Bytes |
| --- | --- | --- | --- | --- |
| Step | 0 | LOAD_CONST 0 | | 3 |
| Step | 2 | STORE name 0 | `count` = 0 | 5 |
| Step | 4 | LOAD name 0 | | 3 |
| Step | 6 | LOAD_CONST 3 | | 3 |
| Step | 8 | LESS_THAN | | 3 |
| Step | 9 | JUMP_IF_FALSE 23 | | 3 |
| Step | 11 | LOAD_STR entry 1 | | 3 |
| Step | 13 | PRINT | | 3 |
| Step | 14 | LOAD name 0 | | 3 |
| Step | 16 | LOAD_CONST 1 | | 3 |
| Step | 18 | ADD | | 3 |
| Step | 19 | STORE name 0 | `count` = 1 | 5 |
| Step | 21 | JUMP 4 | | 3 |
| Repeat | 4 | Pass from position 4 | `count` = 2 | 5 |

Check the byte counts by hand. A step entry with no value is 3 bytes: the kind, 1 byte, and the position, 2 bytes. A STORE entry is 5 bytes: the kind, the position, the name number, and the new value, 1 byte, because `count` is a 1-byte number. The repeat entry is 5 bytes: the kind, the start position, and the one changed value. The setup is 3 plus 5, 8 bytes. The first pass is 11 steps: 10 at 3 bytes and 1 at 5, 35 bytes. The second pass is one repeat entry, 5 bytes. The two passes shown are 8 plus 35 plus 5, 48 bytes, for 13 step entries and 1 repeat entry, standing for 24 steps run. The jump's outcome is not stored: the debugger sees the next entry is position 11, not 23, and knows the jump at position 9 was not taken.

The run goes on the same way. The third pass is another repeat entry, 5 bytes, carrying `count` = 3. Then the test runs a fourth time, and this time it fails, 3 is not less than 3, so the jump at position 9 is taken, and the trace writes the four test steps in full, 12 bytes. Then END at position 23, 3 bytes. The whole trace is 20 entries: 2 for the setup, 11 for the first pass, 1 repeat for the second pass, 1 repeat for the third pass, 4 for the last test, and 1 for END. The entries are 68 bytes: 8 plus 35 plus 5 plus 5 plus 12 plus 3. With the 4-byte magic and the 1-byte version, the trace file is 73 bytes. Read the total honestly, because section 1.5's rule 7 says to report the number that loses too: the trace file, 73 bytes, is one byte bigger than the program file, 72 bytes. Without the repeats, the trace would be 40 step entries, 128 bytes of entries, 133 with the magic and the version. The repeats save 60 bytes: 73 against 133.

Now the loop that runs a thousand times, the case the repeat entry is designed for. Take a loop like the hello loop's, counting to 1,000 instead of 3. The count is a 2-byte number now, because a 1-byte number counts to 255, and 1,000 needs two bytes, the way the table in section 5.1 says. A STORE step is 6 bytes, and a repeat entry is 6 bytes. The first pass is written in full, 36 bytes: 10 steps at 3 bytes and 1 STORE at 6. The other 999 passes are one repeat entry each, 5,994 bytes in all. With the setup, 9 bytes, and the last test and END, 15 bytes, the entries are 6,054 bytes, and the trace file is 6,059 bytes with the magic and the version. Without repeats, the same run would be 36,024 bytes of entries, 36,029 with the magic and the version. The repeat design saves 29,970 bytes.

Now the line map for the hello loop. The program's text is four lines: line 1 is `count = 0`, line 2 is `while count < 3:`, line 3 is `print("hello")`, and line 4 is `count = count + 1`.

| Position | Operation | Text line |
| --- | --- | --- |
| 0 | LOAD_CONST 0 | 1 |
| 2 | STORE name 0 | 1 |
| 4 | LOAD name 0 | 2 |
| 6 | LOAD_CONST 3 | 2 |
| 8 | LESS_THAN | 2 |
| 9 | JUMP_IF_FALSE 23 | 2 |
| 11 | LOAD_STR entry 1 | 3 |
| 13 | PRINT | 3 |
| 14 | LOAD name 0 | 4 |
| 16 | LOAD_CONST 1 | 4 |
| 18 | ADD | 4 |
| 19 | STORE name 0 | 4 |
| 21 | JUMP 4 | 2 |
| 23 | END | 0 |

The JUMP at position 21 belongs to line 2, because it is the loop's jump back to the test. END belongs to line 0, because it came from no line the programmer wrote. The map is 14 entries at 2 bytes each, 28 bytes. The optimizer changed nothing in the hello loop's middle form, so the removed list is empty, and the map file is the 4-byte magic, the 1-byte version, the 28 bytes of line entries, and the 2-byte removed count saying 0: 35 bytes in all.

The debugger reads the trace and the map together, and shows the programmer the two passes in the text view, the text lines with the values beside them:

Line 1, `count = 0`: `count` becomes 0.
Line 2, `while count < 3:`: 0 is less than 3, true.
Line 3, `print("hello")`: prints `hello`.
Line 4, `count = count + 1`: `count` becomes 1.
Line 2, `while count < 3:`: 1 is less than 3, true.
Line 3, `print("hello")`: prints `hello`.
Line 4, `count = count + 1`: `count` becomes 2.

The programmer reads the bug in the words they wrote, not in the numbers the machine ran. The positions, the kind bytes, and the repeat entries are underneath, and the debugger shows them on request, the way section 8.3 requires.

Now work the map through the optimizer, because the hello loop's map has no removed steps. The program is the one section 12.1 worked through: 19 operations in the plain form, 3 in the optimized form, still printing 9. The final program is LOAD_CONST 9 at position 0, PRINT at position 2, and END at position 3. LOAD_CONST 9 keeps line 2: the 9 was computed by the ADD on line 2, with the 5 from line 5 and the 4 from line 6 folded into it. PRINT keeps line 7. END keeps line 0. The map is 3 entries at 2 bytes each, 6 bytes: lines 2, 7, and 0.

The removed list holds the 17 steps the optimizer took away: every plain step except PRINT and END. Count them by the line they came from.

| Text line | Steps removed | Count |
| --- | --- | --- |
| 2, `total = first + second` | LOAD first, LOAD second, ADD, STORE total | 4 |
| 3, `return total` | LOAD total, RETURN | 2 |
| 4, `spare = 99` | LOAD_CONST 99, STORE spare | 2 |
| 5, `limit = 2 + 3` | LOAD_CONST 2, LOAD_CONST 3, ADD, STORE limit | 4 |
| 6, `answer = add(limit, 4)` | LOAD limit, LOAD_CONST 4, CALL add, STORE answer | 4 |
| 7, `print(answer)` | LOAD answer | 1 |
| Total | | 17 |

Check the count: 4 plus 2 plus 2 plus 4 plus 4 plus 1 is 17. The removed list is the 2-byte count, 17, plus 17 entries at 3 bytes each, the line and the operation's code: 2 plus 51 is 53 bytes. The map file is the 4-byte magic, the 1-byte version, the 6 bytes of line entries, and the 53 bytes of removed list: 64 bytes in all.

The suite grows with the layer and the traces. Section 13.2 left the suite at 34 tests, and section 16.3 grew it to 63. This chapter adds 11: six for the log layer, and five for the traces. The log layer's tests are the hello loop's log, 3 lines and 79 bytes; log_read returning the lines in order; the limit, a log past 64 KB dropping to 32,768 bytes or less; logging off, writing nothing and creating no file; the error line, on the disk before the program ends; and the log file living in `logs` beside the program. The traces' tests are the hello loop's trace, 20 entries and 73 bytes, mapping to the 40 steps that ran; the repeat entries, replayed, giving the same steps as the full pass; the map, turning each traced position into the right text line; the map through the optimizer, the 3 kept steps on lines 2, 7, and 0, and the 17 removed steps named; and an untraced run, writing nothing. The suite goes from 63 tests to 74: 63 plus 11 is 74. Each test runs on the target PC, the way section 2.5 demands.

The chapter counts as done when the layer and the traces run on the target PC: the hello loop's log written and read back, the error line on the disk before the program ends, the trace written and mapped to the text, the map through the optimizer naming the 17 removed steps, and the times measured in Chapter 31. That is part 5 of section 3.3: anything the book says runs on the target machine has been run there, or is marked plainly as waiting for the real-PC run.

Part 4 is complete. Chapter 15 designed the startup and the shutdown, Chapter 16 designed the core library, and this chapter designed the debugging that is built into every program from the start. Chapter 20 builds the developer tools, including the debugger that reads the traces this chapter designed. Chapter 31 measures the log layer and the traces on the real PC, and Chapter 32 benchmarks the whole stack against today's, the way section 2.5 requires.

---

Debugging in Riza is built in from the start: every program carries a debug log layer, a plain record of what it did and when, and every traced run writes a trace that fits the binary form. The log file sits in a `logs` folder beside the program, in plain text, held in a 1,024-byte buffer the layer owns, kept under a 64 KB limit by dropping the oldest lines, so the log is always the recent past. The hello loop's log is 3 lines and 79 bytes, written in one write at shutdown, and an error line goes straight to the disk the moment it happens. The log holds what the program did and never a secret, and the user can read it, measure what it costs, and turn it off. The trace records middle-form operations by position, in step entries and repeat entries: the hello loop's full run is 40 steps in 20 entries and 73 bytes, and the thousand-pass loop is 6,059 bytes against 36,029 without repeats. The line map sends each kept step back to the text line it came from and names the steps the optimizer removed. The suite grows to 74 tests, and Chapter 31 measures the layer and the traces on the real PC.
