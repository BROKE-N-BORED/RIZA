# Chapter 3: Rules and limits for this build

Chapter 1 was the diagnosis. Chapter 2 was the idea. This chapter is the rules.

An idea without rules drifts. A long build has a hundred small temptations: a feature that would be easier if the machine were bigger, a tool that would be quicker if it cost money, a claim that would sound better without a number behind it. Each temptation on its own looks small. Together they are how a project built on clear principles ends up looking like everything else. The rules in this chapter exist to stop that drift. They are written down now, at the start, so they cannot be quietly bent later when a feature is tempting.

There are three sections. The first names the machine the system is built for, part by part, and says what each part means for the design. The second names the budget, which is zero, and the network rule, which is that the system works without one. The third names what counts as a finished chapter, so that this book is held to the same standard as the system it describes.

The rules in this chapter are not suggestions. A design that breaks one is wrong, and the design changes, not the rule. If a rule ever has to change, it changes here, in the open, with the reason written down, so that every reader can see what changed and why. A rule changed in secret is not a rule. It is a habit.

## 3.1 Target machine: 8GB Ryzen 3 Mint PC, CPU only

The target machine is one specific computer, not a class of computers. It is the machine on the desk. Everything in this book is built for it, measured on it, and judged by whether it runs well on it. Here it is, part by part:

| Part | What it is |
| --- | --- |
| Machine | HP All-in-One |
| Processor | AMD Ryzen 3 3250U, 2 cores, 4 threads |
| Graphics | Radeon Vega 3, integrated |
| Memory | 8 GB DDR4 |
| Storage | 1 TB spinning hard drive |
| Screen | 1366x768 |
| Operating system | Linux Mint |
| Python | 3.12.3 |
| GPU | None |

Each part of that table is a shape the design must fit. Work through them one at a time.

Two cores means the machine can truly do two things at once, and no more. A core is a worker inside the processor. Two cores are two workers. The four threads mean each worker can juggle two lines of work, switching between them quickly, but switching is not doing. Four lines of work on two cores still take the time of two workers. So the ceiling on parallelism, on doing many things at the same time, is two. A design that needs eight cores to be fast is not a design for this machine. A design that is fast on two cores is fast everywhere. Chapter 24, on processes and scheduling, works inside this ceiling: fair sharing on 2 cores, not a scheduler written for a server.

Eight gigabytes of memory means every megabyte has an owner. The whole system shares it: the operating system, the programs, and the graphics, because integrated graphics takes its share from the same memory instead of having its own. There is no spare pool to hide in. Section 1.2 showed what happens when programs forget this: one fresh editor window can take about a twentieth of the whole machine's memory before a single file is open. On this machine, that kind of appetite is not a detail. It is the difference between a system that runs and a system that crawls. Chapter 6 builds memory management that accounts for every byte, and Chapter 25 carries the same rule into the kernel. Nothing in Riza is allowed to treat memory as free, because on this machine it is the scarcest thing there is.

The spinning hard drive means the disk is fast at one kind of reading and slow at another. A spinning drive reads fastest in long sequential runs, one stretch of data after another, and slowest when it must jump between files scattered across the disk. Each jump costs a seek, and section 1.4 already counted the cost: seeks on a spinning drive take milliseconds each. That is why cold starts hurt on this machine. An interpreter startup is a sequence of small reads scattered across the disk, and each one pays a seek. The drive holds a full terabyte, so room is not the problem. Speed is the problem, and the shape of the reading is what sets the speed. So files in Riza are laid out to be read in long runs, few files instead of many, and a program starts by reading, not by hunting. Chapter 9 defines a binary format built for that shape, and Chapter 15 designs startup around it.

No GPU means everything runs on the processor. There is no separate graphics chip to hand work to, no graphics compute to lean on, no hardware acceleration to assume. The integrated Vega 3 draws the screen, and that is its job. Every calculation in the system, from compiling to searching to drawing a window, runs on the two cores. That is a limit, and it is also a simplification: there is one processor to understand, one thing to measure, and no second chip with its own memory, its own drivers, and its own failure modes. The design does not fight that limit. It takes it as given and builds for the CPU alone.

CPU only also sets the speed of the build itself. The compiler runs on this machine, on two cores, and a compile that takes an hour is a compile nobody runs twice. So the tools in this book are built to be fast on this machine, not fast on a machine with more cores. Chapter 11 builds the first compiler with that in mind: small enough to compile quickly, simple enough to understand in full. A toolchain that is slow on the target machine is a toolchain that slows down every chapter that follows it.

The screen is 1366 pixels wide and 768 tall. That is a small screen by today's standards, and it sets the shape of every window in the system. A window designed for a bigger screen is a window the user cannot use: buttons fall off the edge, text gets cut, the user scrolls to find what should have been in front of them. Every window in Riza must fit and be usable at 1366x768, with text readable at that size, or it does not ship. This is not a complaint about the screen. It is a rule that keeps the interface honest. If a window cannot fit on the screen it was built for, the window is wrong.

Linux Mint and Python 3.12.3 are the starting ground. The machine as it stands today is where the build begins, not a machine with anything added. The bootstrap compiler, the first compiler, is built here, in an existing language, on what is already installed. Chapter 11 does that work. Nothing in the early chapters assumes a tool the machine does not already have, because the build has to start from the machine as it is.

Now the key point, and it is the point of this whole section: constraints drive the design, they do not limit it. A limit is a shape to build around, not a wall to stop at. Inside these specs, anything is fair game. A custom language is fair game. Custom tools are fair game. A custom binary format is fair game. What is not fair game is a design that pretends the machine is bigger than it is. If a design needs more than this machine has, the design is wrong, not the machine. Building around a limit beats declaring it impossible, every time. The history of computing is full of machines smaller than this one doing remarkable work, because their builders treated the limits as the design.

One more thing ties this section to the two chapters before it. The target machine is also the measuring machine. Rule 1 of section 1.5 says a number measured on one machine is never compared with a number measured on another, and section 2.5 says nothing counts without a real number from real hardware. The machine named in this section is the machine those rules point at. The spec and the proof are the same rule: build for this machine, measure on this machine, and let the numbers from this machine be the judge. Appendix F holds the full hardware notes for the Mint PC, so that the spec is written down in one place and stays the same for the whole build.

## 3.2 Hard $0, offline first

The budget for this build is zero, and it stays zero. Hard $0 means no paid services, no subscriptions, no cloud bills, no trial that turns into a charge after thirty days. Everything in the build is free forever, or it is built by hand. There is no step in the build that asks for a card, a payment, or a purchase. If a tool costs money, it is not used. If a service needs a subscription, it is not used. The rule is not "cheap." The rule is zero.

Say plainly what this rules out. Paid APIs are out, because an API that charges per call makes every run of the system cost money, and a system that costs money per run is a system the user cannot afford to use freely. Cloud-only tools are out, because a tool that runs on someone else's computer is a tool the user does not own. Free tiers are out where the free part can be pulled, because a free tier is a gift that can be taken back, and a system built on one is a system with a trapdoor in it. Anything that stops working when the money stops is out, because the money was never there to begin with.

Say plainly what this does not rule out. Free open source tools are in: compilers, editors, libraries, and utilities that anyone can download, use, and keep, with no charge and no expiry. The user's own local models are in: models that run on this machine, on the CPU, with no server and no bill. Anything that runs on the machine itself, using only what the machine already has, is in. The rule is not against tools. It is against dependence. A tool the user owns is a tool. A tool the user rents is a leash.

Offline first means the system works with no network at all. No downloads to start. No license check. No activation. No update that must arrive before the program runs. No telemetry phoning home to a server the user never chose. Pull the network cable out, and the system still runs, still builds, still does its work. If the network is there, the system can use it: to fetch a package, to check for an update the user asked for, to send a message the user wrote. But the network is a bonus, never a requirement. A system that needs the network to start is a system that stops working when the network does, and the network is not always there.

The two rules belong together, because they protect the same thing: independence. A system that needs someone else's server or someone else's money is not yours. It works on someone else's terms, it stops on someone else's schedule, and it can be changed or taken away without asking you. Riza is built to belong to the person using it. The machine is theirs. The tools are theirs. The work is theirs. Nothing in the system phones home, because home is the machine it runs on.

There is a second reason the budget is zero, and it is about time. This build takes months, and the system it produces has to last years. A service that is free today can charge tomorrow. A trial can end mid-build. A company can change its terms, shut down, or be bought, and a system that depends on it breaks the day that happens. A budget of zero removes that risk entirely. Nothing in the build can be taken away, because nothing in it belongs to anyone else. The system built a year from now is built with the same tools, at the same price, as the system built today.

The test for both rules is simple, and any reader can run it. Unplug the network. Take away the wallet. The system still runs. If it does not, it fails the rule, and the part that fails is redesigned until it passes or removed. That test connects to the rule in section 2.4: no hidden work. Telemetry is hidden work. A license check is a phone call home the user did not ask for. A background update check is work on a schedule the user cannot see. The budget rule and the network rule and the no hidden work rule are three ways of saying the same thing: the machine works for the person in front of it, and for nobody else.

## 3.3 How a chapter counts as done

This book is held to the same standard as the system it describes. A chapter is not done because it is written. It is done when it passes the test in this section. The test has five parts, and all five must pass.

1. Written in plain words. A smart reader with no CS degree can follow it. Every technical term is explained the first time it is used. A chapter that hides behind jargon has not explained its subject. It has only described it to people who already know it.
2. Every number measured or sourced. A measured number carries its method and its machine, so a reader can repeat it. A sourced number names its source, so a reader can check it. No number appears without one of the two. A number with no method and no source is a guess wearing a number's clothes.
3. Every claim checkable. A reader can repeat the measurement, follow the reasoning, or look up the source. A claim nobody can check is not a claim. It is a boast, and boasts do not belong in a manual.
4. Scoreboard table filled in, where the chapter claims an improvement. The table is the form section 1.5 defines: the task, today's number, Riza's number, the machine, and the method. A chapter that claims a win and leaves the table empty has not finished its work. A chapter that makes no improvement claim needs no table, but the moment it says "better," the table is due.
5. Anything it says runs on the target machine has been run there, or is marked plainly as waiting for the real-PC run. Chapter 31 is where the real-PC runs happen: boot, tools, daily use. Until a claim has been through Chapter 31, it is marked as waiting, not hidden and not fudged. A claim about the target machine that was only tested elsewhere is a claim about elsewhere, and the book says so.

A chapter that claims what it cannot show is not done. It is a draft, and it says so. There is no shame in a draft. Drafts are how chapters get written. The shame would be in calling a draft done, because a reader who trusts a chapter that is not finished is a reader who has been misled, and this book does not mislead its reader.

When a chapter fails the test, it goes back. It is not shipped half done, and the test is not quietly lowered to meet the draft. The book grows one finished chapter at a time, and one finished chapter is worth more than ten drafts, because a finished chapter can be trusted and a draft cannot. The reader should be able to open any chapter in this book and know it passed the test, without checking. That is what the test is for.

Two more rules govern the chapters themselves. The first is the rule from Chapter 2: if a later chapter contradicts Chapter 2, Chapter 2 wins, and the later chapter is wrong. Chapter 2 is the idea, and the idea does not change to suit a chapter that drifted. The second is the same rule applied to this chapter: a design that breaks a rule written here is wrong, and the design changes, not the rule. If a rule must change, it changes here, in the open, with the reason written down, as the introduction to this chapter says.

The manual has house rules too, and they apply to every chapter including this one. No em-dashes in the text. No invented numbers. No describing Riza as if it already exists before it is built. The first keeps the text readable. The second keeps the numbers honest. The third keeps the book honest about what is an idea, what is a plan, and what is real. A chapter that describes a design as built when it is only planned has broken the third rule, and it is fixed before it counts as done.

The checklist for all of this lives in Appendix E, so that any reader can check a chapter against the done definition without taking the book's word for it. That is the point of writing the definition down. A standard nobody can check is not a standard. It is a wish. This one can be checked, chapter by chapter, by anyone with the book and the time to read it.

Part 1 ends here. Chapter 1 named the waste. Chapter 2 named the idea. Chapter 3 named the rules. The foundations are laid: what is wrong, what the answer is, and what the build is held to. Part 2 builds the language, starting with Chapter 4, the dictionary, where the idea from section 2.1 becomes a real design: the most used operations get the shortest codes, and the dictionary of those codes is shared by every program.

---

The rules are three. Build for this machine and no other, and let its limits shape the design instead of stopping it. Spend nothing and need no network, so the system belongs to the person using it. And hold every chapter to the done test, so the book is as honest as the system it describes. The rest of the book is built inside those rules, one chapter at a time.
