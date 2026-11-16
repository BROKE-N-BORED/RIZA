# Chapter 19: Text tools

Chapter 18 built the file tools, which handle files whole and search inside them: list, copy, move, delete, and search. This chapter builds the text tools, which show, change, and compare the text inside a file: view, edit, and compare, and the dense form that stores text the way Chapter 4 stored names and code. Part 5 continues here, with the tools the user runs every day, written in Riza, on the runtime Parts 3 and 4 designed.

The file tools treat a file as a whole: a run of bytes with a name, to be listed, copied, moved, deleted, or searched. The text tools treat a file as words and lines, to be read by a human. The file tools answer what files there are. The text tools answer what the files say.

There are two sections. The first designs the three tools that work on text: view, edit, compare. The second designs the dense form for text: words stored once in a table and called by number, the way section 4.3 interns names.

One warning, stated up front the way Chapters 4 through 18 stated their own. This chapter designs the text tools and the dense form. It does not build any of them. No tool exists yet, and no claim in this chapter says one does. The chapter also claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own examples, counted by hand. No time is claimed anywhere in this chapter, because no tool has been measured on the target machine, and Chapter 3 forbids numbers without a method. The measured proof comes in Chapter 31, on the real PC.

## 19.1 View, edit, compare

Start with the job, in one sentence. The text tools do one job each, on the text inside a file: view shows it, edit changes it, compare shows how two files differ.

Each tool is a small program, and each one does one job, the way section 2.4 requires. The view tool views. It does not change anything. The edit tool edits. It does not compare. The compare tool compares. It does not change either file. A tool that does one job is a tool the user can trust, because the user knows what it will do and what it will not do.

The tools start and end the way every program does. The loader runs its eight steps from section 15.1, the tool does its job, and the finish routine runs its five steps from section 15.2, closing what is open and reporting the exit in plain words. The tools are not special cases. They are programs, and they follow the same road every program follows.

The tools are built on the core library's text routines from section 16.1, print, join, split, compare, find, length, text_of, and on the file routines from sections 16.1 and 18.1: open_file, read, read_lines, write, write_line, flush, close, and the five Chapter 18 named, list_folder, create_file, move_file, delete_file, read_line. The text tools add one routine of their own, in the second section, for reading dense text. The tools name files by name, in one folder, the way section 18.1 said. Names with folders inside them are the filesystem's work, and Chapter 26 designs them.

### View

The view tool shows a file's text, one page at a time, the way the editor from section 8.3 shows it: the text the programmer wrote, with the line numbers beside it, so the user can name a line to the edit tool.

First, open the file with open_file. If the answer is an error, code 1, the file does not exist: the tool reports the file's name and stops. If the answer is an error, code 2, the file cannot be read: the tool reports that and stops.

Second, read the file line by line with read_line, the way the search in section 18.2 did, and count the lines as they come: 1, 2, 3. The view reads one line at a time, so a file of any size is viewed the same way, because the view never holds more than one line.

Third, print the lines of the page the user asked for, each line with its line number in front of it. The page is 40 lines, a number chosen to fit the screen from section 3.1, 1366 pixels wide and 768 tall, with room for the prompt, and the user can raise or lower it. The user names the file and the page: page 1 is the first 40 lines, page 2 the next 40, and so on. Lines before the page are read and counted, not printed, because the file holds no index of line positions, and building one would be hidden work. If the page asked for is past the end of the file, the tool reports how many pages the file has, and shows nothing. That is a report, not an error.

Fourth, close the file, and report how many lines the page showed.

Work one view through, step by step. The file is notes.txt, the 31 byte file from Chapter 16, three lines: `hello world`, `one two three`, `four`. The user asks for page 1.

The tool opens notes.txt. It reads the first line and prints it with its number. It reads the second line and prints it. It reads the third line and prints it. The fourth call to read_line gives code 7, and the file is done. The tool closes the file and reports.

```
1: hello world
2: one two three
3: four
3 lines
```

Check the counts by hand. The view read 31 bytes, the whole file. It made 10 library calls: 1 to open_file, 4 to read_line, 4 to print, 1 to close. The file fits on one page, so no line was read without being printed. The view changed nothing: it read the file and put it back the way it found it, which is the point of a tool that only shows.

### Edit

The edit tool changes a file's text: replace a line, add a line, remove a line. The user names the file, the line, and the change. The change is written to a new file before it touches the old one, so a failed edit leaves the original file untouched.

First, open the source with open_file. If the answer is an error, code 1, the source does not exist: the tool reports the file's name and stops. If the answer is an error, code 2, the source cannot be read: the tool reports that and stops.

Second, create a new file under a temporary name with create_file, the way the build tool in section 14.1 writes under a temporary name. If the answer is an error, code 3, the disk is full: the tool reports it and stops.

Third, read the source line by line with read_line, and write each line to the new file with write_line, changing the named line as it goes: the new text in place of the old for a replace, the new line written in its place for an add, the line skipped for a remove. If a write gives an error, code 3, the disk is full: the tool closes both files, deletes the new file with delete_file, and reports the failure. The original file is untouched, because it was never written to, only read.

Fourth, when read_line gives code 7, the source is done. Close both files.

Fifth, move the new file into the old file's place with move_file. This is the one step that cannot be undone, and it happens only after the write has succeeded and both files are closed.

Sixth, report what was changed.

One question about move_file has to be settled here, because Chapter 18 left it open. Section 18.1 said move_file takes an old name and a new name, and fails with code 1 if the source does not exist. It did not say what happens when the destination already exists. This chapter fixes it: move_file replaces the destination. The move tool still refuses to ask for a replacement, because the move tool checks the destination first, the way section 18.1 designed it. The edit tool asks for a replacement, because the replacement is the edit, and the replacement happens only after the new file is complete.

Work one edit through, step by step. The file is notes.txt, 31 bytes, three lines. The user replaces line 2, `one two three`, with `two three`.

The tool opens notes.txt and creates the new file. It reads line 1, `hello world`, and writes it: 12 bytes. It reads line 2, `one two three`, and writes `two three` in its place: 10 bytes. It reads line 3, `four`, and writes it: 5 bytes. The fourth call to read_line gives code 7, and the source is done. The tool closes both files, moves the new file into place, and reports.

```
Replaced line 2 of notes.txt.
```

Check the counts by hand. The edit read 31 bytes and wrote 27: 12 plus 10 is 22, plus 5 is 27. The file lost 4 bytes, because `two three` is 4 characters shorter than `one two three`. The edit made 13 library calls: 1 to open_file, 1 to create_file, 4 to read_line, 3 to write_line, 2 to close, 1 to move_file, 1 to print. The original file is gone, replaced by the edited version, and the replacement happened in one move, after the write had succeeded.

Now the error run. The user makes the same edit, but the disk fills during the second write.

The tool opens notes.txt and creates the new file. It reads line 1 and writes it: 12 bytes, written. It reads line 2, and the write of `two three` gives an error, code 3, disk full. The tool closes both files, deletes the new file, and reports the failure in plain words.

```
The disk is full. notes.txt is unchanged.
```

Check the counts by hand. The error run made 10 library calls: 1 to open_file, 1 to create_file, 2 to read_line, 2 to write_line, 2 to close, 1 to delete_file, 1 to print. It read 26 bytes, the first two lines, and wrote 12, the first line only. The new file held 12 bytes when it was deleted, and the original file holds the same 31 bytes it held before the edit began. A failed edit changes nothing. That is the design, and the error run is the proof of it.

One honest note about the edit, stated in the open. The edit tool writes plain text. A dense file that is edited becomes a plain text file, because the edit writes lines, not dense codes. The dense writer in the next section makes it dense again, if the user wants it dense. The edit does not pretend to keep a file dense while rewriting it line by line.

### Compare

The compare tool shows how two files differ, line by line. The user names two files. The tool reads both, one line at a time, and reports the lines that differ, with their line numbers.

First, open both files with open_file. If either answer is an error, code 1 or code 2, the tool reports the file and stops.

Second, read one line from each file with read_line, and count the lines as they come. Compare the two lines with compare from section 16.1, which gives 0 when the two pieces of text are the same. If the answer is 0, the lines match: move on to the next pair. If the answer is not 0, the lines differ: print the line number and both lines.

Third, when one file runs out of lines and the other still has lines, the remaining lines are differences. A line in one file with no line across from it is a line the other file does not have, and the tool reports it as such.

Fourth, when both files give code 7, both are done. Close both files.

Fifth, report the total: how many lines were compared, and how many differed. If no line differed, the tool reports 0 differences, which is a result, not an error. Two files that hold the same text are the same text, and the tool says so.

The compare reads both files as text, dense or plain. A dense file is read with read_dense_line from the next section, and a plain file with read_line, and the comparison is on the text that comes back, not on the form it was stored in. A dense file and a plain file holding the same text compare as the same.

Work one compare through, step by step. The first file is notes.txt, 31 bytes, three lines: `hello world`, `one two three`, `four`. The second file is notes2.txt, 25 bytes, three lines: `hello world`, `one two`, `four`. The two files differ in one line.

The tool opens both files. It reads line 1 from each: `hello world` and `hello world`. Compare gives 0. The lines match. It reads line 2 from each: `one two three` and `one two`. Compare gives a number that is not 0. The lines differ, and the tool prints both. It reads line 3 from each: `four` and `four`. Compare gives 0. The lines match. The next call to read_line on each file gives code 7, and both files are done. The tool closes both files and reports.

```
line 2 in notes.txt: one two three
line 2 in notes2.txt: one two
1 difference in 3 lines
```

Check the counts by hand. The bytes read add up: 31 plus 25 is 56. The read_line calls add up: 4 for the first file, 4 for the second, 8 in all, one extra per file to learn the file is done. The compare calls add up: one per pair of lines, 3 in all. The full count of library calls is 18: 2 to open_file, 8 to read_line, 3 to compare, 3 to print, 2 to close. Two plus 8 is 10, plus 3 is 13, plus 3 is 16, plus 2 is 18. One line differed, and the report names it, with both versions, so the user can see what changed without opening either file.

### What the tools do when something goes wrong

Every failure is an error value with a code from section 16.3 and a message in plain words, the way section 7.3 designed errors. The failures the text tools meet are these.

| What went wrong | Code | What the tool does |
| --- | --- | --- |
| A file that does not exist | 1 | Reports the file's name and stops |
| A file that cannot be read | 2 | Reports the file's name and stops |
| A disk that is full | 3 | Reports the disk is full and stops; the edit deletes its new file first |
| A line that runs out of memory | 4 | Reports the line is too big and stops |

The line that runs out of memory is the one honest limit in the table. The tools hold one line at a time, and a single line bigger than the free memory fails with code 4, out of memory, the way section 6.4 describes. A file of any size is fine, because the file is never held whole. A line of any size is not fine, because the line is the unit the tools hold. The limit is stated here so no reader is surprised by it later.

The tools also write to the debug log layer from section 17.1, so the record is there when something goes wrong. The log holds what the tool did and when: the view of a file, the edit of a line, the compare of two files, the error that stopped a run. The log holds the names and the counts, not the contents. An edit logs the file and the line number. It does not log the text, because the text might hold a secret, and section 16.3 said to log the act, never the secret.

### What the tools do not do

The view does not change the file. It reads the file and puts it back the way it found it. A view run twice shows the same text twice, and if the file changed between the two views, the second view shows the changed text, because the file is the truth and there is nothing to go stale, the way section 9.3 said.

No tool runs in the background. Nothing watches the files while the user is doing something else. Nothing keeps a tally of what changed. The tools run when the user runs them, and they stop when they are done.

No tool does work the user did not ask for. The view does not edit. The edit does not keep a backup copy: the user asked for the change, and the change is the change. The compare does not change either file. Each tool does one job, the way section 2.4 requires, and the user can see what it did, measure what it cost, and turn it off, because the tool is a program like any other, and it stops when it is done.

Ownership follows section 6.1. A tool that opens a file owns it, and closes it when done. The edit owns both files until it closes them, and the close is the tool's own act, not the finish routine's. The finish routine from section 15.2 closes anything still open, but that is the backstop, not the plan, the way section 18.1 said.

## 19.2 Dense text handling

Start with the job, in one sentence. Dense text is text stored the dense way, the way Chapter 4 stored names and code, so that a file of text takes less room and less work to read.

The idea is section 4.3 applied to words. Section 4.3 stored each name once in a table and called it by number everywhere else, because a name used fifty times costs 750 bytes spelled out and 66 bytes interned. Words in a text file repeat the same way. A note about a project uses the word `project` on every line. A list uses the same headings again and again. The dense form stores each repeated word once in a small table, and stores the file as short codes for those words. The words are the table. The file is the codes.

The dense form is the binary form, and the text is the view, the way section 2.2 said. The file on disk is dense. The text the user reads is generated from it, whenever it is needed, by the same fixed rule every time.

The dense file is laid out in four parts, in order: a magic, a version, a word table, and a body. The magic is 4 bytes, the way section 9.1 and section 14.2 start their files: R I Z D, in bytes 52 49 5A 44. The D says dense, the way the P in section 14.2 said package and the T in section 17.2 said trace. After the magic comes a version byte, the value 1.

The word table follows the shape section 4.3 gave the name table: one byte saying how many entries it holds, then each entry as a length byte followed by the characters of the word, spelled out once. The table holds the words the file uses more than once, the most used first. A word used once does not go in the table, because a word used once costs the same in the table as spelled out, and the table is for words that pay for their place. The table holds at most 128 words. A file with more repeated words than that stores the 128 most used in the table, and the rest as literals, the way this section describes below.

The body is a run of items, each one starting with a single byte that says what it is.

| First byte | What it means | What follows |
| --- | --- | --- |
| 0 to 127 | A word code, naming a table entry | Nothing; the word is in the table |
| 128 | A newline, ending the line | Nothing |
| 129 | A literal, text not in the table | A length byte, then the characters |
| 130 to 255 | Reserved for a later version | Nothing yet; a file that uses one is invalid |

A word is the characters between spaces, punctuation included, the way split from section 16.1 divides a line on spaces. `hello,` is one word, and `hello` is another, because the comma is part of the characters between the spaces. Between two words on the same line, the text has one space. The space is not stored; the reader puts it back. That is the one rule the round trip depends on, and it is fixed: two words on a line, one space between them, and the space is implied, never written.

Say how a tool reads a dense file without expanding the whole file into memory, because that is the point of the form. The tool reads the magic and the table first, in one short read: the table is small, and it is held for the run. Then it reads the body one item at a time, expanding each item as it goes: a word code becomes the table word, a literal becomes its characters, a newline ends the line. The tool holds the table and one line, and no more. A dense file of any size is read the same way, because the reader never holds more than one line past the table.

The reading needs one routine section 16.1 did not name, and this chapter names it, in the same shape as the five Chapter 18 named. It joins the core library with them, and the library grows from 33 routines to 34, and the file routines grow from 12 to 13.

| Routine | Takes | Hands back | If it fails |
| --- | --- | --- | --- |
| read_dense_line | An open dense file | The next line, as plain text, without its newline | An error, code 7, when there are no more lines; code 2, if the file cannot be read; code 8, if the file is not dense text |

The first call to read_dense_line reads the magic, the version, and the table, and holds the table for the run. Later calls read the body, one line at a time, expanding each item as they go. If the magic is not R I Z D, the file is not dense text, and the routine gives code 8, wrong kind, the code section 16.3 kept for the ones only the run can see. The view tool reads the first four bytes when it opens a file: if they are R I Z D, it reads with read_dense_line; if not, it reads with read_line. The user does not name the form. The file names itself.

Say how a dense file is written, because a form nothing produces is a form nobody uses. The writer is a small program built on the same routines, and it works in two passes. The first pass reads the plain text line by line with read_line, splits each line on spaces with split, and counts how often each word appears. The second pass writes the dense file: the magic, the version, the table built from the words used more than once, and the body, one item at a time, with write. The writer reads the file twice, once to count and once to write, and holds only the counts between the passes, not the whole file.

Now work one dense text through, step by step, with every byte counted. The file is project.txt, three lines about a project.

```
the project needs work
the project needs time
the project needs help
```

First, the plain text. Each line is 22 characters plus a newline: `the` is 3, a space, `project` is 7, a space, `needs` is 5, a space, and `work`, `time`, or `help` is 4. Three lines at 23 bytes each is 69 bytes.

Now the dense form. The words used more than once are `the`, `project`, and `needs`, each used 3 times. The words used once are `work`, `time`, and `help`, and they stay out of the table.

The word table: 1 byte saying 3 entries, then `the` as 1 length byte plus 3 characters, `project` as 1 plus 7, and `needs` as 1 plus 5. The table is 1 plus 4 plus 8 plus 6, which is 19 bytes.

The body: each line is three word codes, one literal, and one newline. The codes are 1 byte each: 3 bytes. The literal is 129, a length byte, and 4 characters: 6 bytes. The newline is 1 byte. Each line is 3 plus 6 plus 1, 10 bytes. Three lines is 30 bytes.

| Part | Bytes |
| --- | --- |
| Magic, R I Z D | 4 |
| Version | 1 |
| Word table | 19 |
| Body | 30 |
| Total | 54 |

The dense form is 54 bytes against 69 for the plain text: a saving of 15 bytes. The arithmetic is all above, and any reader can recount it.

Now the way back, because the round trip is the promise. The reader reads the table: entry 0 is `the`, entry 1 is `project`, entry 2 is `needs`. Then it reads the body, one item at a time. The first line is code 0, code 1, code 2, the literal `work`, and a newline. The reader writes `the`, a space, `project`, a space, `needs`, a space, `work`, and a newline: 23 bytes, the same as the plain line. The second and third lines rebuild the same way. The text that comes out is 69 bytes, and every character is the same as the text that went in. The round trip is exact, byte for byte, the way section 8.2 designed it for programs.

Read the result honestly, because a single small example proves less than it might look like. The saving is real, but the file is small, and on a small file the fixed costs loom large. The magic, the version, and the table cost 24 bytes, nearly half the dense total, and they would cost the same for a file ten times longer, because the same three table entries would serve hundreds of uses. The dense form wins by more as the file grows, because the table is paid once per file and the uses are paid per use.

Now the honest case, because rule 7 of section 1.5 says to report the number that loses too. Take a file with no repeated words at all.

```
alpha beta gamma
delta epsilon zeta
```

The plain text is 36 bytes: the first line is 16 characters plus a newline, 17, and the second is 18 plus a newline, 19. No word appears twice, so the table holds nothing: 1 byte, the count saying 0. Every word is a literal. The first line is three literals and a newline: `alpha` costs 7 bytes, `beta` costs 6, `gamma` costs 7, and the newline costs 1, which is 21 bytes. The second line is `delta` at 7, `epsilon` at 9, `zeta` at 6, and the newline at 1, which is 23 bytes. The body is 44 bytes. The dense form is 4 for the magic, 1 for the version, 1 for the empty table, and 44 for the body: 50 bytes, against 36 for the plain text. The dense form is 14 bytes bigger.

Say plainly why, because the loss has a reason and the reason is the design. A literal costs 2 bytes more than the word itself, the 129 and the length, and a word used once pays that cost with no saving to set against it. The dense form wins where words repeat, and loses where they do not. The round trip still holds: the 50 byte dense form rebuilds to the same 36 bytes of plain text, byte for byte. The form is correct. It is only bigger, and the book says so instead of hiding the case where the design loses.

Say honestly what the dense form does not handle, because the limits are as much a part of the design as the layout. Text with more than one space between words, or with leading spaces, the way a program's indentation puts four spaces at the start of a line, does not fit the one space rule, and those runs of spaces are stored as literals. A program listing stored as dense text would pay the literal cost on every indented line. The dense form is designed for ordinary prose, where words are separated by single spaces. Programs have their own dense form, from Chapter 4, and the text form is not a second way to store them.

The dense form also has a size limit the reader should know. The table holds at most 128 words, because the word codes run from 0 to 127. A file with more than 128 repeated words stores the 128 most used in the table, and the rest as literals. The saving shrinks as the table fills, and a file with hundreds of repeated words gains less than a file with a few. The limit is stated here so no reader is surprised by it later, the way section 9.2 stated the name table's limit of 255 names.

The test suite grows with this chapter. One new test covers the new routine: read_dense_line returns the lines of a dense file in order, and code 7 at the end. Nine new tests cover the tools: the view shows the 3 lines of notes.txt; the edit replaces line 2 and the file holds 27 bytes; the edit with a missing source reports code 1 and writes nothing; the edit with a full disk deletes the new file and the original is untouched; the compare finds 1 difference in the two files; the compare of a file with itself reports 0 differences; the dense form of the 69 byte file is 54 bytes and round-trips to 69; the dense form of the 36 byte file with no repeats is 50 bytes and round-trips to 36; and the view of a dense file shows the same lines as the view of the plain file. That is 10 new tests, and the suite grows from 85 to 95: 85 plus 10 is 95. Each test runs on the target PC, the way section 2.5 demands.

The chapter counts as done when the tools run on the target PC: the view shows the file, the edit changes the line and leaves the original untouched on a failure, the compare finds the difference, the dense file round-trips byte for byte, and the times are measured in Chapter 31. Until that run happens, the chapter's claims are marked plainly as waiting for the real-PC run, the way part 5 of the done test in section 3.3 requires.

Chapter 20 builds the developer tools, the formatter and the checker, and the formatter is the tool that holds the programmer's text to the canonical form section 8.2 designed. Chapter 21 builds the shell, where the tools are run as commands. Chapter 31 measures the tools on the real PC, and Chapter 32 benchmarks them against today's stack, including the standard task from section 1.5: formatting a 1,000 line file.

---

The text tools are small programs that do one job each, on the text inside a file. View shows a file's text one page at a time, one line at a time, with the line numbers beside it. Edit changes a line by writing a new file and moving it into place only when the write succeeds, so a failed edit leaves the original untouched. Compare reads two files line by line and reports the lines that differ, with their line numbers. Dense text stores the repeated words once in a table and the file as short codes: a 69 byte file becomes 54 bytes and comes back as the same 69, and a file with no repeated words becomes 50 bytes against 36, reported as the loss it is. One new routine joins the core library, and the suite grows from 85 tests to 95. The tools do no hidden work, keep no index, run nothing in the background, and log the act, never the secret.
