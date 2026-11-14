# Chapter 8: The human view: readable text form

Chapters 4 through 7 built the dense form: short codes for common work, values of stated sizes, memory freed by ownership, functions called by number, errors as values. The machine side of the language is designed. This chapter is the human side: the readable text a programmer writes and reads, and the proof that it matches the binary exactly.

Section 2.2 made the promise. Binary is the truth and text is a view. The programmer writes words on a screen. The editor shows text, with names the programmer chose, in lines the programmer can read. Saving a program writes the binary form, and the text the programmer sees is a view generated from that binary form whenever it is needed. The programmer's experience is text. The machine's experience is binary. Neither side is asked to live in the other's form. This chapter keeps that promise, by designing the text form and proving the round trip.

There are three sections. The first names the text form a programmer writes: the syntax, in plain readable words, with one way to write each thing. The second proves the round trip: text to binary and back produces the same text, byte for byte, and shows it worked through on one short line, with every byte counted. The third names what the editor shows and what it hides, and the rule that decides between them.

One warning, stated up front the way Chapters 4 through 7 stated their own. This chapter designs the text form and the editor's role. It does not build the compiler that reads the text. That is Chapter 11. The chapter also claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own example, counted by hand. The full grammar, every last rule of the text form, is a reference document that grows with the compiler in Chapter 11. This chapter designs the shape of the syntax, not every last rule.

## 8.1 Text syntax

The reader has already seen the text form. The worked examples in sections 4.4 and 7.2 showed programs in a plain readable form close to Python: names spelled out, one line per step, indentation for blocks. That form was not a placeholder. It is the shape of the text form this chapter designs. The exact syntax of Riza, the full set of rules, is defined here, in its shape, and the shape is the one the reader has been reading all along.

The first principle is that the text is readable first. A program is read far more often than it is written: the programmer writes a line once and reads it a dozen times, while writing the next line, while fixing a mistake, while coming back to the program a month later. So the text is written for the reader, not for the writer. Names are real words the programmer chose, not short codes, and section 4.3 already showed that a long name costs almost nothing in the dense form, one byte per use after the table entry, so the text can afford names a human would want to read. A line should read like a sentence about what the program does, because the person reading it is a person, not a parser.

The second principle is that there is one way to write each thing. Many languages give the programmer three spellings for the same idea: three ways to write a loop, two ways to define a function, a short form and a long form for the same test. Each extra spelling is a choice the programmer must make, a shape the reader must recognize, and a case the tools must handle. One way to write each thing removes all three costs. There is one way to write a branch, one way to write a loop, one way to define a function, one way to check an error. The language is smaller to learn, smaller to read, and smaller to build tools for, because it has one spelling per idea and no more.

The third principle is that costs are visible in the line, the way section 5.4 demands. A line of code must look like what it does, and cost what it looks like it costs. If a line copies a large value, the line says copy. If a line changes a number from one kind to another, the line says convert. If a line grows a list, the line says grow. If a line hands a value over to a function, the line says so, and if it lends the value instead, the line says that, because section 6.1 made handing over and lending different acts, and the text must show which one the programmer meant. The syntax has a word for each costly act, and no costly act happens without its word.

The fourth principle is that indentation shows structure. A block, a group of lines that belong together, is indented four spaces, the way the worked examples showed it. The lines inside a loop are indented under the loop. The lines inside a branch are indented under the branch. The lines inside a function are indented under the function. There are no brackets to count, no closing mark to hunt for, no missing bracket that turns the rest of the program into part of the wrong block. The shape of the text on the screen is the shape of the program: what sits together is grouped together, and the grouping is visible at a glance. A reader who looks at the left edge of the text can see the structure of the whole program without reading a single word.

The fifth principle is that sizes are stated where they matter. A number's type says how many bytes it takes, from Chapter 5, and the text shows the size where the size decides what the program can do. A number that names a position in memory is written with its size, so the reader can see it is big enough for the job. The text does not state every size on every line, because that would drown the program in bookkeeping. It states the size where the size is the point, and the compiler knows the rest, because the types are checked at build time. The programmer states what matters. The compiler checks what is stated.

Walk through the shapes from Chapters 5 through 7, written in this text form, so the reader can see the whole language on one page.

A value is made by naming it. `count = 0` makes a number and names it `count`. `name = "hello"` makes a string and names it `name`. The name is spelled out, the value is written plainly, and the line says what it does: this name holds this value.

A list is values in order, written between brackets. `[1, 2, 3]` is a list of three numbers, stored in one run, the way section 5.2 laid it out. A record is named fields, written as names with values, in the fixed order section 5.2 defined, so the reader sees the layout in the words.

A function is defined with the word `define`, followed by its name, the values it takes, and the lines of its work, indented under it. A function that adds two numbers reads like this:

```
define add(first, second):
    total = first + second
    return total
```

The definition says what the function takes and what it returns, in the open, so the compiler can check every call against that promise before the program runs, the way section 7.1 described. A call is the function's name with its values: `add(3, 4)`. The text shows the name, not the number, because the number is the machine's business and the name is the human's.

A branch is written with the word `if`, a test, and the lines to run when the test is true, indented under it. The worked example in section 7.2 showed it: `if age < 18:` followed by the indented line. A loop is written with the word `while`, a test, and the lines to repeat, indented under it. The worked example in section 4.4 showed it: `while count < 3:` followed by the indented lines. One word for a decision, one word for repetition, and the indentation shows what belongs to each.

An error check is a branch, the way section 7.3 described it. A function that can fail returns either its result or an error, and the caller checks which one it got before it uses the result. The check is written as a branch on the answer:

```
answer = open_file("notes.txt")
if answer is error:
    print(answer)
```

The first line calls a function that can fail. The second line tests whether the answer is an error. The third line, indented under the test, deals with it. The check is in the code, visible, one branch, and the compiler refuses a program that uses the result without the check, before it runs. The text shows the failure path in the open, next to the success path, so a reader can see what happens when each thing fails.

That is the shape of the syntax: names spelled out, one way to write each thing, costs visible in the line, indentation for structure, sizes stated where they matter. The full grammar, the exact rules for every corner, grows with the compiler in Chapter 11, as a reference document, written down and checked, not a set of habits the reader must guess. What this chapter fixes is the shape, and the shape is what the reader has been reading since Chapter 4.

One more thing about the text form, and it is about who else uses it. The same text a programmer writes is the text the shell runs as scripts, because Chapter 21 builds scripts in the same language. A script is written in the same words, with the same rules, as any other program, so learning the language once covers programs and scripts. The text form is one form for every human use, the way the middle form from section 2.3 is one form for every machine use.

## 8.2 Text to binary and back, exact round trip

Section 2.2 made a promise about the text form, and this section proves it. The promise is that text to binary and back produces the same text, byte for byte, with nothing lost and nothing silently changed. The programmer writes text, the system stores binary, and the text the programmer sees again is the text they wrote. This section shows how that is possible, works one line all the way through, and says plainly what it means in practice.

How it is possible is the fixed rule. The text is not stored alongside the binary, kept in sync by hand, hoping the two agree. The text is generated from the binary, by one fixed rule, every time it is needed. The rule says: one operation in the code becomes one shape in the text. A load becomes a name. A number becomes its digits. A jump back becomes a loop. A jump forward becomes a branch. The names come from the name table, spelled out in full. Because the rule is fixed, the same binary always generates the same text. There is no second version of the text to drift away from the first, because there is only one text, and it is generated fresh from the binary whenever anyone asks for it.

The other direction follows the same rule. The programmer's text goes into the binary by the same fixed mapping, in reverse: the parser reads the text and builds the binary from it, turning the text into the shared middle form from section 2.3, which Chapter 10 defines. The parser lives in the editor, the way section 2.2 said, where a human is writing and needs it. And the editor keeps the programmer's text in the canonical form, the form the rule produces: the same spacing, the same four space indentation, one newline per line. The formatter, built in Chapter 20, is the tool that does this, and it does it visibly: if the programmer writes odd spacing, the editor shows the text in its canonical form, so the programmer sees what the system will store. Nothing is changed in secret.

Work one line all the way through, the way sections 4.4 and 7.2 worked their examples, with every byte counted. The line is `print("hello")`, one line of text, plus its newline.

First, the text as the programmer writes it. `print("hello")` is 14 characters: `print` is 5, the open bracket is 1, the quote is 1, `hello` is 5, the close quote is 1, the close bracket is 1. Add the newline, the invisible character that marks the end of the line, and the text is 15 bytes.

Now the dense form, in two parts: the code, and the name table from section 4.3. The code uses the illustrative operation values from section 4.2. They are illustrations, not the final codes: the final codes come from measured frequency and live in Appendix A. The positions are counted in bytes from the start of the code, starting at 0, the way section 4.4 counted them.

| Position | Bytes | Meaning |
| --- | --- | --- |
| 0 | `11 00` | LOAD_STR entry 0: put the string `hello` on the pile |
| 2 | `40` | PRINT: take the top value off and print it |
| 3 | `00` | END |

The code is 4 bytes in total: 2 bytes for the load, 1 for the print, 1 for the end. The name table holds the one piece of text the program uses, the string `hello`. It follows the shape from section 4.3: one byte saying how many entries, then each entry as a length byte followed by the characters.

| Bytes | Meaning |
| --- | --- |
| `01` | One entry in the table |
| `05 68 65 6C 6C 6F` | Entry 0: length 5, then `hello` |

The table is 1 byte for the count, plus 6 bytes for the entry: 7 bytes in total. The dense form is 4 bytes of code plus 7 bytes of table, 11 bytes against the text form's 15.

Now the way back. The generator reads the binary and applies the fixed rule. The code says: load string entry 0, print it, end. The name table says entry 0 is `hello`. The rule turns that into the line `print("hello")`, followed by one newline. Count it: 14 characters plus 1 newline is 15 bytes, and every character is the same as the text that went in. The round trip is exact, byte for byte, because the rule that generated the text is the same rule the text was written to.

| Step | Form | Bytes |
| --- | --- | --- |
| Text in | `print("hello")` plus newline | 15 |
| Dense form | 4 bytes of code, 7 bytes of table | 11 |
| Text out | `print("hello")` plus newline | 15 |

The text that comes out is the text that went in, character for character. A reader can check it by hand, because every byte is listed above. That is the proof, in the plainest form there is.

The same holds for the longer programs the reader has already checked. The loop in section 4.4, 68 bytes of text and 37 of dense form, generates back as the same 68 bytes. The branch in section 7.2, 59 bytes of text and 38 of dense form, generates back as the same 59 bytes. The round trip does not depend on the program being small. It depends on the rule being fixed, and the rule is fixed for every program, long or short.

Say plainly what this means in practice, because the promise is only worth something if it changes how the programmer works.

Version control stores the text view. Version control is the tool that keeps the history of a program, recording what changed and when, so the programmer can look back, compare versions, and undo a mistake. It stores text, because text is what a human reads: a change shows as lines added and lines removed, in words the programmer can read. A change to a binary file shows as bytes changed, which tells the reader nothing. So the history is readable, line by line, the way a history should be.

Two versions of a program are compared as text. The programmer who wants to know what changed between yesterday and today compares the two text views, side by side, line by line, and the comparison shows the changed lines in words. The binary form is never compared by a human, because a human cannot read it. A printed listing is the text view too: a program printed on paper, or shown on a screen too small for the editor, is the same text, generated by the same fixed rule, with no second form that can drift.

And if the two forms ever disagree, that is a bug in the system, not a difference of opinion. Section 2.2 said this, and this chapter is where it is proved. The binary is the truth the machine keeps. The text is the truth the human keeps. A text view that says one thing and a binary that does another is a broken generator or a broken parser, and it is fixed, not argued about. The round trip is the test: text in, binary out, text back out, and the two texts must match, byte for byte, every time. A mismatch is a failure, reported as a failure, the way rule 7 of section 1.5 demands.

One honest question has to be answered, because a careful reader will ask it. What about the notes a programmer writes for other humans, the comments in the margin of the code, the words that explain why a line is there? A comment is not an instruction for the machine. It is a word for the next human who reads the program. The answer is that the notes survive the round trip, because they are stored in the file with the program, alongside the code, and the generator puts them back in the text where the programmer wrote them. The exact place they are stored is Chapter 9's work, because Chapter 9 defines the binary file and its sections. What this chapter fixes is the rule: nothing the programmer wrote is thrown away, and the text that comes back holds the notes as well as the code.

Two honest notes close the section, the same kind the earlier chapters closed with. The first is about what the example is not. It is not the output of a compiler, because no compiler exists yet. Chapter 11 builds the first one. The dense form above was written by hand, following the scheme in sections 4.2 and 4.3, to show what the scheme produces, and the generated text was produced by applying the fixed rule by hand, to show what the rule produces. When the compiler exists, it will produce dense forms like this one automatically, and the generator will produce text from them automatically, and Chapter 11 tests every opcode to prove it. The second note is about the full binary file. The file defined in Chapter 9 adds a header of a few fixed bytes on top of the code and the table, and those bytes are not counted here, the way they were not counted in sections 4.4 and 7.2. The header belongs to the file, not to the round trip.

## 8.3 What the editor shows, what it hides

The editor is where the text view lives, and it is the tool the programmer spends the most time in. Every other tool in the system works on the binary form or the middle form, because those are the machine's forms. The editor works on the text form, because the text form is the human's form. This section names what the editor shows, what it hides, and the rule that decides between them.

The parser lives in the editor. Section 2.2 said this, and it is worth repeating here, because it is the decision that shapes the whole editor. The parser is the part of the system that reads text and understands it: it splits the text into tokens, builds the tree, and reports mistakes in a helpful way. It is a large, complicated piece of software, and today it is shipped inside every runtime and every browser, because every runtime must read text. In Riza, the parser lives in one place, the editor, where a human is writing and needs it. The loader never parses, so the loader is smaller and faster. One parser, maintained once, used by every tool, is better than a parser reimplemented inside every runtime.

The editor shows the text. That is its first job, and it sounds obvious, but it is worth stating, because the text the editor shows is the canonical text, the form the fixed rule produces. The programmer writes, and the editor holds the text to the rule: the same spacing, the same four space indentation, one newline per line. The formatter, built in Chapter 20, is the tool that does this, and the editor runs it as the programmer works, so the text on the screen is always the text the system will store.

The editor shows the names. Every name in the program is spelled out in full, the way the programmer wrote it, because the names are the programmer's words, and the editor is the human's view. The name table numbers are not shown, because the machine's numbers are not the human's business. The programmer sees `calculate_total`. The machine sees entry 12. The editor shows the human's side of it.

The editor shows the costs, next to the lines. Section 5.4 promised this: the text view can show the cost next to the line, so the programmer sees it without writing it twice. A line that copies a large value shows its cost beside it: how many bytes move, how many records are copied. A line that grows a list shows the copy it performs. A line that converts a number shows the conversion. The costs are computed from the types in Chapter 5 and the layouts in Chapter 6, so the programmer reads the line and sees the price in the same glance.

The editor shows the errors, where they happen. Section 7.3 made every error a value the program checks, and the compiler refuses a program that uses a result without checking for the error. The editor shows those refusals as the programmer types: the line that uses an unchecked result is marked, the line that hands the wrong kind of value to a function is marked, the line that leaves a value with no owner is marked. The checker, built in Chapter 20, runs the compiler's checks in the editor, so the programmer finds out about a mistake while writing, not after building. The error is shown at the line that caused it, in plain words, because an error shown somewhere else is an error the programmer has to hunt for.

The editor shows the ownership the compiler checked. Section 6.1 made ownership a rule the compiler checks before the program runs: every value has one owner, and the owner frees it when it is done. The editor shows that check while the programmer writes. A value that is handed over shows its new owner. A value that is lent shows the loan, and when the loan ends. A value with no owner is marked before the program is ever built, because a value nobody frees is a leak, and a leak is a mistake the editor can catch while the programmer is still looking at the line.

The editor hides the byte codes and the name table numbers. The programmer never sees the operation values, the `10` for a load or the `40` for a print, because those are the machine's codes, and the machine's codes are not the human's business. The programmer never sees entry 0 or entry 12, because the numbers are how the machine finds the names, and the programmer finds the names by reading them. The programmer sees the words, and the words are the program. The codes and the numbers are the storage, and the storage is the system's business.

The editor hides the jump targets and the cache lines. The programmer never sees position 23 or position 4, because the jumps are how the machine moves through the code, and the programmer moves through the code by reading the loop and the branch. The loop is the shape. The jump is the mechanism. The editor shows the shape. And the programmer never sees which 64 byte line a value sits in, because the cache is managed by the hardware, and section 5.3 already said the program cannot order it around. The layout is the programmer's business, because the layout is in the code. The cache line is the hardware's business, because the hardware owns it.

Hiding is a choice the programmer can undo, not a wall. That is the rule section 2.4 demands. Anything the editor hides can be shown on request, because hiding for good would be hidden work, and hidden work is forbidden. A programmer who wants to see the byte codes can ask, and the editor shows them, next to the words they came from. A programmer who wants to see the jump targets can ask, and the editor shows them, next to the loop they belong to. A programmer who wants to see the name table can ask, and the editor shows it, entry by entry. The hidden things are hidden by default, because the default view is the human's view, and the human's view is words. But nothing is hidden from the programmer who asks, because the programmer owns the program. The editor hides to keep the view readable. It never hides to keep the programmer out.

The editor must fit the screen it was built for. Section 3.1 set the rule: every window in Riza must fit and be usable at 1366x768, with text readable at that size, or it does not ship. What the editor shows must earn its space, because the screen is small and every line of it is the programmer's working room. The costs shown next to a line must be worth the space they take. The editor does not fill the screen with panels the programmer did not ask for, because a panel the programmer did not ask for is space taken from the program. The text is the work, and the work gets the screen.

The text tools in Chapter 19 build on the editor's view. View, edit, and compare all work on the text form, because the text form is the form a human reads. A tool that shows a program shows the text view. A tool that compares two programs compares the text views. A tool that searches a program searches the text view, because the programmer searches for words, not bytes. The editor is the first of the text tools, and Chapter 19 builds the rest on the same view, so every tool that shows a program to a human shows the same text, generated by the same fixed rule.

Part 2 ends here. Chapter 4 gave the language its short codes. Chapter 5 gave it its values. Chapter 6 gave it memory without a collector. Chapter 7 gave it functions, flow, and errors. Chapter 8 gave it the human view: the readable text a programmer writes, the exact round trip that proves the text and the binary are the same program, and the editor that shows the one and hides the other, on the programmer's terms. The language is designed, from the dictionary to the human view. Part 3 builds the binary format and the compiler that make it real, starting with Chapter 9, where the file the loader reads is defined, section by section, and the dictionary section holds the name table this chapter's text is generated from.

---

The human view is the language as a programmer meets it. The text form is readable first, with one way to write each thing, costs visible in the line, indentation for structure, and sizes stated where they matter, and it is the same form the worked examples have used since Chapter 4. The round trip is exact by construction: the text is generated from the binary by a fixed rule, the editor holds the programmer's text to the same rule, and one line worked through by hand shows 15 bytes of text becoming 11 bytes of dense form and coming back as the same 15 bytes. The editor shows the text, the names, the costs, the errors, and the ownership, and hides the byte codes, the table numbers, the jump targets, and the cache lines, and anything hidden can be shown on request, because hiding is a choice, not a wall. Part 2 is done. The language is designed. Part 3 builds it.
