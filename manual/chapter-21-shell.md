# Chapter 21: The shell

Chapter 18 built the file tools, which handle files whole and search inside them: list, copy, move, delete, and search. Chapter 19 built the text tools, which show, change, and compare the text inside a file: view, edit, and compare, and the dense form for text. Chapter 20 built the developer tools, which the programmer runs on programs: the formatter, the checker, and the tester, with the debugger named among them. This chapter builds the place the tools are run from: the shell, the program the user types commands into. Part 5 closes here, with the tools and the shell that runs them, written in Riza, on the runtime Parts 3 and 4 designed.

The shell's job is one sentence long. It runs commands. A command is a name the user types, with arguments after it, and the shell does what the name says. In Riza, a command is a library call, not a separate program. The shell holds the tools inside it as routines it calls, the way a program calls the core library. And a script, a file of commands, is a small program in the same language, checked and run the way any program is checked and run.

There are two sections. The first designs commands as library calls: how the shell reads a line, finds the routine by name, and calls it, and what that saves against the startup cost every command pays today. The second designs scripts in the same language: how the shell runs a file of commands as a program, and what a script can do that a typed command cannot.

One warning, stated up front the way Chapters 4 through 20 stated their own. This chapter designs the shell. It does not build it. No shell exists yet, and no claim in this chapter says one does. The chapter also claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own examples, counted by hand. No time is claimed anywhere in this chapter, because no shell has been measured on the target machine, and Chapter 3 forbids numbers without a method. The measured proof comes in Chapter 31, on the real PC.

## 21.1 Commands as library calls, not separate programs

Start with what a shell does today, because the waste is the reason for the design. Section 1.2 described the layers a keystroke travels through, each one doing its own work and passing the result on. A command travels through layers of its own. The user types a command, and the shell starts a whole new program for it: the shell finds the program file on the disk, the loader reads it, the program starts, runs, and shuts down, and the shell waits for it to finish. Then the user types the next command, and the whole sequence runs again, from the start, for the next program. Every command pays the full startup cost, and the cost is not small. Section 1.4 measured it: the smallest program in the table, `/bin/true`, which does nothing at all, costs 0.9 milliseconds and 1.3 megabytes to start, and a Python command costs 28 milliseconds and 10 megabytes before it does any work of its own. The startup cost is paid once per command, however small the command is. Ten commands pay it ten times.

Section 15.1 named what that cost is made of, in the loader's eight steps: read the header, run the first four validity checks, read the sections, run the last two checks, place the sections, translate the middle form, set up the pile and the static slots, and jump to the entry point. A shell today pays those eight steps, or its own system's equivalent of them, once per command. A session of ten commands pays them ten times. The work is the same every time: the same checks, the same translation, the same setup, for a program that may do one small thing and exit.

In Riza, the shell pays the startup cost once. The shell is a program, and it is loaded once, when it starts, the way section 9.3 described loading: the file is read, the checks run, the sections are placed, and the program runs. After that, the shell holds the tools inside it. The file tools from Chapter 18, the text tools from Chapter 19, and the developer tools from Chapter 20 are routines the shell calls, the way a program calls the core library from Chapter 16. A command is a call. The shell does not start a program to list a folder. It calls the list routine, which is already loaded, already checked, already translated, sitting in the shell's own program. The loader's eight steps run once, when the shell starts, not once per command.

Say how the tools reach the shell, because a routine cannot live in two places at once, and the honest answer matters. Each tool's work is one routine, written once, in one source file. The build tool from section 14.1 joins that routine into the shell's program, and it joins the same routine into the tool's own program file, for when the tool runs without the shell. One source, two programs: the routine is written once, checked once, and tested once, and it is reached two ways. The cost of that, stated plainly the way rule 7 of section 1.5 requires, is that the routine's bytes live twice on the disk, once in the shell's file and once in the tool's own file. The shell's file is bigger than any one tool's file, because it holds all the tools. The shell holds all the tools in memory for the whole session, even the ones the user never types. And a session of one command pays the full shell startup for that one command, which is more than running the tool on its own would cost. The saving is in the session, not in the single command, and the chapter counts it honestly below.

The shell works as a loop, and the loop has six steps, in order.

First, print the prompt. The shell prints a short prompt, two characters, `> `, with print from section 16.1, so the user knows the shell is waiting. The prompt is two characters and one call, and it is counted in the worked example below.

Second, read a line. The shell reads one line of text from the keyboard, up to the newline. Say plainly what the chapter does not name here: no core-library routine is named for reading from the keyboard, because the keyboard is a device, and devices are the drivers' work, which Chapter 27 designs. The read adds no call to the worked example's count, because there is no routine to count it against. The line the user typed is text, and the shell holds it until the command is done.

Third, split the line. The shell splits the line into pieces with split from section 16.1, dividing at the spaces. The first piece is the command name. The rest are the arguments. Splitting is dividing text at spaces, not parsing: the shell does not turn the command text into a tree, and it does not turn it into a program. It divides a line into words.

Fourth, find the routine by name. The shell holds a table of commands, one entry per command, and it looks the name up in the table. The table is described below.

Fifth, call the routine with the arguments. The shell hands the routine the list of arguments from the split, and the routine takes what it needs with get from section 16.1. A command is a call the way section 7.1 described a function call: the caller hands over values, the routine does its work, and the routine returns. The routine is stored once in the shell's file, the way section 7.1 said a function is stored once and called as often as needed, and typing the command ten times calls the same routine ten times without loading anything again.

Sixth, report the result. The routine returns a value or an error, and the shell reports it in plain words. A command that worked reports what it did. A command that failed reports the error, with the reason, in plain words, and the shell goes on to the next command. Then the loop starts again at the first step, and the shell prints the prompt.

The command table is the shell's memory of what it can do. It holds one entry per command: the name, in the shape section 4.3 fixed, a length byte followed by the characters, and one byte holding the routine number, which says which routine the name calls. A count byte stands first, saying how many entries the table holds. The table is data, not code, and it lives in static memory, the way section 6.3 described: loaded once with the program, held for the whole run, freed at clean shutdown. The table holds 14 commands.

| Command | Tool | Designed in |
| --- | --- | --- |
| list | List a folder | Section 18.1 |
| copy | Copy a file | Section 18.1 |
| move | Move a file | Section 18.1 |
| delete | Delete a file | Section 18.1 |
| search | Search inside files | Section 18.2 |
| view | Show a file's text | Section 19.1 |
| edit | Change a file's text | Section 19.1 |
| compare | Compare two files | Section 19.1 |
| format | Format a program | Section 20.1 |
| check | Check a program | Section 20.1 |
| test | Run the tests | Section 20.2 |
| build | Build a program | Section 14.1 |
| run | Run a script | Section 21.2 |
| exit | End the shell | Section 15.2 |

`test all` and `build all` are not separate commands in the table. They are the test command and the build command with the argument `all`, the way section 20.2 and section 14.1 defined them: the argument tells the routine to run everything, and the routine reads it with get, like any other argument. `exit` ends the shell, and the finish routine runs its five steps from section 15.2: flush, close in reverse order, free what the program owned, release the static memory, and report the exit in plain words.

Count the table by hand. The names hold 66 characters: list 4, copy 4, move 4, delete 6, search 6, view 4, edit 4, compare 7, format 6, check 5, test 4, build 5, run 3, exit 4. Four plus 4 is 8, plus 4 is 12, plus 6 is 18, plus 6 is 24, plus 4 is 28, plus 4 is 32, plus 7 is 39, plus 6 is 45, plus 5 is 50, plus 4 is 54, plus 5 is 59, plus 3 is 62, plus 4 is 66. Each entry adds two bytes, one for the length and one for the routine number, so 14 entries add 28 bytes. The count byte adds 1. The table is 1 plus 28 plus 66, which is 95 bytes. The count is one byte, so the table holds at most 255 commands, and the limit is stated plainly: a shell with more commands than that is a new version, the way section 9.1 versions the file format.

Say plainly what the shell does not do, because the list keeps it small, the way section 2.4 requires. It does not start a new program per command. It does not parse the command text into a new program. It does not load a file per command. It does not read startup files on login, the waste section 1.1 named: a shell that reads and parses its startup files on every login is a shell doing work the user did not ask for, before the user has typed anything, and this shell does none of it. It does nothing in the background, runs nothing on a schedule the user cannot see, and starts nothing the user did not type. There is nothing to see past the prompt, the cost is the commands the user typed, and the shell ends when the user types `exit`.

Now work one command through, step by step, the way sections 4.4 and 7.2 worked their examples, with every piece counted. The command is `list docs`, typed at the prompt, listing the same folder section 18.1 listed: notes.txt, counts.txt, and hello.

The user types the line: `list docs`, 9 characters, plus the newline, 10 bytes. The shell prints the prompt first, 2 characters, 1 call to print. It reads the line, 10 bytes. It splits the line with split, 1 call, and the split gives two pieces: `list` and `docs`. The shell looks up `list` in the command table and finds the list routine. It calls the routine with the arguments, one argument, `docs`. The routine calls list_folder once, gets three records, and prints four lines: one per file, and one total line. The output reads like this:

```
notes.txt  31 bytes
counts.txt  6 bytes
hello  72 bytes
3 files, 109 bytes
```

Check the counts by hand. The sizes add up the way section 18.1 counted them: 31 plus 6 is 37, plus 72 is 109. The command made 7 library calls: 1 to print for the prompt, 1 to split, 1 to list_folder, and 4 to print for the output. One plus 1 is 2, plus 1 is 3, plus 4 is 7. The list read no file contents, the way section 18.1 said: it read only the folder's entries. And the loader's eight steps ran zero times for this command, because they ran once, when the shell started, before the prompt ever appeared.

Now count the saving, because a design that claims a saving must show the arithmetic. Take a session of ten commands. Today, each command starts a program, so the session pays ten startups. In Riza, the shell starts once, and the ten commands are ten calls inside it, so the session pays one startup. Ten startups against one: 9 startups skipped. The saving is counted in startups, not in time, because no time was measured, and Chapter 31 measures it on the real PC. The saving grows with the session: a session of a hundred commands skips 99 startups. The standard tasks from section 1.5 are the measure that will tell: listing 10,000 files, formatting a 1,000 line file, and searching a 10 MB text file, each run through the shell, each counted in Chapter 31.

Now the command the shell does not know, because a shell that cannot say so plainly is a shell the user cannot trust. The user types `dance docs`. The shell prints the prompt, 1 call, reads the line, and splits it, 1 call, giving `dance` and `docs`. It looks up `dance` in the command table and finds nothing. The lookup fails, and the failure comes back as an error, code 7, not found, the way section 16.3 defined it: the same code find returns when the piece is not in the text, and the same code random_choice returns when the list is empty. The shell reports it in plain words, 1 call to print:

```
No command named dance.
```

The command made 3 library calls: 1 to print the prompt, 1 to split, 1 to print the error. The shell does not stop, does not crash, and does not ask again. It prints the prompt and waits for the next command. An error is a value, the way section 7.3 designed it, and a failed command is an error value, reported in plain words, not a crash.

The missing argument is the same shape, handled by the routine, not the shell. The user types `list` with no folder name. The shell splits the line, finds the list routine, and calls it with an empty argument list. The routine asks for its first argument with get, finds none, and reports that it needs a folder name, in plain words. The shell does not check arguments for the routines, because each routine knows what it needs and the shell does not. One rule, in one place, per routine.

The shell logs what it ran, the way section 17.1 designed the debug log layer. The shell is a program, so it keeps its log the way any program does: a file in the `logs` folder beside the shell's file, named `logs/shell.log`, one line per entry, in plain text. Each command is one line: the command name, the arguments, and the result, or the error code. The log holds what the shell did, never a secret, the way section 16.3 said: log the act, never the secret. A command that takes a password as an argument logs that the command ran, not the password. The log is kept small the same way: a buffer of 1,024 bytes, a limit of 64 KB, the oldest lines dropped whole when the limit is reached, and logging off costs zero, because the user can turn it off. One line in the log proves the design: the started line is written once per session, when the shell starts, however many commands the session holds. Two commands in one session produce one started line, and that is the test that proves the loader's eight steps ran once.

Ownership is the shell's between commands, the way section 6.1 requires. The shell owns the values it holds between commands, and between commands it holds almost nothing: the command table, in static memory, and the log buffer. Each command's values are freed when the command ends, because their owner, the command, is done, and a command that ends with an error frees its values the same way. Nothing from one command leaks into the next.

Adding a command is adding a routine and registering its name. The programmer writes the routine, in one source file, the way any tool's routine is written. The build tool joins it into the shell's program, and the programmer adds one entry to the command table: the name, in the length byte plus characters shape, and the routine number. The registration is the same shape as the name table from section 4.3, which registers names once and refers to them by number. There is no second mechanism, no separate registry, no file to edit by hand. A command the table does not name does not exist, and a command it names runs when typed.

## 21.2 Scripts in the same language

A script is a file of commands, written in Riza, run by the shell. That is the whole definition, and the design follows from it. Because the shell's commands are library calls, a script is a small program. It is written in the same text form Chapter 8 designed, checked by the same stage-one checks section 11.1 designed, and compiled to the same middle form Chapter 10 designed. There is no second language for scripts, no second parser, no second form. Learning the language once covers programs and scripts, the way section 8.1 promised: the same text a programmer writes is the text the shell runs as scripts.

The shell runs a script with the `run` command, naming the file: `run count_files` runs the script in the file `count_files`. The run routine works in five steps, in order.

First, read the file. The run routine reads the script's text with read_lines from section 16.1, which opens the file, reads its lines into a list, and closes it, in one call. If the answer is an error, code 1, the file does not exist, or code 2, it cannot be read, the routine reports it and stops, and nothing runs.

Second, parse the text. The run routine parses with the shared parser from section 2.2, the one parser, maintained once, used by every tool: the editor uses it while the programmer writes, the compiler uses it at build time, the formatter uses it to hold the canonical form, and the shell uses it here. Because it is the same parser, the four can never disagree about what a script means. If the text does not parse, the routine reports the error, with the line number and the reason in plain words, and runs nothing.

Third, check the script. The run routine runs the compiler's stage-one checks from section 11.1 on the tree: the types, from Chapter 5; the ownership, from Chapter 6; the errors, from Chapter 7; the calls, from section 7.1; and the names, defined before use. They are the same checks the compiler runs, so the two can never disagree: a script the shell runs, the compiler would build, and a script the compiler refuses, the shell refuses. If any check fails, the routine reports the failure, naming the line and saying what is wrong, and runs nothing. A script with a mistake is a script that never starts.

Fourth, compile the script to the middle form. The run routine runs the compiler's stage two, producing the middle form from Chapter 10, the same six operation groups plus END, the same one way to call the library. The middle form is compiled in memory: no file is written, because a script does not need one. The compiler writes a file because a program is built to be kept and run again. A script is run once, now, and the file would be a thing to store, to name, and to throw away. The middle form lives in memory for the run and is freed when the run ends.

Fifth, run it. The shell runs the middle form the way the loader runs any program, the way section 10.3 designed the translation: the pile lives on the machine's stack, named values live in fixed slots, and the translation runs at load time, in the loader. Section 10.1 said the shell executes the middle form as scripts, and this is where that happens: the script's middle form is translated and run, in the shell, the way a program's middle form is translated and run, in the loader. The same form, the same translation, the same run.

Say what a script can do that a typed command cannot, because the difference is the reason scripts exist. A typed command is one call, with arguments. A script is a program, and a program can loop, branch, and check errors, because it is written in the language, with the language's branches and loops from section 7.2 and the language's error checks from section 7.3. The shell's loop reads one line at a time; a script's loop reads the script's own lines, as many times as the script says.

Now work one script through, step by step, with every byte counted. The script is `count_files`, a small script that lists a folder, counts the files, and prints the total. It defines one function, calls it once, and prints the answer. The function passes an error up, the way section 7.3 described: if listing the folder fails, the function returns the error, and the caller prints it.

| Line | Text | Bytes |
| --- | --- | --- |
| 1 | `define count_files(folder):` | 27 characters + 1 newline = 28 |
| 2 | `    entries = list_folder(folder)` | 33 characters + 1 newline = 34 |
| 3 | `    if entries is error:` | 24 characters + 1 newline = 25 |
| 4 | `        return entries` | 22 characters + 1 newline = 23 |
| 5 | `    count = length(entries)` | 27 characters + 1 newline = 28 |
| 6 | `    return count` | 16 characters + 1 newline = 17 |
| 7 | `print(count_files("docs"))` | 26 characters + 1 newline = 27 |
| Total | | 182 bytes |

The script is 182 bytes: 28 plus 34 is 62, plus 25 is 87, plus 23 is 110, plus 28 is 138, plus 17 is 155, plus 27 is 182. The indentation is four spaces per level, the way section 8.1 designed it, and the error check is a branch, the way section 8.1 showed it: `if entries is error:`. The script checks the result before it uses it, because section 11.1 refuses a program that uses a result without checking for the error, and the checker would refuse this script if line 5 came before line 3.

The user types `run count_files` at the prompt. The shell prints the prompt, 1 call to print, reads the line, and splits it, 1 call, giving `run` and `count_files`. It calls the run routine, which reads the file with read_lines, 1 call, 182 bytes, 7 lines. The parser reads the text, the checks pass, and the middle form is compiled in memory. The run starts. The function calls list_folder once, with `docs`, and gets three records, the same three section 18.1 listed. The error check finds no error. The function calls length once and gets 3. It returns 3. The last line prints it, 1 call to print. The output is one line:

```
3
```

Check the counts by hand. The script made 6 library calls: 1 to print the prompt, 1 to split, 1 to read_lines, 1 to list_folder, 1 to length, and 1 to print the answer. One plus 1 is 2, plus 1 is 3, plus 1 is 4, plus 1 is 5, plus 1 is 6. The function call is not a library call: it is a call inside the program, the way section 7.1 described. The script read 182 bytes and printed 1 character and a newline.

Now the error run, because a script that only works when the folder exists is a script the user cannot trust. Suppose the folder `docs` does not exist. The user types `run count_files` again. The shell prints the prompt, splits the line, and the run routine reads the file, parses it, checks it, and compiles it, the same as before. The function calls list_folder, and the answer is an error, code 1, file not found. The error check finds it, and the function returns the error. The last line prints it, 1 call to print. The output is the error, in plain words:

```
Error 1: file not found: docs
```

The error run made 5 library calls: 1 to print the prompt, 1 to split, 1 to read_lines, 1 to list_folder, and 1 to print the error. The script did not count anything, because there was nothing to count, and it did not crash, because the error is a value the script checked. The failure is reported where it happened, in the open, the way section 7.3 requires.

Now the loop, because a script that cannot loop is a list of commands with a file around it. A second small script counts to three, and it is four lines.

| Line | Text | Bytes |
| --- | --- | --- |
| 1 | `count = 1` | 9 characters + 1 newline = 10 |
| 2 | `while count < 4:` | 16 characters + 1 newline = 17 |
| 3 | `    print(count)` | 16 characters + 1 newline = 17 |
| 4 | `    count = count + 1` | 21 characters + 1 newline = 22 |
| Total | | 66 bytes |

The script is 66 bytes: 10 plus 17 is 27, plus 17 is 44, plus 22 is 66. It prints 1, then 2, then 3, three lines, three calls to print, and then it ends, because the count reaches 4 and the loop stops. A typed command cannot do this: the user would have to type `print` three times, with three different arguments, and the shell would run three commands. The script types it once and runs it three times, because the loop is in the language, not in the user's hands.

Say plainly what the shell does not do with scripts, because the list is the design. It does not have a second language for scripts. It does not parse scripts differently from programs: the parser is the one parser, and the script is text in the form Chapter 8 designed. It does not run a script without checking it first: the stage-one checks run on every script, every time, before the first line runs. A script is a program, and it is treated as one, from the first read to the last line.

The test suite grows with this chapter. Eight new tests cover the shell. Five cover commands: `list docs` shows the three files and the 109 bytes; an unknown command reports code 7 and the shell goes on to the next command; `list` with no argument reports that it needs a folder name; two commands in one session produce one started line in the log, proving the loader's eight steps ran once; and a newly registered command runs when typed. Three cover scripts: the count script prints 3; a script with a mistake is refused and runs nothing; and a script with a loop prints 1, 2, 3. That is 8 new tests, and the suite grows from 104 to 112: 104 plus 8 is 112. Each test runs on the target PC, the way section 2.5 demands.

This chapter adds no new routines to the core library. The shell is built on the 34 routines already named: the text routines and the file routines from section 16.1, the five Chapter 18 named, and read_dense_line from Chapter 19. The parser is shared code, the way section 2.2 designed it, not a numbered routine. The keyboard read is a driver's work, which Chapter 27 designs, and it names no routine here. The library stays at 34 routines, and the suite grows to 112 tests.

The chapter counts as done when the shell runs on the target PC from section 3.1: the shell starts once and runs ten commands as calls, `list docs` shows the three files and the 109 bytes, the count script prints 3, the unknown command reports code 7, and the times are measured in Chapter 31. Until that run happens, the chapter's claims are marked plainly as waiting for the real-PC run, the way part 5 of the done test in section 3.3 requires.

Part 5 is complete. Chapters 18, 19, and 20 built the tools, and this chapter built the shell that runs them: commands as library calls, scripts as small programs, one language from the typed line to the compiled file. Chapter 22 designs the kernel, and the shell is the first program the kernel starts, the way Chapter 23 designs it. Chapter 23 designs boot, from power on to the prompt, where the prompt is the shell. Chapter 30 builds the full system image, and the shell is part of it. Chapter 31 measures the shell on the real PC, and Chapter 32 benchmarks it against today's stack.

---

The shell is the program the user types commands into, and in Riza a command is a library call, not a separate program. The shell starts once, pays the loader's eight steps once, and holds the tools inside it as routines it calls, the way a program calls the core library. A command is read as a line, split into a name and arguments, looked up in a 95 byte table of 14 commands, and called, with the result or the error reported in plain words. One command worked through by hand shows the proof: `list docs` is 10 bytes typed, 7 library calls, and the same 109 bytes Chapter 18 counted, with zero startups paid for the command itself, and a session of ten commands skips 9 startups. An unknown command is an error, code 7, not found, reported in plain words, and the shell goes on. A script is a file of commands in the same language, read, parsed with the shared parser, checked with the stage-one checks, compiled to the middle form in memory, and run the way the loader runs any program. One script worked through by hand shows the proof: 182 bytes, 6 calls, and the answer 3, with the error run reporting code 1 in plain words and a loop printing 1, 2, 3. No new routines join the library, and the suite grows from 104 tests to 112. Part 5 is complete: the tools, and the shell that runs them.
