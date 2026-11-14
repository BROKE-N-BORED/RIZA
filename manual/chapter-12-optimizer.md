# Chapter 12: The optimizer: only what pays

Chapter 11 built the compiler. The smallest compiler that works produces the middle form straight from the text, step for step, with no cleverness. A program that computes a value and never uses it keeps the computation. A program that adds two numbers the compiler already knows keeps the addition. A small function called in one place keeps the call. The compiler does the plain thing, and correctness is easier to prove when the compiler does the plain thing.

This chapter is the clever part, kept separate. The optimizer sits between the compiler's stage two, which produces the middle form, and stage three, which writes the file. It takes the middle form stage two wrote, improves it, and hands the improved form to stage three. It is a stage of the compiler, not a separate tool, because it works on the compiler's own form. The compiler stays plain and provable. The optimizer holds the cleverness, in one place, where it can be checked. And the optimizer can be skipped, the way section 2.4 requires: a programmer who asks for the plain form gets it.

The rule for what the optimizer does is in the chapter's title: only what pays. An optimization pays when the saving is proven, not guessed. Proven means the proof is in the code itself. The dead step is unreachable in the code. The constant is written in the code. The inlining arithmetic is counted from the code. A saving that depends on a hope about how the program will be used, or about what data it will meet, is not proven, and the optimizer does not do it. Section 12.2 names what that rule leaves out.

There are two sections. The first names the three optimizations that pay: dead code, constants, inlining, each with what it saves, how the compiler proves it is safe, and what it must not touch. The second names what the book skips on purpose, each with why.

One warning, stated up front the way Chapters 4 through 11 stated their own. This chapter designs the optimizer. It does not build it. No optimizer exists yet, and no claim in this chapter says one does. The chapter also claims no measured improvement on real hardware, so no scoreboard table is due. The numbers here are arithmetic on the chapter's own examples, counted by hand. Whether the three optimizations actually pay on the real PC, in time and in energy, is a measurement, and it belongs to Chapter 31, with the comparison against today's stack in Chapter 32.

## 12.1 Dead code, constants, inlining

The optimizer does three things. Each one is proven by the code itself, and each one has a list of what it must not touch. Take them one at a time, then work one program through all three.

Dead code is code the compiler can prove never runs, and values computed and never used. Two flavors, one rule: if the compiler can prove, from the code itself, that a step never runs or a value is never used, the step goes.

The first flavor is a step that never runs. The optimizer walks the middle form from the entry point, follows every jump, and marks every step it can reach. A step no jump lands on, and no step falls through to, never runs. After an unconditional JUMP, the next step runs only if some jump lands on it; if none does, it never runs. A branch whose test the optimizer knows is always false, after the constants pass below, has a skipped side that never runs. The proof is the walk: reachability is a property of the code, and the code says what is reachable.

The second flavor is a value computed and never used. A name stored and never loaded is a value nobody reads. The store goes, and the computation that produced the value goes with it. Section 11.1 named this case: a program that computes a value and never uses it keeps the computation in the plain compiler's output, because removing it is the optimizer's job. This is that job.

A jump whose target is the very next operation goes too, because the program arrives at the same place without it. The jump does nothing, and a step that does nothing is dead by the same rule.

Say plainly what dead code removal must not touch, because the list is the safety of the whole pass. It must not remove an error check. Section 7.3 made the error check the program's promise: a result used without checking for the error is refused by the compiler, and the check is the branch that keeps the promise. Whether an error happens is known only at run time, when the file is missing or the memory runs out, so the optimizer can never prove a check is dead. It never tries. It must not remove a call that touches the world. PRINT is the one this book has named, and the library calls are the rest. A PRINT whose result nobody reads is not dead code, because the printing is the effect. The act is the point, and the act stays. And it must not move a free across a use. Removing a computation removes the value, so there is nothing to free, and the values that remain are freed in the same order they were. The optimizer changes the code, not the lifetimes.

Constants are values the compiler can compute at build time, computed once, not every time the program runs. Two moves, both proven by the code itself.

The first move is folding. A computation whose values are all known at build time is computed by the compiler, and the answer is written into the code. `2 + 3` becomes `5`. The three steps, LOAD_CONST 2, LOAD_CONST 3, ADD, become one step, LOAD_CONST 5. The computation is gone, and the answer is in its place. The proof is in the code: the values 2 and 3 are written there, and their sum is 5, which any reader can check.

The second move is carrying. When a name is assigned a known value once, and never changed, the optimizer carries the value to the places that load the name. `limit = 5`, and later LOAD limit becomes LOAD_CONST 5, because the optimizer can see, in the code, that limit is 5 and nothing between the assignment and the use changes it. The proof is the code: one assignment, no other assignment, and the use. If the name is assigned twice, or changed in a loop, the optimizer does not carry.

Say plainly what constant folding must not touch. It must not touch a value read from outside. A number read from a file, the time asked from the clock, a word typed by the user: none of those is known at build time, and the optimizer never guesses them. A guess about the outside world is a guess, and the optimizer does not guess. It must not fold a computation that overflows. Section 5.4 refuses silent overflow: adding 1 to 255 in a 1-byte number does not silently become 0, and what happens instead is an error the program can see and handle. An overflowing computation has no exact answer in its type, so the optimizer leaves it alone, and the program gets its error at run time, the way section 5.4 designed it. Folding `255 + 1` into 0 would be a lie, and the optimizer does not lie.

The two passes feed each other, and that is why they run in order, constants first and dead code second. When a value is carried to all its uses, the store that made it is dead, and the dead code pass removes it. When a test folds to a known answer, the branch it guards is dead on one side, and the dead code pass removes the side that never runs. The passes run in order, and they run again if a pass changed anything, until a full round changes nothing. The worked example below shows why: inlining puts two constants next to an ADD, and the second round folds them.

Inlining is a small function called in one place, written into the caller, saving the call and the return. Section 7.1 named it: when a function is small and called in one place, the compiler can inline it, copying its code into the caller, so there is no call at all. That is the optimizer's decision, made when it pays, and this is where it pays.

How it works on the pile is the key, and it is simpler than it looks. The caller puts the arguments on the pile, the way section 7.1 described. The function's body, as the plain compiler wrote it, loads its parameters from their slots, because a separate function cannot see the caller's pile. Inlined, the body runs where the call was, and the arguments are already on the pile, in the body's own working order. So the body's parameter loads go with the call: they would put on the pile exactly the values the caller already put there. The body's remaining operations move into the caller as they are, the RETURN goes, because there is nowhere to return to, and the function body is removed from the file, because nothing calls it anymore.

The rule for when inlining pays is arithmetic, counted from the code. Inline only when the total code gets smaller or stays the same, counting every call site. A function called in one place, with a body smaller than the call site plus the body, pays. A function called in fifty places, with a body copied fifty times, does not, and section 12.2 works that arithmetic. The rule is a count, not a guess, and the count is made before the inlining happens.

Say plainly what inlining must not touch. It must not inline a function that calls itself. Section 7.1 named the self-calling function: each call puts its values on the pile, and the pile grows with each call. Inlining a function into itself would copy the body into the body, forever, and the copy would never end. A recursive function is never inlined. And it must not change the ownership rules. The inlined body's values are used and consumed in the same order, in the caller, as they were in the function, and the optimizer checks that the changed form still satisfies section 6.1: no value used after it is freed, no value left without an owner. A changed form must still satisfy the rules, and the optimizer checks it the way stage one checked the text.

Now work one small program through the optimizer, pass by pass, the way sections 4.4 and 7.2 worked their examples, with every byte counted. The program uses the add function from section 8.1, a value never used, and a constant the compiler can compute.

First, the text form, with the byte count of each line, newlines included the way section 4.4 counted them.

| Line | Text | Bytes |
| --- | --- | --- |
| 1 | `define add(first, second):` | 26 characters + 1 newline = 27 |
| 2 | `    total = first + second` | 26 characters + 1 newline = 27 |
| 3 | `    return total` | 16 characters + 1 newline = 17 |
| 4 | `spare = 99` | 10 characters + 1 newline = 11 |
| 5 | `limit = 2 + 3` | 13 characters + 1 newline = 14 |
| 6 | `answer = add(limit, 4)` | 22 characters + 1 newline = 23 |
| 7 | `print(answer)` | 13 characters + 1 newline = 14 |
| Total | | 133 bytes |

The program prints 9. Limit is 2 plus 3, which is 5. Add of 5 and 4 is 9. Print of answer prints 9. The optimizer must keep that answer. Every pass below is checked against it: the program prints 9 before, and it prints 9 after.

The plain compiler's middle form, stage two's output, step for step, is 19 operations and 33 bytes of code. The code uses the illustrative operation values from sections 4.2 and 7.1, not the final codes, which come from measured frequency and live in Appendix A. The main program comes first, at the entry point, and the function body follows it.

| Position | Bytes | Meaning |
| --- | --- | --- |
| 0 | `10 63` | LOAD_CONST 99: put the number 99 on the pile |
| 2 | `30 04` | STORE name 4: take the top value off and store it as `spare` |
| 4 | `10 02` | LOAD_CONST 2: put the number 2 on the pile |
| 6 | `10 03` | LOAD_CONST 3: put the number 3 on the pile |
| 8 | `20` | ADD: take two values off, put their sum back |
| 9 | `30 05` | STORE name 5: take the top value off and store it as `limit` |
| 11 | `31 05` | LOAD name 5: put `limit` on the pile |
| 13 | `10 04` | LOAD_CONST 4: put the number 4 on the pile |
| 15 | `60 00` | CALL function 0: call `add` |
| 17 | `30 06` | STORE name 6: take the top value off and store it as `answer` |
| 19 | `31 06` | LOAD name 6: put `answer` on the pile |
| 21 | `40` | PRINT: take the top value off and print it |
| 22 | `00` | END |
| 23 | `31 01` | LOAD name 1: put `first` on the pile |
| 25 | `31 02` | LOAD name 2: put `second` on the pile |
| 27 | `20` | ADD: take two values off, put their sum back |
| 28 | `30 03` | STORE name 3: take the top value off and store it as `total` |
| 30 | `31 03` | LOAD name 3: put `total` on the pile |
| 32 | `61` | RETURN: put the result on the pile and jump back to the caller |

Check the positions. The main program is 13 operations, 23 bytes, positions 0 to 22. The function body is 6 operations, 10 bytes, positions 23 to 32. The code is 33 bytes in total, 19 operations.

The name table holds the seven names, in the shape section 4.3 defined: one byte saying how many entries, then each entry as a length byte followed by the characters.

| Bytes | Meaning |
| --- | --- |
| `07` | Seven entries in the table |
| `03 61 64 64` | Entry 0: length 3, then `add` |
| `05 66 69 72 73 74` | Entry 1: length 5, then `first` |
| `06 73 65 63 6F 6E 64` | Entry 2: length 6, then `second` |
| `05 74 6F 74 61 6C` | Entry 3: length 5, then `total` |
| `05 73 70 61 72 65` | Entry 4: length 5, then `spare` |
| `05 6C 69 6D 69 74` | Entry 5: length 5, then `limit` |
| `06 61 6E 73 77 65 72` | Entry 6: length 6, then `answer` |

The table is 1 byte for the count, plus 4 plus 6 plus 7 plus 6 plus 6 plus 6 plus 7 for the entries: 43 bytes in total. The dense form is 33 bytes of code plus 43 bytes of table, 76 bytes. The complete file, in the shape Chapter 9 defined, is 33 bytes of header, 33 of code, 0 of data, and a dictionary section of 50 bytes, the 43 byte name table plus a 6 byte function mapping plus a 1 byte comments count: 116 bytes in total.

Pass one, constants. Fold `2 + 3` to `5`: the three steps at positions 4, 6, and 8 become one step, LOAD_CONST 5. Carry limit to its use: limit is assigned once, never changed, so LOAD limit at the call site becomes LOAD_CONST 5. The function body is unchanged.

| Position | Bytes | Meaning |
| --- | --- | --- |
| 0 | `10 63` | LOAD_CONST 99 |
| 2 | `30 04` | STORE name 4: `spare` |
| 4 | `10 05` | LOAD_CONST 5 |
| 6 | `30 05` | STORE name 5: `limit` |
| 8 | `10 05` | LOAD_CONST 5 |
| 10 | `10 04` | LOAD_CONST 4 |
| 12 | `60 00` | CALL function 0: `add` |
| 14 | `30 06` | STORE name 6: `answer` |
| 16 | `31 06` | LOAD name 6: `answer` |
| 18 | `40` | PRINT |
| 19 | `00` | END |

The main program is 11 operations, 20 bytes. With the unchanged body, the code is 17 operations, 30 bytes. The fold saved 2 operations and 3 bytes. The carry saved nothing in size, both are 2 bytes, but it made the next pass possible, because limit is no longer loaded anywhere.

Pass two, dead code. `spare` is never used, so LOAD_CONST 99 and STORE spare go. `limit` is never loaded now, after the carry, so LOAD_CONST 5 and STORE limit go. The name table loses the two entries, and the rest are renumbered: add 0, first 1, second 2, total 3, answer 4. The function body is unchanged.

| Position | Bytes | Meaning |
| --- | --- | --- |
| 0 | `10 05` | LOAD_CONST 5 |
| 2 | `10 04` | LOAD_CONST 4 |
| 4 | `60 00` | CALL function 0: `add` |
| 6 | `30 04` | STORE name 4: `answer` |
| 8 | `31 04` | LOAD name 4: `answer` |
| 10 | `40` | PRINT |
| 11 | `00` | END |

The main program is 7 operations, 12 bytes. With the unchanged body, the code is 13 operations, 22 bytes. The pass saved 4 operations and 8 bytes of code, and 12 bytes of name table, the `spare` entry and the `limit` entry. The table is 31 bytes now.

Pass three, inlining. Add is called in exactly one place, so the count is made: the call site is 8 bytes and the body is 10 bytes, 18 bytes together, and the inlined form will be smaller, so it pays. The CALL goes. The RETURN goes. The body's LOAD first and LOAD second go, because the arguments, 5 and 4, are already on the pile where the body expects them. The body's ADD, STORE total, and LOAD total move into the caller. The function body is removed from the file, and its names, `add`, `first`, and `second`, leave the table. The remaining entries are renumbered: total 0, answer 1.

| Position | Bytes | Meaning |
| --- | --- | --- |
| 0 | `10 05` | LOAD_CONST 5 |
| 2 | `10 04` | LOAD_CONST 4 |
| 4 | `20` | ADD |
| 5 | `30 00` | STORE name 0: `total` |
| 7 | `31 00` | LOAD name 0: `total` |
| 9 | `30 01` | STORE name 1: `answer` |
| 11 | `31 01` | LOAD name 1: `answer` |
| 13 | `40` | PRINT |
| 14 | `00` | END |

The code is 9 operations, 15 bytes, and there are no functions left. The pass saved 4 operations and 7 bytes of code, and 17 bytes of name table. The table is 14 bytes now. One honest note: STORE total followed at once by LOAD total is a store and a load of the same value, and a sharper optimizer would drop both. This one does not, because that is a fourth optimization, and the chapter's rule is three.

The passes run again, because a pass changed something. Inlining put two constants next to an ADD, so the second round folds 5 plus 4 to 9, carries 9 to the print, and the stores of total and answer go dead, with the loads that fed them. The program settles at three operations.

| Position | Bytes | Meaning |
| --- | --- | --- |
| 0 | `10 09` | LOAD_CONST 9 |
| 2 | `40` | PRINT |
| 3 | `00` | END |

The code is 3 operations, 4 bytes. The name table is empty, 1 byte, the count saying 0. A third round changes nothing, so the optimizer stops. The program prints 9, the way it did before every pass.

Now the totals, stage by stage, with the file totals in the shape Chapter 9 defined.

| Stage | Code operations | Code bytes | Name table | Dense total | File total |
| --- | --- | --- | --- | --- | --- |
| Plain compiler | 19 | 33 | 43 | 76 | 116 |
| After constants | 17 | 30 | 43 | 73 | 113 |
| After dead code | 13 | 22 | 31 | 53 | 93 |
| After inlining | 9 | 15 | 14 | 29 | 64 |
| Fixed point | 3 | 4 | 1 | 5 | 40 |

Read the result honestly, because a single small example proves less than it might look like. This program folds completely because it has no inputs: every value is known at build time, so the optimizer can prove the whole program is just printing 9. A real program reads files, takes input, and loops on data, and most of it does not fold. The example shows the mechanism, the three passes and the way they feed each other, not a promise that every program shrinks to 3 operations. What the example does prove is the rule: the optimizer changed the middle form, from 19 operations to 3, and the result did not change. The program prints 9 before, and it prints 9 after.

The same cascade works on the branch from section 7.2, which checks an age of 20 against 18 and prints `done`, skipping `too young`. The plain form is 11 operations, 18 bytes of code, a 20 byte name table, 38 bytes dense. Fold it: age is 20, a known value, so the test `20 < 18` folds to false. The JUMP_IF_FALSE always jumps, so the `too young` print never runs, and it goes. Age is never loaded, so its store goes. The jump itself lands on the very next operation, so it goes too. What is left is LOAD_STR `done`, PRINT, END: 3 operations, 4 bytes of code, a 6 byte name table holding only `done`, 10 bytes dense. The program still prints `done` and skips `too young`, because the test was always false.

One more saving belongs in this section, because Chapter 10 named it and gave it to the optimizer. A comparison whose result is only ever tested by the next jump can skip the widening and the write back, because the jump can read the compare's flags directly. The optimizer's part is the proof: it walks the middle form and shows that the comparison's result is used only by the jump that follows it, and by nothing else, and with that proof in hand the translation skips the widening and the write back. LESS_THAN is 5 native instructions and JUMP_IF_FALSE is 3, 8 together. Fused, the pair is 4: pop the second value into a register, compare the first value with it, pop the first value into a spare register, because the pile still holds it after the compare, and jump if the first is not less than the second. The hello loop's translation drops from 23 native instructions to 19. The middle form keeps both steps, and the saving is in the native code. The comparison follows the type the operation carries, signed or unsigned, the way section 10.3 described.

Now the tests, because an optimizer that changes the middle form must prove it changed nothing else. The optimizer's output passes the same suite section 11.3 built, 33 tests, with each test running at the stage it belongs to. The 17 operation tests run on optimized programs, and expect the same answers: ADD still gives 5, add of 3 and 4 still gives 7, the hello loop still prints `hello` three times, the branch still prints `done`. The 7 refusal tests happen in stage one, before the optimizer runs, so the refusals are unaffected. The 6 validity tests run on the optimized files, because the optimizer must produce a valid file: it recomputes every jump target and every table number after each pass, so the positions still add up, the sections still sit inside the file, and every reference still points inside it. And the end-to-end test still expects the hello loop's 72 byte file, because the optimizer changes nothing in it: no dead code, no foldable constants, no functions. The optimizer must not change a program it has nothing to improve, and the hello loop is the test of that.

A careful reader will ask about the round trip, and it needs an honest answer. Section 8.2 promised that text to binary and back produces the same text, byte for byte. The optimizer changes the middle form by design: the `spare` line is gone, `2 + 3` is `5`, the add function is gone. So the round trip is a property of the compiler's middle form, tested in section 11.3 on the compiler's output, before the optimizer runs. The text view the programmer sees comes from the form the compiler wrote, which is the program the programmer wrote. The optimized file is what runs. And the editor can show the optimized form on request, the way section 8.3 requires, because anything hidden can be shown. The costs the editor shows next to a line are the costs of the optimized form, because those are the costs the machine pays.

## 12.2 What we skip on purpose

The optimizer does three things. Everything else it could do, it skips, and this section names the four biggest skips and why. The test for each is the same as the test for the three that pay, applied in reverse: the saving is a guess, the cost is certain, or the code gets bigger and the cache pays. A skip is a decision, stated here in the open, with its reason.

Loop unrolling is writing a loop out several times to skip the test. Instead of a body and a test that runs once per pass, the body is written out four times with one test per four passes. The saving is real in one sense: three tests and three jumps per four passes go away, and a jump is one native instruction, the way section 10.3 counted it. But the saving is counted in instructions, and the time of one instruction is not a number this book has measured. The cost is counted in bytes, and bytes are certain. The hello loop's body is 10 bytes: LOAD_STR, PRINT, LOAD, LOAD_CONST, ADD, STORE. Written out four times, it is 40 bytes instead of 10, 30 extra bytes, for a saving in instructions whose time is unmeasured. And the bigger code takes more room in the instruction cache, 64 KB per core from section 5.3, pushing other code out. For a loop whose trip count is not known at build time, unrolling needs a remainder loop for the leftover passes, which adds more code still. The optimizer does not pay certain bytes for unmeasured time.

Vectorization is doing several values at once with special instructions, instead of one value at a time. The saving depends on the data, not the code: how many values there are, and whether they sit together in memory. The compiler cannot see the data at build time. A vector built to hold eight values gains nothing on a loop over three, and the setup, loading the vector and handling the leftover values, costs code whether the data cooperates or not. The saving is a guess about data the compiler cannot see, and the cost is certain code. Vector operations would also be new middle-form operations, growing the shared form, and section 2.4 keeps the form small. The optimizer does not guess about data.

Aggressive inlining is inlining a large function, or a function called in many places. Section 7.1 worked the arithmetic: a function of 20 bytes of code, called 50 times, stored once, costs 20 bytes for the function plus one 2-byte call at each of the 50 call sites, 120 bytes in total. Inlined at all 50 sites, the file holds 50 copies of the 20 byte body, 1,000 bytes. The call it saves is 2 bytes per site. The body it copies is 20 bytes per site. The net is 18 extra bytes per site, 880 extra bytes in total, and the cache pays for the bigger code. The rule in section 12.1 is the line between inlining that pays and inlining that does not: inline only when the total code gets smaller or stays the same, counting every call site. Add, called once, pays. A 20 byte function called 50 times does not. The optimizer counts before it copies.

Speculative optimization is guessing what the program will do and preparing for the guess: laying out the code so the guessed path is straight, or preparing fast code for a value guessed to be of one kind, with a slow fallback if the guess is wrong. A right guess saves. A wrong guess costs more than no guess at all, because the fallback code must exist in the file too, more bytes, and the check that the guess still holds costs a test every time the program runs that path, more work every time. The saving exists only when the guess is right. The cost exists always. And the compiler cannot know at build time whether the guess will be right, because the guess is about how the program will be used, not about what the code says. It is a bet, and the optimizer does not bet. A speed that depends on a guess the programmer cannot see is also hidden work, and section 2.4 forbids hidden work.

The closing rule is the chapter's title. An optimization that cannot prove its saving, from the code itself, is an optimization the book skips. The three that pay are proven by the code: the dead step is unreachable in the code, the constant is written in the code, the inlining arithmetic is counted from the code. The four that are skipped are not proven: the saving is a guess about time, about data, or about use, and the cost is certain bytes. Chapter 31 measures whether the three that pay actually pay on the real PC, in time and in energy, because section 2.5 says nothing counts without a real number from real hardware, and a proof in the code is not a measurement on the machine. And the skips are decisions, not laws: if a measurement proves a skip wrong, the decision changes in the open, with the numbers, the way section 2.5 requires.

---

The optimizer is the clever part of the compiler, kept separate so the compiler stays plain and provable: it sits between stage two and stage three, works on the middle form, and does only what pays. Three optimizations pay, each proven by the code itself: dead code, a step that never runs or a value never used, removed, with error checks and world-touching calls never touched; constants, a computation known at build time computed once and carried to its uses, with outside values and overflowing computations left alone; and inlining, a small function called in one place written into the caller, only when the total code gets smaller, counting every call site. One program worked through by hand shows the three passes feeding each other, from 19 operations and a 116 byte file to 3 operations and a 40 byte file, still printing 9, and the same suite of 33 tests proves the result did not change. Four optimizations are skipped on purpose, loop unrolling, vectorization, aggressive inlining, and speculative optimization, because each one asks the book to pay certain bytes for a guessed saving. Chapter 31 measures whether the three that pay actually pay.
