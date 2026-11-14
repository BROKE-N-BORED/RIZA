# Chapter 7: Functions, flow, and errors

Chapter 6 said where values live, and when they are freed. This chapter says how the program is put together around those values: how code is organized into functions, how it decides and repeats, and how it handles failure.

A program is not one long run of operations. It is built from named blocks that can be called, from decisions that send it one way or the other, and from repetitions that do the same work many times. And it fails, sometimes, because files are missing, memory runs out, and numbers grow past their size. This chapter designs the rules for all three: calling and returning, branches and loops, and errors as values.

There are three sections. The first names the function: a named block of code the program can call, with values passed in and a result returned. The second covers branches and loops: decisions and repetition, both built from jumps. The third covers errors: failures as values the program holds and decides about, not crashes.

One warning, stated up front the way Chapters 4, 5, and 6 stated their own. This chapter designs the rules. It does not fix the final text syntax a programmer types. That is Chapter 8. It does not build the compiler that checks the rules. That is Chapter 11. The chapter also claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own examples, counted by hand, and the measured proof comes in Chapter 31, on the real PC, with the comparison against today's stack in Chapter 32.

## 7.1 Calling and returning

A function is a named block of code the program can call. The program gives the function its name once, in the name table from section 4.3, and calls it by number after that. The function takes values in, does its work, and hands a result back. The rest of the program does not need to know how the function does its work. It needs to know what the function takes, what it returns, and what can go wrong, and section 7.3 covers the last of those.

Walk through a call, step by step, because the steps are the design.

First, the caller hands values to the function. The values go on the stack, the pile from section 6.3, one on top of another, in the order the function expects them. A function that adds two numbers gets two numbers on the pile. A function that reads a file gets the file's name on the pile. The handover follows the ownership rule from section 6.1: the caller either hands a value over, so the function owns it now and the caller is done with it, or lends it, so the function borrows it and the caller keeps ownership. The line that calls the function says which one it is, the way section 5.4 demands. Nothing is copied in silence, and nothing is shared by accident.

Second, the function does its work. The machine jumps to the function's code, and the function works on the values on the pile. Its own short lived values go on top of the pile, above the values the caller handed over, and the caller's values sit below, untouched, waiting for the function to finish. A function can call another function, and that call puts its values on top again, so the pile grows with each call and shrinks with each return. The pile is the record of who called whom, and it is also the memory: the stack from section 6.3, doing the job it was designed for.

Third, the function returns. It puts its result on the pile, and the machine jumps back to the caller, to the line after the call. The function's own values are freed in a single act, by moving the stack pointer back, the way section 6.3 described: the pointer that marks the top of the pile moves down past the values that are done, and those values are forgotten at once. The result stays on the pile, handed to the caller, and the caller takes it from there. The call is over, and nothing of the function remains except its result.

The cost of a call is visible, and it is small. The call itself is one operation in the code, CALL, followed by one byte naming the function, the way a jump is followed by a byte naming where to jump to. The return is one operation, RETURN, with no operand. In the illustrations in this chapter, CALL is `60` and RETURN is `61`, one byte operations in the scheme from section 4.2, and they are illustrations, not the final codes, the way the values in section 4.4 were illustrations. The values passed are the values the caller put on the pile, counted by their sizes from Chapter 5. The work done is the function's own code, counted the same way. There is no hidden setup, no hidden cleanup, no frame the runtime builds in secret. A call costs what it looks like it costs: the jump, the values, the work, the return. The loader knows where each function's code starts, and Chapter 9 defines how that mapping is stored in the binary file.

Why functions matter for the dictionary is worth one paragraph, because it is the same stored-once rule as Chapter 4, applied to code instead of names. A function called fifty times is stored once. Its code sits in one place in the file, and each call is a short reference to it, the way each use of a name is a short reference to the name table. Work through the arithmetic. Take a function of 20 bytes of code, called 50 times. If the code were copied out at every call, the file would hold 50 copies: 50 times 20 is 1,000 bytes. Stored once, the file holds the function once, 20 bytes, plus one 2-byte call at each of the 50 call sites: 20 plus 100 is 120 bytes. The saving is 880 bytes, and the stored form is just over a tenth of the copied form. The more a function is called, the bigger the win, and the most called functions are the ones the program leans on most. The dictionary's rule, store the repeated thing once, is the function's rule too.

One honest note about the cost of a call. A call is not free. The jump, the values on the pile, and the return are real work, and a function called in a tight loop pays that work every time. When a function is small and called in one place, the compiler can inline it: copy its code into the caller, so there is no call at all. That is the optimizer's decision, made when it pays, and Chapter 12 covers it, under the rule that the optimizer only does what pays. The design does not pretend a call is free. It makes the call cheap, counts its cost, and lets the optimizer remove it where removal is the win.

The function also gives the compiler something to check. A function says what it takes, what it returns, and what can fail, and the compiler checks the call against that promise before the program runs. A call that hands the function the wrong kind of value is refused. A call that hands it the wrong number of values is refused. A function that can fail says so, and section 7.3 says what its caller must do about it. The checking happens at build time, once, the way section 6.1 described for ownership, so it costs nothing while the program runs.

One more honest note, about a function that calls itself. A function can call itself, and each call puts its values on top of the pile, so the pile grows with each call. If the calls never stop, the pile grows until it runs out of room, and running out is a clean failure, the way section 6.4 described: an error the program can see, not a crash. The answer is the same as everywhere in this book: the chain of calls is visible in the code, the programmer can read it and see whether it ends, and the compiler counts the stack cost of one call, so the programmer knows what each call costs. A chain that never ends is a bug the programmer can find by reading, not a mystery the machine hides.

A call is inside one program. Calling across programs, asking another program to do work, is a different thing, and it belongs to Chapter 24, where processes and scheduling live. This chapter is the call inside one program: the caller, the function, the pile, and the return.

## 7.2 Branches and loops

A branch is a decision. A loop is repetition. Both are jumps, and this section shows both in the dense form, so the reader can see that a decision and a repetition are the same machine act with a different direction.

A branch says: if this is true, do this, otherwise do that. The program tests a value, a true or false from Chapter 5, and jumps one way or the other depending on what it finds. The test is an operation, the jump is an operation, and both are counted in the code like any other operation. There is no hidden decision maker. The program tests, and the program jumps.

A loop says: do this while a condition holds. The program tests a condition, does the work, and jumps back to the test. The jump back is the loop. The loop ends when the test fails, and the program jumps forward, past the loop, instead of back to the test. A loop is a branch that points backward, and a branch is a loop that never goes back. The two are one mechanism, and the mechanism is the jump.

The worked example from section 4.4 is already a loop, and it is worth reading again with this section in mind. The program prints `hello` three times. The loop test is at position 4: LOAD `count`, LOAD_CONST 3, LESS_THAN. The decision is at position 9: JUMP_IF_FALSE to position 23, the END. When the test fails, when `count` reaches 3, the jump lands on END and the program stops. The repetition is at position 21: JUMP to position 4, back to the test. Two jumps, one forward and one back, and together they are the loop. The positions are counted in the table in section 4.4, and the jump targets check out: `17` in hexadecimal is 23, and `04` is 4. Nothing new is needed to explain a loop. The loop is already there, in the example the reader has already checked.

Now work a branch all the way through, the way section 4.4 worked the loop, with every byte counted. The program checks an age. If the age is under 18, it prints `too young`. Either way, it prints `done`.

First, the text form, in the plain readable form close to Python that section 4.4 used. The exact text syntax of Riza is defined in Chapter 8.

| Line | Text | Bytes |
| --- | --- | --- |
| 1 | `age = 20` | 8 characters + 1 newline = 9 |
| 2 | `if age < 18:` | 12 characters + 1 newline = 13 |
| 3 | `    print("too young")` | 22 characters + 1 newline = 23 |
| 4 | `print("done")` | 13 characters + 1 newline = 14 |
| Total | | 59 bytes |

Check the counts by hand. Line 1 is `age`, a space, `=`, a space, `20`: 3 plus 1 plus 1 plus 1 plus 2 is 8. Line 2 is `if`, a space, `age`, a space, `<`, a space, `18`, `:`: 2 plus 1 plus 3 plus 1 plus 1 plus 1 plus 2 plus 1 is 12. Line 3 is four spaces of indentation, then `print("too young")`, which is 18 characters: 4 plus 18 is 22. Line 4 is `print("done")`, which is 13 characters. Add the four newlines, and the total is 59 bytes.

Now the dense form. It has two parts: the code, and the name table from section 4.3. The code uses the illustrative operation values from section 4.2. They are illustrations, not the final codes: the final codes come from measured frequency and live in Appendix A. The positions are counted in bytes from the start of the code, starting at 0, the way section 4.4 counted them.

| Position | Bytes | Meaning |
| --- | --- | --- |
| 0 | `10 14` | LOAD_CONST 20: put the number 20 on the pile |
| 2 | `30 00` | STORE name 0: take the top value off and store it as `age` |
| 4 | `31 00` | LOAD name 0: put `age` on the pile |
| 6 | `10 12` | LOAD_CONST 18: put the number 18 on the pile |
| 8 | `21` | LESS_THAN: take two values off, put back true if the first is less than the second |
| 9 | `50 0E` | JUMP_IF_FALSE to position 14: if the top value is false, skip the `too young` print |
| 11 | `11 01` | LOAD_STR entry 1: put the string `too young` on the pile |
| 13 | `40` | PRINT: take the top value off and print it |
| 14 | `11 02` | LOAD_STR entry 2: put the string `done` on the pile |
| 16 | `40` | PRINT: take the top value off and print it |
| 17 | `00` | END |

Check the positions. The first operation is 2 bytes, so the second starts at position 2. The fifth operation, LESS_THAN, is 1 byte at position 8, so the sixth starts at 9. The last operation, END, is 1 byte at position 17, so the code is 18 bytes in total. The jump target checks out: JUMP_IF_FALSE carries the byte `0E` in hexadecimal, which is 14 in ordinary counting, the position of the `done` print. A reader can follow the branch by hand: age is 20, the test 20 less than 18 is false, so the jump lands at position 14, the `too young` print is skipped, and `done` prints. If the age were 16, the test would be true, the jump would not fire, and both lines would print.

The name table holds the three pieces of text the program uses: the name `age` and the strings `too young` and `done`. It follows the shape from section 4.3: one byte saying how many entries, then each entry as a length byte followed by the characters.

| Bytes | Meaning |
| --- | --- |
| `03` | Three entries in the table |
| `03 61 67 65` | Entry 0: length 3, then `age` |
| `09 74 6F 6F 20 79 6F 75 6E 67` | Entry 1: length 9, then `too young` |
| `04 64 6F 6E 65` | Entry 2: length 4, then `done` |

The table is 1 byte for the count, plus 4 bytes for the first entry, plus 10 bytes for the second, plus 5 bytes for the third: 20 bytes in total.

Now the totals, side by side:

| Form | Bytes |
| --- | --- |
| Text form | 59 |
| Dense form, code | 18 |
| Dense form, name table | 20 |
| Dense form, total | 38 |

The dense form is 38 bytes against 59 for the text form: 64 percent of the size, a saving of 21 bytes, or 36 percent smaller. The arithmetic is all above, and any reader can recount it. The same honest notes from section 4.4 apply here. The program is small, so the name table looms large: 20 bytes, more than half the dense total, for three entries that would serve a program ten times longer at the same cost. The full binary file, defined in Chapter 9, adds a header of a few fixed bytes on top, and those bytes are not counted here. And the example is hand written, not compiler output, because no compiler exists yet. Chapter 11 builds the first one.

The cost of a branch and a loop is visible, and it is the cost of the jumps. Each jump is an operation in the code, counted like any other. A branch costs one jump each time the program reaches it. A loop costs one jump back each time around, plus one jump out when the test fails. A loop that runs 100 times pays the jump back 100 times, which is why the loop body must be small: the body is paid 100 times too, and every extra operation in the body is paid 100 times with it. The reader can count all of it from the code, because every operation is listed, and the count is the cost. There is no loop the machine runs for free, and no branch that costs nothing. The code says what it costs, and the count says how often.

One honest note about a loop that never ends. A loop whose test never fails runs forever, and the program never moves past it. The design answer is the same as everywhere in this book: the test is visible in the code, the programmer can read it and see whether it can fail, and the program that loops forever is a program the programmer can find by reading. The system does not guess whether a loop will end, because guessing would be hidden work, and section 2.4 forbids hidden work. The loop ends when its test says so, and the test is in the open.

Branches and loops are middle form operations, the simple steps from section 2.3, and Chapter 10 defines them in the shared form, so the tools see the same jumps the compiler sees. The text form a programmer writes for a branch and a loop is Chapter 8's work. This section only needs the shape: a decision is a jump one way or the other, a repetition is a jump back to the test, and both are counted in the code.

## 7.3 Errors as values, not crashes

A program fails. A file is missing. Memory runs out. A number grows past the size its type allows. A disk fills up. These are not surprises. They are the ordinary facts of running on a real machine, and a language that pretends they do not happen is a language that leaves the programmer with no plan for the day they do.

The rule in Riza is one sentence: an error is a value the program holds and decides about, like any other value, not a crash.

An error is a value with two parts. The first is a code, a small number naming what went wrong. One byte covers 256 kinds of error, which is room for every failure the system names, and the standard codes live in the core library, where Chapter 16 keeps them. The second is a message, text in plain words saying what happened, stored once in the name table like any string, so the message costs bytes once no matter how often the error is raised. The error is a value with a stated size, like every value in Chapter 5, and its cost is countable like any other.

A function that can fail returns either its result or an error, and the two never mix. The caller checks which one it got before it uses the result. The check is in the code, visible, a branch of the kind section 7.2 described: if the answer is an error, deal with it; if it is a result, use it. A program that uses a result without checking for the error is refused by the compiler, before it runs, because using a result that might be an error is using a value the program has not looked at. The check costs one branch, and the branch is in the code, so the cost is visible.

The program has three ways to deal with an error, and the choice belongs to the program, because the program knows what its work is worth.

The first is to retry. A file that is busy now may be free in a moment. A network request that failed may succeed on a second try. The program tries again, and if the retry fails, it tries the second or third way. The retry is in the code, and the number of retries is in the code, so the reader can see how many times the program will try before it gives up.

The second is to give up cleanly. The program saves its work, closes its files, and stops, with the error stated in plain words, the way Chapter 15 designs shutdown. Giving up is not a failure of the program. It is the program doing its job when the job cannot be done: telling the user what happened, leaving nothing half done, and exiting with the error stated. A frozen screen tells the user nothing. An error tells the user what happened, and what happened is the truth.

The third is to pass the error up to the caller. A function that gets an error it cannot deal with hands it to its caller, unchanged, the way it hands back a result. The error travels up the chain of calls, through the same return path as any value, until it reaches a function that can deal with it, or the top of the program, where the program stops cleanly with the error stated. Each caller in the chain either deals with the error or passes it on, and the code says which one it does. Nothing jumps. Nothing is thrown. The error moves the way a value moves, one return at a time.

Say plainly what is refused, with a concrete example for each.

Hidden control flow is refused. In some languages, an error throws an exception: the program jumps to a handler somewhere else, skipping every line in between, and the jump is invisible in the code that was skipped. The reader cannot tell by reading a line whether the program will move to the next line or leap to a handler three functions up. That is hidden work, and section 2.4 forbids hidden work. In Riza, the error arrives where the work happened, and the program deals with it there, or passes it up through the return path, in the open. The reader can follow the program line by line, because the program goes line by line.

Silent failure is refused. An error nobody checks is an error nobody knows about. A function that returns an error and a caller that never looks at it is a program that failed in secret, and a secret failure is a program that carries on with a result it does not have. The compiler refuses it: a function that can fail says so, and its caller must deal with the error or pass it on, visibly, or the program does not compile. The check is at build time, once, the way section 6.1 described for ownership, so it costs nothing while the program runs.

Crashes for conditions the program could have handled are refused. A missing file is not a reason to crash. A full disk is not a reason to crash. A number that grows past its size is not a reason to crash: section 5.4 already said silent overflow is refused, and what happens instead is an error the program can see and handle, of the kind this section describes. A crash is what happens when the program has no plan, and this section is the plan. The only thing that stops a program is a failure it cannot handle and cannot pass on, and even then it stops cleanly, with its work saved and the error stated, the way Chapter 15 designs it. It does not freeze, and it does not vanish.

Two callbacks tie this section to the chapters before it. The first is section 6.4: out of memory is an error of this kind. The program asks for memory, the request fails cleanly, and the failure comes back as an error the program can see and handle, in the four steps that section laid out. The second is section 5.4: silent overflow is refused, and the error this section describes is what takes its place. The refusals in Chapter 5 and the rules in Chapter 6 both point here, to the same mechanism: the failure is a value, the value is checked, and the program decides.

The debug log layer in Chapter 17 records errors as they happen, so the evidence is already there when something goes wrong. An error that is handled is still worth recording: what failed, when it failed, and what the program did about it. The log is a plain record, not a hidden trace, and the programmer reads it the way the user reads the error itself: in plain words, in the open.

The honest counterpoint is that error handling costs the programmer some lines. Every call that can fail needs a check, and every check is a line of code. A language that hides those lines feels lighter to write, and that lightness is real. The book's answer is the same as section 5.4's: the line is the price of knowing, and the price is worth paying. A program that checks its errors is a program the reader can trust, because the reader can see what happens when each thing fails. A program that hides its errors is a program the reader cannot trust, because the reader cannot see what happens when anything fails. The programmer writes the program once. The user runs it every day, and the user is the one who meets the error. The check belongs in the program, where it can be seen, not in the user's machine, where it cannot.

The program is put together now: functions that are stored once and called by number, branches and loops built from jumps, and errors as values the program holds and decides about. The next chapter is the human view: the readable text form, where the programmer writes the functions, the branches, and the error checks this chapter designed, and where the round trip from text to binary and back is proved exact.

---

Functions, flow, and errors are the shape of a program. A function is a named block of code, stored once and called by number, with values handed over or lent on the pile and the result handed back, while the call's stack values are freed by moving one pointer. A branch is a jump one way or the other, a loop is a jump back to the test, and both are counted in the code, with a worked branch showing 38 bytes of dense form against 59 of text, every byte listed. And an error is a value, not a crash: a code and a message, checked by the caller, dealt with by retry, by giving up cleanly, or by passing it up, with hidden control flow, silent failure, and crashes for handleable conditions all refused. The compiler checks all of it before the program runs, so the checking costs nothing while the user is waiting.
