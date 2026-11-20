# Appendix E: Checklists: chapter done, system done

This appendix holds the book's two done tests written out as checklists, so they can be used one item at a time. The first is Chapter 3's test for a finished chapter, the test every chapter in this book was held to, opened part by part into the items a reader can check. The second gathers what Chapters 30 to 34 fixed as the first version being done: the image built, the install run, the proof run on the real PC, the first benchmark round, the update and the recovery walked on the real machine, and the version's counts printed. Every item names the chapter it comes from. Nothing here is a new test. These checklists gather the book's own, the way Chapter 3 said they would: so that any reader can check a chapter, and later the system, against the done definition without taking the book's word for it.

The rule of use is the same for both lists. An item is checked only when it is true, all of it, not nearly. A checklist with an unchecked item is an honest state, not a failure to hide: it says exactly what is done and what is not, in the open, which is section 2.4's no hidden work applied to the book itself. A chapter with any box unchecked in the first list is a draft, and it says so. A system with any box unchecked in the second list is not finished, and its record says which box and why. Neither list is ever lowered to meet what stands in front of it. Chapter 3 fixed that for chapters: the test is not quietly lowered to meet the draft. The same holds here.

## The chapter done checklist

Chapter 3, section 3.3, fixed the test in five parts, and all five must pass. The parts stand below in the test's own order, each opened into its checkable items, with nothing added and nothing dropped.

**Part 1: Plain words** (Chapter 3, part 1 of the test)

- [ ] A smart reader with no CS degree can follow the chapter from start to finish.
- [ ] Every technical term is explained the first time it is used, and the book's own words stand defined in Appendix D's glossary.
- [ ] The chapter says what it designs and what it does not build, and never describes Riza as already existing before it is built. (The house rules, Chapter 3.)

**Part 2: Every number measured or sourced** (Chapter 3, part 2 of the test)

- [ ] Every measured number carries its method and its machine, so a reader can repeat it.
- [ ] Every sourced number names its source, so a reader can check it.
- [ ] No number stands with neither a method nor a source behind it.
- [ ] No number is invented. (The house rules, Chapter 3.)

**Part 3: Every claim checkable** (Chapter 3, part 3 of the test)

- [ ] Every claim can be checked one of three ways: the measurement repeated, the reasoning followed, or the source looked up.
- [ ] Where the reasoning is arithmetic, the working is shown, so following it is possible and the count can be redone.
- [ ] Every deferral names the chapter that settles it. The chapters keep this practice throughout: Chapter 30 opens by naming the promises it settles, and Chapter 33 opens by naming the four deferrals it settles.

**Part 4: The scoreboard, where an improvement is claimed** (Chapter 3, part 4 of the test; the table's form is section 1.5's)

- [ ] Where the chapter claims an improvement, the scoreboard table is filled in: the task, today's number, Riza's number, the machine, and the method.
- [ ] The two numbers in a row were measured on the same machine. A number from one machine is never set against a number from another. (Rule 1 of section 1.5, the form's own ground.)
- [ ] Where a row loses, the loss stands in the table as a loss, with the trade stated, not averaged away or left unrun. (Rule 7 of section 1.5.)
- [ ] Where the chapter claims no improvement, it says so, and no table is owed. Chapter 30 states the shape: it claims no measured improvement on real hardware, so no scoreboard table is due.

**Part 5: The target machine** (Chapter 3, part 5 of the test)

- [ ] Anything the chapter says runs on the target machine has been run there, or is marked plainly as waiting for the real-PC run.
- [ ] The waiting claims name Chapter 31 as where their proof happens, and are marked as waiting, not hidden and not fudged.
- [ ] A claim about the target machine that was only tested elsewhere is called what it is: a claim about elsewhere. (Chapter 3, part 5 of the test.)

**The rules behind the test** (Chapter 3)

- [ ] The chapter does not contradict Chapter 2. If it seems to, Chapter 2 wins, and the chapter is the one that changes.
- [ ] The chapter breaks no rule written in Chapter 3. A rule that must change changes in Chapter 3, in the open, with the reason written down, never quietly inside a later chapter.

**The bookkeeping at the close** (the book's practice, gathered)

Two more items stand at every chapter's close. No chapter states them as a rule; every chapter does them, in its closing paragraphs, so they are gathered here as the book's practice, not as a chapter's law.

- [ ] The close restates the test suite's size and the core library's routine count, and both agree with the running totals the chapter before it left.
- [ ] The close names what comes next: the chapter that follows and the work it takes up, including anything this chapter leaves to it.

A chapter that passes all five parts, stands under both rules, and closes with both bookkeeping items counts as done. A chapter that fails any item goes back, the way Chapter 3 fixed: it is not shipped half done, and the checklist is not bent to meet it.

## The system done checklist

This list says what the first version being done means, gathered from Chapters 30 to 34 in the order the proof runs. Its ground is Chapter 3's: everything on it happens on the target machine from section 3.1, the HP All-in-One with the Ryzen 3, the 8 GB, and the spinning drive, and under section 3.2's rules, nothing bought and no network needed. Chapter 3's own test for those rules stands behind the whole list: unplug the network, take away the wallet, and the system still runs.

- [ ] Every chapter of the book passes the chapter done checklist above. (Chapter 3.)
- [ ] The image builds from scratch, from the system's sources and the bootstrap compiler alone, using no Riza binary the build did not just make. (Chapter 30.)
- [ ] The build's report prints, naming every file's real size beside its place's size and the image's total bytes. A file that outgrows its place fails the build, and no partial image is called done. (Chapter 30.)
- [ ] The permission record inside the image reads back in its first form: 8 entries, 67 bytes, each entry wide enough for its program's job and no wider. (Chapter 30.)
- [ ] The install runs on the Mint PC: the image checked six ways before anything is written, the disk taken whole only after the user confirms that state in so many words, and everything written read back byte for byte against the image. (Chapter 33.)
- [ ] The installed machine boots to the shell prompt on its own, and in the proof run the cold boot reaches the prompt in all 5 cold boots. The boot log's four lines are read back and checked against the acts they name, and the shell's list of running programs shows the shell alone. (Chapters 33 and 31.)
- [ ] The kernel's common path fits the 64 KB instruction cache on each core, read from the build's report against section 5.3's size. It fits, or Chapter 22's design goes back. (Chapter 31.)
- [ ] The secret bytes held their shape: 200 answers across a restart, no two alike and no sequence repeated, and the record claims the shape only, at exactly the strength Chapter 29 claimed. (Chapter 31.)
- [ ] The whole test suite runs on the installed system, every test under its own name and its owning chapter, and the run's record reports the result whole: a fail is named with its chapter and what happened, never averaged away into a percentage. (Chapter 31.)
- [ ] The crash tests pass at all 6 cuts: after each cut, every file reads back whole, its old version or its new one, and the rebuilt free record agrees with the folder records. (Chapter 31.)
- [ ] Every refusal in the run answers code 10, and no guarded act goes through that the permission record does not allow. (Chapter 31, testing Chapter 29's design.)
- [ ] Every measurement on Chapter 31's list has a number beside it, or an honest not measured with its reason. A not measured line counts as open, never as a pass. (Chapter 31.)
- [ ] The daily use sitting is worked by hand at the shell, its seven steps in their fixed order, and the user's own account of the sitting stands in the record in the user's words. (Chapter 31.)
- [ ] After any fail, the work went back to the chapter that owns the design, the fix was made in that chapter in the open, the image was rebuilt, and the whole run repeated from the cold boot. (Chapter 31.)
- [ ] The first benchmark round is complete: all 19 rows filled, both sides measured fresh on the target machine in the same round, the method beside every number, losses printed as losses with the trade stated in plain words, and no total row anywhere on the board. (Chapter 32.)
- [ ] An update has been swapped and swapped back on the real machine: its nine checks ran before anything moved, the old version was kept, the swap back ran by the same mechanism, and the scoreboard holds the change's before and its after. (Chapters 33 and 32.)
- [ ] The update order has been cut by power at each point it names, for a program in the general space and for a file at a fixed place, and the machine booted after every cut. (Chapter 33.)
- [ ] A recovery has been walked on the real machine: a damaged fixed file rewritten from the kept image, bytes first and size record last, read back matching; the permission record restored at its first form, no entry wider without the user's own approval; and every file in the general space left byte for byte untouched. (Chapter 33.)
- [ ] The version's four counts are printed in its record: the image's total size, the core library's routine count, the fixed region's blocks, and the suite's size, each matching the build's own report. (Chapter 34.)

The list closes with Chapter 34's own definition, which is the standard the list serves: the first version is finished when its proof runs, Chapter 31's run and Chapter 32's first round, not when its list of features feels long.

---

These checklists add no new test and no new threshold: every item above is a chapter's own words gathered for use, and where a chapter and a checklist item ever disagree, the chapter wins. They add no tests and no routines: the test suite stays at 217 tests, and the core library stays at 39 routines, the closing counts Chapter 34 left as the first version's state.
