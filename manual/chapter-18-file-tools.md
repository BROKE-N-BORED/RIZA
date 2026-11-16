# Chapter 18: File tools

Part 1 named the waste, the idea, and the rules. Part 2 built the language. Part 3 built the binary and the compiler. Part 4 built the runtime and the core library: Chapter 15 designed the startup and the shutdown, Chapter 16 designed the 28 routines every program calls, and Chapter 17 designed the debugging that is built into every program from the start. Part 5 begins here, with the tools and utilities, and this chapter is the first of them: the file tools.

The tools are the programs the user runs every day. The language, the compiler, and the runtime are the workshop. The tools are what the workshop makes for the user to hold. Every program is a file, every document is a file, and the tools that list, copy, move, delete, and search files are the tools the user reaches for first. They are written in Riza, on the runtime Parts 3 and 4 designed, built by the compiler from Chapter 11, run by the loader from Chapter 15, and they call the core library's routines from Chapter 16. The tools are ordinary programs, and that is the point: the system builds its own tools, in its own language, on its own runtime.

There are two sections. The first designs the four tools that handle files whole: list, copy, move, delete. The second designs the tool that looks inside files: search.

One warning, stated up front the way Chapters 4 through 17 stated their own. This chapter designs the file tools. It does not build any of them. No tool exists yet, and no claim in this chapter says one does. The chapter also claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own examples, counted by hand. No time is claimed anywhere in this chapter, because no tool has been measured on the target machine, and Chapter 3 forbids numbers without a method. The measured proof comes in Chapter 31, on the real PC.

## 18.1 List, copy, move, delete

Start with the job, in one sentence. The file tools do one job each, on whole files: list shows what is in a folder, copy makes a second file with the same contents, move gives a file a new name, and delete removes a file.

Each tool is a small program, and each one does one job, the way section 2.4 requires. The list tool lists. It does not copy. The copy tool copies. It does not move. A tool that does one job is a tool the user can trust, because the user knows what it will do and what it will not do. A tool that does five jobs is five tools sharing a name, and the user cannot tell which one is running.

The tools start and end the way every program does. The loader runs its eight steps from section 15.1, the tool does its job, and the finish routine runs its five steps from section 15.2, closing what is open and reporting the exit in plain words. The tools are not special cases. They are programs, and they follow the same road every program follows.

The tools are built on the core library's file routines from section 16.1: open_file, read, read_lines, write, write_line, flush, close. But four routines the tools need are not in section 16.1, and this chapter names them, in the same shape, because the tools cannot be built without them. They join the core library, and the library grows from 28 routines to 33, and the file routines grow from 7 to 12.

| Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- |
| list_folder | A folder name | A list of records, one per entry, each holding the entry's name, whether it is a file or a folder, and its size in bytes | An error, code 1, if the folder does not exist; code 2, if it cannot be read |
| create_file | A name | The new file, empty | An error, code 3, if the disk is full |
| move_file | An old name and a new name | Nothing; the file has its new name | An error, code 1, if the source file does not exist |
| delete_file | A name | Nothing; the file is gone | An error, code 1, if the file does not exist |

The record list_folder hands back is a record in the shape section 5.2 defined: fields at fixed positions. The name is the entry's name, as text. The kind is one byte, saying file or folder. The size is the entry's size in bytes, for a file, and 0 for a folder, because a folder's size is the filesystem's business, and Chapter 26 designs it. The list comes in the order the filesystem holds the entries, and the list tool prints them in that order, without sorting, because sorting is work the user did not ask for.

The tools name files by name. Names with folders inside them, a path through more than one folder, are the filesystem's work, and Chapter 26 designs them. This chapter does not design path syntax, and the examples name files in one folder.

### List

The list tool shows what is in a folder, names and sizes. It works in three steps.

First, call list_folder with the folder's name. If the answer is an error, code 1, the folder does not exist, and the tool reports the folder's name and stops. If the answer is an error, code 2, the folder cannot be read, and the tool reports that and stops. If the answer is a list, the tool has the entries.

Second, for each entry, print one line: the name and the size, for a file, or the name and the word folder, for a folder.

Third, print a total line: how many files it showed, and how many bytes they hold together.

Work one list through, step by step. The folder is called docs, and it holds three files the reader already knows: notes.txt, the 31 byte file from Chapter 16, counts.txt, the 6 byte file the word counter wrote, and hello, the 72 byte hello loop program file from Chapter 9.

The tool calls list_folder once, and gets three records. It prints three lines, one per file.

```
notes.txt 31
counts.txt 6
hello 72
```

Then it prints the total line.

```
3 files, 109 bytes
```

Check the counts by hand. The sizes add up: 31 plus 6 is 37, plus 72 is 109. The tool made 5 library calls: 1 to list_folder, 4 to print. The list read no file contents. It read only the folder's entries, which is the filesystem's work, and Chapter 26 designs how the filesystem stores them. The list shows what is there, and nothing else.

### Copy

The copy tool makes a second file with the same contents, byte for byte. It works in six steps, and the checks run before the work, so a copy that will fail fails before it reads anything, the way section 9.4 orders its checks cheapest first.

First, check the destination. Call list_folder on the folder, and compare each entry's name with the destination name, using compare from section 16.1, which gives 0 when the two pieces of text are the same. If any entry matches, the destination already exists: the tool reports it and stops, without writing anything. The copy never overwrites a file the user did not ask it to replace.

Second, open the source with open_file. If the answer is an error, code 1, the source does not exist: the tool reports the file's name and stops.

Third, read the source with read, which hands back the file's text. If the answer is an error, code 2, the source cannot be read: the tool reports it and stops.

Fourth, create the destination with create_file, which hands back the new empty file. If the answer is an error, code 3, the disk is full: the tool reports it and stops.

Fifth, write the text to the destination with write, which hands back the number of bytes written. If the answer is an error, code 3, the disk is full: the tool reports it and stops.

Sixth, close both files, and report what was copied and how many bytes.

Work one copy through, step by step. The source is notes.txt, 31 bytes, in the docs folder. The destination is backup.txt, which does not exist yet.

The tool calls list_folder once, and gets the three entries: notes.txt, counts.txt, hello. It compares backup.txt with each name: not the same, not the same, not the same. Three comparisons, no match. The destination does not exist, and the copy proceeds.

The tool opens notes.txt, and reads it: 31 bytes. It creates backup.txt, and writes the 31 bytes. It closes both files, and reports.

```
Copied notes.txt to backup.txt, 31 bytes.
```

Check the counts by hand. The copy read 31 bytes and wrote 31 bytes. It made 11 library calls: 1 to list_folder, 3 to compare, 1 to open_file, 1 to read, 1 to create_file, 1 to write, 2 to close, 1 to print. The two files hold the same 31 bytes, and the copy is exact, because the copy moves bytes, not meanings: it does not read the file as text, does not change its line endings, does not convert anything. A copy of a program file is a program file, and a copy of a text file is a text file.

Now the error run. The user asks for a copy of missing.txt, a file that was never there, to backup.txt, in the same docs folder.

The tool calls list_folder, and compares backup.txt with the three names, no match. The destination check passes. The tool calls open_file on missing.txt, and the answer is an error, code 1, file not found. The tool reports it in plain words and stops.

```
The file missing.txt does not exist.
```

Check the counts by hand. The error run made 6 library calls: 1 to list_folder, 3 to compare, 1 to open_file, 1 to print. It read 0 bytes of file contents and wrote 0 bytes. No file was created, and nothing was changed. The error is a value, the way section 7.3 designed errors, with a code and a message, and the tool reports it and gives up cleanly.

One honest note about the copy, stated in the open. The copy holds the whole file in memory at once, because read hands back the whole file. A file bigger than the free memory fails with code 4, out of memory, the way section 6.4 describes, and the copy reports it and stops. The limit is stated here so no reader is surprised by it later. The search tool in the next section avoids the limit by reading one line at a time.

### Move

The move tool gives a file a new name, or a new folder. It works in three steps.

First, check the destination, the same way the copy does: call list_folder, compare the names, and if the destination already exists, report it and stop, without moving anything.

Second, call move_file with the old name and the new name. If the answer is an error, code 1, the source does not exist: the tool reports it and stops.

Third, report what was moved.

The move reads 0 bytes and writes 0 bytes of file contents. The rename moves no bytes at all. The file's contents stay where they are on the disk, and the filesystem updates its own record of the name, which is Chapter 26's work. The move of notes.txt to archive.txt, in the docs folder, makes 6 library calls: 1 to list_folder, 3 to compare, 1 to move_file, 1 to print. The work is small, and the chapter says so, because the move is a name change, not a copy.

### Delete

The delete tool removes a file. It works in two steps.

First, call delete_file with the name. If the answer is an error, code 1, the file does not exist: the tool reports it and stops.

Second, report what was deleted.

The delete of backup.txt makes 2 library calls: 1 to delete_file, 1 to print. It reads 0 bytes and writes 0 bytes.

Two things are stated plainly about the delete, because they are the kind of thing a reader must not learn by surprise. The delete tool does not ask twice. The user named the file, and the naming is the asking. And there is no undo. A deleted file is gone. The tool reports what it deleted, so the record is in the log, but the file does not come back.

### What the tools do when something goes wrong

Every failure is an error value with a code from section 16.3 and a message in plain words, the way section 7.3 designed errors. The four failures the file tools meet are these.

| What went wrong | Code | What the tool does |
| --- | --- | --- |
| A file that does not exist | 1 | Reports the file's name and stops |
| A folder that does not exist | 1 | Reports the folder's name and stops |
| A file that cannot be read | 2 | Reports the file's name and stops |
| A disk that is full | 3 | Reports the disk is full and stops |

The tools also write to the debug log layer from section 17.1, so the record is there when something goes wrong. The log holds what the tool did and when: the copy of notes.txt to backup.txt, the delete of a file, the error that stopped a run. The log holds the names, not the contents. A copy logs the two names and the byte count. It does not log the file's text, because the text might hold a secret, and section 16.3 said to log the act, never the secret.

### What the tools do not do

The list tool does not build a hidden index. It reads the folder when asked, and keeps no record of it after. A list run twice reads the folder twice, and the second list is current, because the folder is the truth and there is nothing to go stale, the way section 9.3 said the file is the truth.

No tool runs in the background. Nothing scans the files while the user is doing something else. Nothing watches the folders, and nothing keeps a tally of what changed. The tools run when the user runs them, and they stop when they are done.

No tool does work the user did not ask for. The list does not read file contents, only names and sizes. The copy does not compress or convert. The move does not copy bytes. The delete does not ask twice, and it does not keep a copy. Each tool does one job, the way section 2.4 requires, and the user can see what it did, measure what it cost, and turn it off, because the tool is a program like any other, and it stops when it is done.

Ownership follows section 6.1. A tool that opens a file owns it, and closes it when done. The copy owns both files until it closes them, and the close is the tool's own act, not the finish routine's. The finish routine from section 15.2 closes anything still open, but that is the backstop, not the plan: a tool that leaves its files open has left work for the shutdown to do, and the tools do their own work.

## 18.2 Search inside files

Start with the job, in one sentence. The search finds the files that hold a word, and shows where in each file the word is: the file's name, the line number, and the line itself.

The user names a folder and a word. The search reads each file in the folder, one file after another, and reports every line that holds the word. The search needs one routine section 16.1 did not name, and this chapter names it, in the same shape as the four in the last section. It joins the core library with them, and the file routines grow to 12.

| Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- |
| read_line | An open file | The next line, without its newline | An error, code 7, when there are no more lines; code 2, if the file cannot be read |

The line comes back without its newline, because the newline is the separator between lines, not part of the line. When the file has no more lines, read_line gives an error, code 7, not found, and the search knows the file is done. The search checks the code: code 7 means done, and any other code means a real failure.

The search works in five steps.

First, call list_folder on the folder. If the answer is an error, code 1, the folder does not exist: the tool reports it and stops, because there is nothing to search. If the answer is an error, code 2, the folder cannot be read: the tool reports it and stops.

Second, for each entry that is a file, open it with open_file. A folder inside the folder is skipped: the search covers one folder, not the folders inside it. If open_file gives an error, code 1 or code 2, the tool reports the file and moves on to the next file. One unreadable file does not stop the search of the rest.

Third, read the file line by line with read_line, and count the lines as they come: 1, 2, 3. For each line, call find from section 16.1, with the line and the word. If find gives a position, the line holds the word: print the file's name, the line number, and the line. If find gives an error, code 7, the line does not hold the word: move on to the next line.

Fourth, when read_line gives code 7, the file is done. Close it, and move on to the next file.

Fifth, when every file is done, report the total: how many files were searched, and how many matches were found.

The search reads each file once, in one long run, the way section 3.1 said the spinning drive reads fastest, and compares the word against the file's text as it reads, without loading the whole file into memory. The memory the search holds is one line at a time, plus the word. A file of any size is searched the same way, because the search never holds more than one line. That is the shape the copy does not have: the copy holds the whole file, and the search holds one line. The search is the tool that scales to a file bigger than the memory, because it never tries to hold the file.

A match is a line that holds the word, reported once per line, even if the word appears twice in the line. The report names the line, not the position in the line. The position is find's answer, and the search uses it only to decide that the line matches. If no line in any file holds the word, the search reports 0 matches, which is a result, not an error.

Work one search through, step by step. The folder is called words, and it holds three small files. The word is hello.

The first file is notes.txt, 31 bytes, three lines: `hello world`, `one two three`, `four`. The search reads the first line and calls find: hello is in it, at position 0. One match, line 1. The second and third lines do not hold the word. The fourth call to read_line gives code 7, and the file is done.

The second file is greet.txt, 22 bytes, two lines: `hello there`, `say hello`. The first line holds the word, at position 0. The second line holds the word, at position 4. Two matches, lines 1 and 2. The third call to read_line gives code 7, and the file is done.

The third file is count.txt, 8 bytes, two lines: `one`, `two`. Neither line holds the word. No matches. The third call to read_line gives code 7, and the file is done.

The search prints the three matches.

```
notes.txt line 1: hello world
greet.txt line 1: hello there
greet.txt line 2: say hello
```

Then it prints the total line.

```
Searched 3 files, found 3 matches.
```

Check the counts by hand. The bytes read add up: 31 plus 22 is 53, plus 8 is 61. The matches add up: 1 plus 2 is 3, plus 0 is 3. The read_line calls add up: 4 for the first file, 3 for the second, 3 for the third, 10 in all, one extra per file to learn the file is done. The find calls add up: one per line read, 3 plus 2 plus 2, 7 in all. The full count of library calls is 28.

| Routine | Calls |
| --- | --- |
| list_folder | 1 |
| open_file | 3 |
| read_line | 10 |
| find | 7 |
| close | 3 |
| print | 4 |
| Total | 28 |

One plus 3 is 4, plus 10 is 14, plus 7 is 21, plus 3 is 24, plus 4 is 28.

### What the search does not do

The search does not build an index in advance. It reads the files when asked, and keeps nothing after. A search run twice reads the files twice, and the second search is current, because the files are the truth and there is nothing to go stale.

The search does no background work. Nothing scans the files while the user is doing something else, and nothing keeps a record of what was in them.

The search has no memory of past searches. Each search starts fresh, reads the files, and forgets them. The user asks again, and the search reads again.

The search covers one folder, not the folders inside it. A folder inside the folder is skipped, and its files are not searched. That is the design's scope, stated plainly.

The search logs the act, not the matches. The log line says the search ran, how many files it read, and how many matches it found. It does not say what the matching lines were, because the lines might hold a secret, and section 16.3 said to log the act, never the secret.

### What the search does when something goes wrong

A folder that does not exist gives an error, code 1, and the search reports it and stops, because there is nothing to search. A file that cannot be read gives an error, code 2, and the search reports the file and moves on to the next file, because one unreadable file is not a reason to stop the search of the rest. Both are error values with a code and a message, the way section 7.3 designed errors, reported in plain words.

The test suite grows with this chapter. Five new tests cover the five new routines: list_folder returns the folder's entries with their names and sizes; read_line returns the lines in order and code 7 at the end; create_file makes an empty file; move_file renames a file; delete_file removes a file. Six new tests cover the tools: the list shows the three files and the total of 109 bytes; the copy copies the 31 byte file byte for byte; the copy with a missing source reports code 1 and writes nothing; the move renames a file and the old name is gone; the delete removes a file and the name is gone; the search finds 3 matches in 61 bytes. That is 11 new tests, and the suite grows from 74 to 85.

The chapter counts as done when the tools run on the target PC: the list shows the folder, the copy copies the file byte for byte, the move renames, the delete removes, the search finds the matches, and the times are measured in Chapter 31. Until that run happens, the chapter's claims are marked plainly as waiting for the real-PC run, the way part 5 of the done test in section 3.3 requires.

Chapter 19 builds the text tools, which view, edit, and compare files, and the search's line by line reading is the shape the view tool uses. Chapter 21 builds the shell, where the tools are run as commands. Chapter 26 builds the filesystem, where folders live on disk and the rename and the delete happen. Chapter 31 measures the tools on the real PC, and Chapter 32 benchmarks them against today's stack, including the standard tasks from section 1.5: listing 10,000 files and searching a 10 MB text file for a word.

---

The file tools are small programs that do one job each, on the core library's file routines. List shows a folder's entries, names and sizes. Copy makes a second file with the same contents, byte for byte, and refuses to overwrite. Move renames a file, and moves no bytes. Delete removes a file, with no undo. Search reads each file once, one line at a time, and reports the file's name, the line number, and the line. Five new routines join the core library, and the suite grows from 74 tests to 85. The tools do no hidden work, keep no index, run nothing in the background, and log the act, never the secret.
