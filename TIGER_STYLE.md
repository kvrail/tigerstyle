# Adapting TigerStyle to Your Codebase

TigerStyle is the coding style of the TigerBeetle database. It puts safety first, performance second and developer experience third, and every rule serves one of those goals. This guide restates the style for other codebases and says how far each rule travels.

Many of TigerStyle's mechanisms were built for systems programming, where the program manages memory and other resources explicitly. In other settings they often apply as goals but not as mechanisms. The rules are therefore sorted into three levels: baseline requirements, project-profile decisions and local conventions. Blocks that fit only some projects open with a line that says when they apply.

It is an independent rewrite of [TigerStyle](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md). The core rules are theirs. The wording, the three levels, the applicability lines, the examples, and the sections on security, adoption and language differences are additions. Several rules are deliberately looser than the original; where a number or a ban was relaxed, the text gives TigerStyle's own setting. Each rule is an imperative followed by its reason.

## In brief

The guide in ten lines:

- Safety comes first, then performance, then developer experience.
- A rule applies only where its subject exists, and the profile names the specialized blocks in force.
- Every resource has a limit, and every limit has a stated reaction.
- Input from outside is hostile until it is parsed at the boundary.
- Every error is handled, and the error paths are tested.
- Authority is explicit: deny by default, and check at the point of use.
- Internal invariants are asserted, and a failed assertion follows a written policy.
- Memory, recursion, concurrency and the numeric limits are decided once, in the project profile.
- Naming and layout follow the language first and this guide second.
- A change fixes what it touches and records what it does not.

## Scope and precedence

Two questions decide what a rule asks of a project. Does it apply here? If it does, how binding is it?

### Applicability

A rule applies where its subject exists: the input, resource, mechanism or requirement that it is about.

- **Apply the common core wherever its subject appears.** Most rules concern what nearly every program does: taking input, handling failure, bounding work and stating contracts. Each binds where that happens: the rule on outside data holds wherever such data enters the system.
- **Switch on a specialized block by its trigger.** Some rules presuppose a mechanism or a requirement, such as explicit memory management, a protocol of your own, elevated privileges or a strict latency budget. They bind only a project that has it. Each such block opens with a line that says when it applies, and a single conditional rule names its condition itself.
- **Do not record inapplicability as an exception.** A rule whose subject is absent is out of scope and needs no reason and no owner. Otherwise an ordinary project would have to justify not being an infrastructure system.

### Levels

The rules that apply do not bind equally. Each section is marked with one of three levels, and a higher level wins.

| Level | What it covers | Who decides | Sections |
| --- | --- | --- | --- |
| Baseline requirement | Checking untrusted input, handling errors, bounding resources, controlling authority | Nobody: where it applies, it holds in every codebase and language | Limits and errors; Security |
| Project profile | Memory strategy, recursion, concurrency model, assertion policy, performance design, dependencies | The project, once, in writing | Control flow; Assertions; State, memory and numbers; Performance; Dependencies and tooling |
| Local convention | Naming, line length, function size, argument order, comment and commit form | The language's ecosystem first, then the project | Functions; Naming; Comments, commits and layout |

- **Read a section's level as the default.** A few rules belong to another level than their section. Each of those carries its own label, and that label wins.
- **Let the higher level win.** A baseline requirement beats a profile decision, and a profile decision beats a convention. If a function can stay under its line limit only by dropping an error check, keep the check.
- **Let the language win over a convention.** Where the ecosystem has a standard formatter, naming scheme or argument order, follow it. A convention exists for consistency, and the ecosystem is the larger body of code to be consistent with.
- **Let the language shape the profile, not the baseline.** A garbage-collected runtime changes how you bound memory. It does not change whether you bound it.
- **Keep the goal when you change the mechanism.** When a rule's mechanism does not fit, replace it with one that serves the same goal, and record the choice in the profile.
- **Treat TigerStyle's settings as one example of a profile.** Its figures and bans were chosen for a database. A different value in your profile is not in itself a lower standard. Judge each setting against what the project requires.
- **Enforce the limits you set.** Once the profile sets a limit, tooling enforces it, and it is not reopened case by case. Assertion density is the one exception (*Read assertion density as a signal, not a quota*).
- **Record every exception.** Write its scope, its reason and its owner in the profile. Mirror it in the automated check as a suppression of exactly that scope, and point to the profile entry from the code. A baseline requirement is never waived, only met by a different mechanism.

## Principles

Three goals decide every question of style, in this order: safety, performance, developer experience.

- **Rank the goals.** All three matter. When they pull apart, safety beats performance and performance beats developer experience.
- **Treat style as design.** Judge every choice by whether it adds to or takes from the three goals. Readability is the entry fee, not the aim.
- **Work for simplicity.** The simple design is rarely the first draft. It is usually the one idea that serves all three goals at once. Keep sketching until you find it, and be ready to discard a draft.
- **Think before you build.** Design time is the cheapest time in a project. Implementation, testing and years of operation each cost far more, so spend the thought before any code exists.
- **Add no technical debt.** A change ships without known defects of its own. Fix what you introduce and what you touch before it merges, because it will never be cheaper than now.
- **Record what is out of scope.** A problem you find outside the current change gets a tracked issue, not a rewrite in passing. Keep each change small and about one thing. This binds AI agents too: no unrelated refactoring inside a small task.
- **Let no showstopper through.** A known latency cliff, unbounded growth or an exponential algorithm does not ship in the code you are changing, however unlikely the trigger.
- **Ship less, but solid.** A missing feature is acceptable; a feature that misses the goals is not. Revisiting a decision after you learn more is normal. Shipping what you already know to be broken is not.
- **Prefer small.** Less code, fewer abstractions and fewer moving parts leave less to get wrong.
- **Give reasons.** State the reason with every rule, decision and surprising line. A reason lets the reader judge the choice and tell when it has stopped applying.

## Limits and errors

**Level:** baseline requirement

Every resource has a bound and every failure has a handler, in any language.

### Limits

- **Bound everything.** Every loop, queue, buffer, batch, retry and wait gets a fixed upper limit. Real systems are finite, so state where the edge is.
- **Say what happens at the limit.** Choose a reaction for each limit: reject, apply backpressure, shed load or stop. Then fail fast when the limit is crossed, because a limit with no stated reaction is only a hope.
- **Mark the loops that never end.** An event loop is the one exception. Follow it with an assertion that the code after the loop is unreachable.

### Errors and tests

- **Handle every error.** No ignored return codes and no empty catch blocks. A [study of production failures](https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-yuan.pdf) in distributed systems traced 92% of the catastrophic ones to non-fatal errors that were handled wrongly.
- **Test the error paths.** Error handling is the least-exercised code in most systems. In that study, simple tests of error-handling code would have exposed the fault behind 58% of the catastrophic failures.
- **Test both sides.** Give tests valid data, invalid data, and data that crosses from valid to invalid.
- **Describe each test.** Open a test with a short note on its goal and its method, so a reader can follow it or skip it. **Level:** local convention

## Security

**Level:** baseline requirement

Security is safety with an adversary. Safety asks what happens when input is wrong by accident; security asks what happens when someone chooses it. The other rules already remove whole classes of attack, and this section adds what an adversary changes.

### Threat model

- **Write the threat model before the code.** One page is enough. List what you protect, who might attack and how, where trust changes hands, and what you choose not to defend. It does for security what the paper sketch does for performance.
- **Assume the attacker has read the source.** Security rests on keys and checks, never on the design staying secret.
- **Reopen the model when the surface changes.** A new input, endpoint, privilege or dependency changes the threat model, and is reviewed as such.

### Trust boundaries

- **Treat everything that crosses a boundary as hostile.** That covers the network, files, environment variables, other processes and tenants, and your own storage if anyone else can write to it.
- **Parse at the boundary.** Convert untrusted bytes, in one place, into typed values that cannot represent an invalid state ([Parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)). Code inside the boundary accepts only those types and never sees raw input.
- **Allow what is valid and reject the rest.** Define the accepted set, and refuse everything outside it. Finish validating a whole message before any part of it takes effect.
- **Accept one encoding per value.** Reject non-canonical forms. Two components that read the same bytes differently give an attacker a gap to work in.
- **Return errors at the boundary; assert behind it.** Bad input from outside is an operating error, expected and handled. An assertion that outside input can trip is a remote off switch.
- **Never mix data into commands.** Do not build SQL, shell commands, HTML, file paths, format strings or log lines by joining strings with untrusted data. Use parameterized interfaces, encode output for the place it lands, and never decode untrusted data into arbitrary object types.

### Limits as defense

- **Bound before you allocate.** Check every claimed length, count, nesting depth and decompressed size against a maximum before committing memory to it. A length field is a claim, not a fact.
- **Bound time.** Every wait has a timeout, every retry has a cap, and every request has a deadline that covers all of its steps.
- **Give every client a budget.** Set a quota on every shared resource, so that one client cannot starve the rest. When over capacity, refuse early and cheaply.
- **Spend less than the attacker.** Run cheap checks, such as size, format and quota, before expensive ones.
- **Do not amplify to an unverified address.** Some transports, such as UDP, let a sender forge its address. On those, send back no more than a small multiple of what arrived until the peer has shown that it receives at that address. A large reply to a small request is otherwise fine, with or without user authentication.
- **Plan for the worst case.** When an attacker picks the input, average-case speed is irrelevant. Use keyed hashing for hash tables whose keys come from outside, pattern matching that runs in linear time, and parsers with no quadratic paths.

### Memory and arithmetic

- **Prefer a memory-safe language.** Where you cannot, confine unsafe code to small, marked modules. Fuzz those under sanitizers, and ship with every toolchain mitigation and bounds check turned on.
- **Choose overflow behavior by name.** Arithmetic is checked unless the code says wrapping or saturating. Check size calculations, such as a count times a unit, before they reach an allocator or a copy.
- **Never hand out stale memory.** Initialize what you allocate, and zero the unused part of a buffer before it crosses a boundary.

### Secrets

**Applies when:** the program handles keys, passwords, tokens or other secrets.

- **Keep secrets out of source, logs, error messages, URLs and metrics.** Wrap each secret in a type that prints a placeholder, so that exposing it takes a deliberate call. This stops accidental output. It does not remove the bytes from memory.
- **Set a policy for memory dumps.** A core dump or heap snapshot holds every secret the process has loaded. Decide whether dumps are disabled, restricted or scrubbed, and handle any dump as secret material.
- **Keep few copies, for a short time.** Load a secret as late as possible, pass it by reference and drop it when the operation ends. Every copy and every second adds exposure.
- **Know the limits of wiping.** Where you control memory, erase a secret with a zeroing call the compiler will not remove. In a garbage-collected runtime, immutable strings and relocated objects leave copies you cannot reach, so use mutable byte buffers and short lifetimes.
- **Compare secrets in constant time.** A comparison that stops at the first difference tells an attacker how much of a guess was right.
- **Make every secret narrow, short-lived and replaceable.** Use one key for one purpose. Build rotation in from the start and practice it, because the leak of a secret you cannot rotate has no remedy.
- **Rotate what leaks.** A secret that reached a repository, a log or a chat is compromised. Deleting it from history does not undo that.

### Cryptography

**Applies when:** the code chooses, configures or calls cryptography. With a ready-made secure transport, the rules on protocol design bind that transport; the application only configures it.

- **Do not invent cryptography.** Use standard protocols and reviewed, maintained implementations, through their highest-level interface: "seal this message", not "run this block cipher".
- **Restrict the algorithms you accept.** Configure a short list of current algorithms and refuse the rest. Never fall back to a weaker option when negotiation fails.
- **Protect what is negotiated.** The handshake must authenticate the negotiated parameters, so that an attacker in the middle cannot force a weaker choice.
- **Fix one suite per version in a protocol of your own.** One suite per version removes algorithm negotiation and the complexity that comes with it. The version can still be downgraded, so authenticate the choice of version and refuse versions that are no longer allowed.
- **Authenticate before you interpret.** Use authenticated encryption, and verify a signature or tag before parsing or acting on the data it covers.
- **Take randomness only from the operating system's secure generator.** Inject it as a dependency, so that tests stay deterministic and production cannot fall back to a weak source.
- **Make reuse impossible by construction.** A nonce never repeats under a key. A key, hash or signature made for one purpose is rejected for any other, so bind the purpose, version and context into what is signed.

### Authority

- **Deny by default and fail closed.** Access needs an explicit grant. If a check errors, times out or cannot decide, the answer is no.
- **Authorize at the point of use.** Authenticate once at the boundary, then check on every operation that this principal may do this action to this object. Never rely on a check that the client performed.
- **Check and use the same object, atomically.** Open the file and inspect the handle, rather than inspecting a path and opening it later. Any gap between check and use belongs to the attacker.
- **Acquire at startup, then drop privileges.** Where the program starts with rights it will not need later, open the sockets, files and keys it needs. Then give up the right to acquire more. This is *Allocate at startup, then stop*, applied to authority. **Level:** project profile
- **Pass authority explicitly.** A function receives the handle it may use as an argument, and has no ambient access to the file system, network or credentials. Its signature then states what it can touch.
- **Keep parsers away from keys.** The component that handles untrusted input holds no secrets and few privileges. The one that holds keys parses as little as possible.
- **Shrink the surface.** Every endpoint, flag, feature and debug hook is attack surface. Remove what is unused, make the default configuration the safe one, and give unsafe options names that say so.

### Errors, logs and data

- **Tell outsiders little and operators enough.** Errors returned across a boundary are uniform and reveal no internals. The content, size and timing of a response should not show which check failed.
- **Log security events.** Record who did what, when and with what outcome: sign-ins, denials, privilege changes, key use. Keep logs append-only, bounded, and free of secrets and personal data.
- **Hold less data, for less time.** What you never collect cannot leak. Data lifetime gets an upper bound like everything else.

### Supply chain

- **Treat a dependency as a committer.** Its authors, and the authors of everything it pulls in, run code with your privileges. This is the security case for *Aim for zero dependencies*.
- **Pin every dependency by hash.** Commit the lockfile, vendor where practical, and build without fetching anything that is not pinned.
- **Update on purpose.** Upgrade one dependency at a time, read what changed, and subscribe to vulnerability alerts for all of them.
- **Lock down the build.** Builds are reproducible from pinned tools. Untrusted pull requests get no secrets, and release-signing keys stay off machines that run untrusted code.
- **Require a second reviewer.** A second person reads every change before it merges, so that one mistaken or stolen account cannot alter the code alone. **Level:** project profile

### Verification and response

- **Attack your own boundaries in tests.** Feed each one malformed, oversized and truncated input. Assert that it is refused and that no unauthorized change reaches application state.
- **Specify duplicates, reordering and replays.** The protocol decides what happens to each: reject, deduplicate, reorder or process idempotently. Test that behavior. Audit records and rate-limit counters may still change, and the tests should allow for it.
- **Fuzz every parser, continuously.** Keep each crashing input as a permanent regression test.
- **Simulate a malicious peer, not only a faulty one.** Random faults find accidents. An adversary picks the worst moment on purpose.
- **Assert only what the execution model keeps stable.** Access rights can change between two checks, so a second denial is an ordinary refusal, not a broken invariant. Recheck access with *Authorize at the point of use*, and keep assertions for internal invariants.
- **Carry no security debt.** A known vulnerability is a showstopper and is fixed when found. Then fix the class: add the type, assertion, lint or fuzz target that stops its siblings.
- **Be ready before the first report.** Publish how to report a vulnerability in private, and rehearse shipping a fix.

## Control flow

**Level:** project profile

Control flow should be plain enough that a reader can see every path and every bound.

- **Keep control flow simple and explicit.** Use plain branches and loops that a reader can trace by eye. Clever dispatch and hidden jumps create paths that nobody checks.
- **Ban recursion, or bound it.** TigerStyle bans recursion: a loop with a stated bound shows that execution ends and that the stack stays flat. Where recursion is the idiom, recurse only over data with a known bound, or carry an explicit depth limit. Check the limit before each further call, and refuse input that would exceed it.
- **Use few abstractions, and only excellent ones.** Every abstraction has a cost and can leak. Keep one only when it models the domain better than plain code would.
- **Set your own pace.** Do not act directly on each external event. Queue events and handle them in bounded batches on your own schedule. The program, not its callers, then decides how much work happens per tick.

## Assertions

**Level:** project profile

An assertion states an internal invariant that the programmer believes always holds. What a failed assertion does is a policy to choose and write down.

- **Know which kind of error you have.** A full disk or a dropped connection is an operating error: it will happen, so write the code that deals with it. A broken internal invariant is a programmer error: the state can no longer be trusted.
- **Decide what a failed assertion does.** The default is to stop the process, or at least the unit of work whose state is suspect. Write the policy down, because an assertion may raise a catchable exception, end only one task, or be compiled out.
- **Never continue on suspect state.** Whatever the policy, code must not catch an assertion failure and carry on with the data that failed it. Isolate everything the failed unit of work could have touched. Resume only from state that a verified recovery has rebuilt.
- **Make stopping safe.** Stopping limits the damage but does not undo writes already made. Keep stored data consistent after an abrupt stop, with atomic writes or a recovery path, and test the recovery.
- **Keep assertions on in production.** A failure caught at the invariant is cheaper than corrupt data found later. Check the build configuration, since many toolchains strip assertions from release builds.
- **Check before you compute.** Assert arguments on the way in, results on the way out, and invariants in between. Unchecked data should never reach the logic of a function.
- **Read assertion density as a signal, not a quota.** TigerStyle's figure is two assertions per function on average, and density is what makes fuzzing pay. A function far below the profile's figure earns a closer look in review; it does not fail the build. An assertion that cannot fail, or that repeats the line above it, adds noise and no safety.
- **Assert in pairs.** Check the same property in two places that reach it by different routes. Check a message where it is encoded and again where it is decoded. A mistake then has to slip past both ([It Takes Two to Contract](https://tigerbeetle.com/blog/2023-12-27-it-takes-two-to-contract)).
- **Assert both sides.** Check what you expect to happen, and check what must never happen. Bugs gather at the line where valid data turns invalid.
- **Put one condition in each assertion.** Write `assert(x > 0); assert(x < limit);`, not one assertion that joins the two with `and`. A failure then names the exact condition that broke.
- **Assert implications with a guarded line.** `if (is_final) assert(pending_count == 0);` says that the first implies the second.
- **Assert at compile time.** If the compiler can check a fact, check it there: how constants relate, how large a type is, how a structure is laid out. A design error then fails the build instead of a run.
- **Let an assertion replace a comment** now and then. Where a fact matters a great deal and a reader would not expect it, an assertion documents it and enforces it too.
- **Understand first, then assert.** An assertion records what you know; it cannot stand in for knowing it. Build the mental model, encode it as assertions, and defend it to your reviewer in code and comments.
- **Treat the fuzzer as a backstop.** Fuzzing and simulation come last, to find what you and your reviewer both missed. A fuzzer that finds nothing has not shown that nothing is there.

## State, memory and numbers

**Level:** project profile

Keep state small, single and close to where it is used. Most bugs live in the gap between where a value is checked and where it is used.

### Memory

- **Allocate at startup, then stop.** Where the program manages memory itself, size every buffer and pool from a stated maximum during initialization. Allocate or free nothing afterwards. This avoids use-after-free and allocation stalls. It also forces you to design memory use up front, which tends to give a simpler and faster system.
- **Where the runtime manages memory, budget it.** Set a memory and allocation budget for each hot path, measure it with a profiler, and treat a regression as a defect. Zero allocations is not a goal in itself.
- **Group acquire and release.** Put a blank line before the line that acquires a resource and after the line that schedules its release. The pair reads as one unit, and a missing release stands out.
- **Zero what you do not fill.** A partly used buffer still holds old bytes in its tail. Clear the padding, or stale data can leak and identical inputs can stop giving identical outputs.

### Variables

- **Use the smallest scope.** Declare a variable where it is first needed and let it end as soon as it is not. The fewer names in view, the fewer chances to use the wrong one.
- **Keep one copy.** A value lives in one place and is read from there. A second copy or an alias will drift from the first.
- **Check next to the use.** Compute or validate a value right before the code that depends on it. Distance between check and use, in lines or in time, is where assumptions go stale. Time-of-check to time-of-use bugs are the best-known case.
- **Pass large values by read-only reference.** When a value is large and no copy is intended, do not pass it by value. An accidental copy costs time and can hide a bug. TigerStyle's threshold is 16 bytes. Set yours in the profile, and follow the language where copying small values is the idiom.
- **Build large structures in place.** Initialize a big value directly in its final location, through a pointer to it, instead of building it elsewhere and moving it. Its address stays stable and the stack does not grow with hidden copies. If one field is built in place, build the whole container in place.

### Concurrency

**Applies when:** more than one task, thread or process can touch the same state.

- **Choose one concurrency model for each component.** Write down whether its state is owned by one thread, shared under locks or passed in messages. What can change between two lines of code depends on that choice.
- **Run to completion where the model allows it.** In single-threaded, cooperative code, a function that does not suspend keeps its preconditions true from start to end. If it yields, they may be false by the time it resumes.
- **Do not treat the absence of `await` as a lock.** Not suspending protects state only from tasks on the same thread. Other threads can still change whatever they share.
- **Check and apply in one atomic step.** After asynchronous work, validate the state you depend on and make the change inside the same critical section, transaction or compare-and-swap. A second check on its own still leaves a gap before the write.
- **Keep critical sections short.** Do slow and asynchronous work outside the lock, then take the lock only to check and apply.
- **Never hold an ordinary blocking lock across a suspension.** It can deadlock, or stall every task on the thread. Hold a lock across `await` only when the lock is built for that and the design needs it.
- **Treat an expected concurrent change as a conflict.** Refuse, report or retry. An assertion is for an invariant the model guarantees, not for a race the design allows.

### Numbers

- **Know which kind of number you have.** Fixed-width integers overflow, arbitrary-precision integers grow without bound, and floating-point numbers lose precision. Each needs its own check.
- **Use explicitly sized integers where the language has them.** Choose the width on purpose, such as `u32` or `i64`, and avoid types whose size depends on the machine.
- **Treat index, count and size as three types.** For a zero-based sequence, a count is at least zero and a valid index satisfies `0 <= index < count`. `index + 1` is the number of elements up to and including that index. `count - 1` is the last index only when the sequence is not empty.
- **Compute sizes with an overflow check.** A size is a count multiplied by a unit, and the multiplication can overflow. Name variables so the reader can tell an index from a count from a size.
- **Say how division rounds.** Use an operation that names exact, floor or ceiling division. The reader then knows you thought about the remainder.

## Performance

**Level:** project profile

**Applies when:** throughput or latency is a stated requirement. Without one, *Let no showstopper through* still holds.

Performance is decided in the design, before there is anything to measure.

- **Design for performance from the start.** The order-of-magnitude wins come from the architecture, which is hardest to change once code exists. Profiling later can only polish what the design allows.
- **Sketch on paper first.** For each candidate design, estimate bandwidth and latency for the network, the disk, memory and the CPU. A rough sketch is cheap, and it is usually enough to choose the right design.
- **Work with the machine.** Know how the hardware behaves and design to suit it: sequential access, data that stays in cache, I/O in batches.
- **Optimize the slowest resource first.** The usual order is network, disk, memory, CPU. Then weight each by how often it is used. A memory access in a hot loop can cost more in total than one network call per request.
- **Decide per batch, not per item.** Keep the control plane, which decides, apart from the data plane, which does the work. The control path can then afford heavy checking, and the data path stays tight.
- **Batch to amortize.** Pay a fixed cost, such as a system call, a network round trip or a disk flush, once per batch instead of once per item.
- **Keep the CPU on a straight line.** Predictable code is fast code. Give the processor long runs of similar work instead of making it hop between unrelated tasks.
- **Do not count on the optimizer.** Write the fast path explicitly. A hot loop belongs in its own function whose arguments are plain values, not an object with fields. Nothing then depends on the compiler proving that a field cannot change mid-loop, and duplicated work is easy to see.

## Dependencies and tooling

**Level:** project profile

Every dependency and every tool is a cost that you pay for as long as the project lives.

- **Aim for zero dependencies.** Beyond the language toolchain, each dependency brings supply-chain risk, bugs you cannot fix, performance you do not control and a slower install. In foundational software, everything built on top inherits those costs.
- **Own the exceptions.** When a dependency is unavoidable, record why, pin it, and read it as carefully as code you wrote yourself.
- **Keep the toolbox small.** Every extra tool has to be installed, learned, upgraded and debugged by everyone. Reach first for the tool the team already uses, even when a specialized one would do the job slightly better.
- **Script in the project's main language.** Write build and release scripts in the language of the codebase. They are then checked by the same compiler, run wherever the code runs, and can be maintained by anyone on the team. A shell script fails on the one machine whose shell differs.
- **Standardize early.** Each new teammate brings preferred tools, and the variety compounds. Agreeing on one costs an individual some speed now and saves the team far more later.
- **Turn on every compiler warning at the start.** Use the strictest level the compiler has and fail the build on any warning. A warning tolerated early becomes noise that hides the one that matters.

## Functions

**Level:** local convention

The function is the unit of review: keep it short, give it a narrow interface and make its contract visible.

### Shape

- **Limit every function to 70 lines.** That is about one screen, and a function you can see whole is one you can check whole. The number is TigerStyle's setting (*Treat TigerStyle's settings as one example of a profile*). Finding the split that reads well is the craft.
- **Keep the ends narrow and the middle wide.** A good function takes few parameters, returns something simple and does real work in between.
- **Branch in the parent, loop in the helper.** When a function grows too long, split it so that every decision stays in one place. The parent chooses what to do; the helpers do it without branching ([Push Ifs Up And Fors Down](https://matklad.github.io/2023/11/15/push-ifs-up-and-fors-down.html)).
- **Change state in one place.** Helpers calculate and return. Only the parent applies the result to its own local state, so the functions at the leaves have no side effects.
- **Return the smallest type that states the whole contract.** Every outcome a function can return becomes a branch in each caller, so return no more than the operation can produce.
- **Keep the outcomes that callers must tell apart.** Do not merge distinct failures, or hide an error, to make a signature look simpler. A caller can handle only what the contract tells it. **Level:** baseline requirement

### Call sites

- **Name arguments that could be swapped.** A function that takes two parameters of the same type gets named arguments or an options struct: `resize({ width, height })`, not `resize(width, height)`.
- **Make an absent argument readable.** If an argument may be null, the call site must show by name what is being left out.
- **Pass long-lived dependencies by position.** An allocator, a clock or a tracer each has a type of its own and cannot be mistaken for another. Hand them to constructors positionally, in a fixed order, general before specific.
- **Put callbacks last.** They run last, so they read last.
- **Spell out the library options that matter.** When a default affects correctness, safety, durability or resource use, write the option in the call, or in the one wrapper the codebase uses. An upgrade that changes the default cannot then change your behavior behind your back. Restating every cosmetic default only adds noise.

### Conditions

- **Split a compound condition when its cases differ.** If the parts need different handling, or the whole is hard to verify, nest simple `if`/`else` branches so that each case is visible. Unroll long `else if` chains the same way. A short condition that expresses one idea, or a named predicate, can stay whole.
- **Account for the other branch of every `if`.** Decide what the `else` means: handle it, assert that it cannot happen, or leave it out when a guard clause already says everything. The goal is that no case goes unconsidered, not that every `if` grows an `else`.
- **State conditions in the positive.** Test for the state in which things are right, such as `if (length <= capacity)`, and put the failure in the `else`. Negated tests are easy to get wrong by one.

## Naming

**Level:** local convention

Names are the main interface to the code, so spend real time on them.

- **Choose nouns and verbs with care.** A good name tells the reader what something is, or what it does, without a comment. Keep looking until the names in a module fit one another as well as they fit the things they name.
- **Use `snake_case` for functions, variables and files** where the language leaves the choice open. Words set apart by underscores read almost like prose, so nobody is tempted to shorten a name to keep it legible.
- **Do not abbreviate.** Write `connection_count`, not `conn_cnt`. A single letter is acceptable only for a plain integer in a sort or a matrix calculation.
- **Spell out flags in scripts.** `--verbose` explains itself. `-v` is for typing at a prompt.
- **Capitalize acronyms as acronyms.** `HTTPServer`, not `HttpServer`.
- **Put units and qualifiers last, most significant word first.** `retry_delay_ms_max` and `retry_delay_ms_min` sort together and line up, and everything about the retry delay sits in one group.
- **Let a name carry its obligations.** `scratch_buffer` tells the reader that its contents do not survive the call. `buffer` tells them nothing.
- **Give related names the same length.** Choose `lower` and `upper`, or `client` and `server`. Derived names such as `client_address` and `server_address` then align, and an asymmetry in the code stands out.
- **Prefix a helper with its caller's name.** A helper or callback that serves one function takes that function's name as a prefix: `flush_batch()` and `flush_batch_callback()`. A reader can then tell from the name alone who calls it.
- **Give each name one meaning.** If `node` already means a cluster member, a tree element needs another word. A term that changes meaning with context will be misread.
- **Pick names that work outside the code.** Names end up in documents, tickets and conversations. A noun such as `queue.backlog` can be used there unchanged, and extends naturally to `backlog_max`. An adjective or participle such as `queue.waiting` has to be rephrased every time.
- **Put first things first.** People read a file from the top down, so the important things go there, and `main` goes first of all. Inside a type, put fields first, then nested types, then methods.
- **Flatten and sort where it helps.** Move a complex nested type to the top level. Where no order is natural, sort alphabetically; names that start with their most significant word then group themselves.
- **Follow the language's standard style guide for everything else.**

## Comments, commits and layout

**Level:** local convention

Write for the reader who arrives later with no context.

### Comments

- **Explain why.** Code shows what happens. Only a comment can say why it was written this way and what was ruled out.
- **Write comments as sentences.** Put a space after the comment marker, capitalize the first word and end with a full stop. End with a colon when the comment introduces the lines below it.
- **Keep trailing comments short.** A comment that trails a line of code may be a phrase without punctuation.

### Commits

- **Write commit messages worth reading.** Say what changed and why, for the person who runs blame on this line years from now.
- **Do not lean on the pull request description.** It lives with the hosting service, not in the repository, and blame will never show it.

### Layout

- **Run the formatter.** Use the language's standard formatter on every file and do not argue with its output.
- **Indent by 4 spaces** where the formatter leaves the choice open. The structure is easier to see from a distance than with 2.
- **Limit every line to 100 columns.** Two files then fit side by side on one screen, and nothing hides behind a horizontal scrollbar. The number is TigerStyle's setting (*Treat TigerStyle's settings as one example of a profile*). Set a ruler in your editor and use the width before you wrap.
- **Let the formatter wrap.** Add a trailing comma to a long signature, call or literal, and the formatter will lay it out one item per line.
- **Brace every `if` that spans lines.** Only an `if` that fits on one line may go without braces. This guards against the bug where a second statement looks conditional and is not.

## Adapting the rules to your language

Each profile rule names a goal and TigerStyle's mechanism for it. Where your language differs, keep the goal and use the mechanism below.

| Rule | Goal | Where the language differs |
| --- | --- | --- |
| Allocate at startup | Bounded, predictable memory | In a garbage-collected runtime, apply *Where the runtime manages memory, budget it*: size collections from stated maximums, and reuse buffers where a profiler shows that it matters. |
| Explicitly sized integers | No surprises from arithmetic | Fixed width (C, Go, Rust, Zig): use explicit widths, and convert platform-sized types such as `int` or `usize` with a checked cast. Arbitrary precision (Python `int`, JavaScript `BigInt`): check the domain range and cap the size, because a huge value costs time and memory. Floating point (JavaScript `Number` is IEEE 754 binary64): for an integer field require `Number.isSafeInteger` and a range check, and use `BigInt` for larger exact values. |
| Assertions always on | Failure at the invariant, in production | C `assert` disappears under `NDEBUG`, Python `assert` under `-O`, and Rust `debug_assert!` in default release builds. Java `assert` is off unless enabled. Use a check that cannot be compiled out, keep it free of side effects, and confirm what a failure does: in several languages it raises an exception that code can catch. |
| Compile-time assertions | Design errors fail the build | `static_assert` in C and C++, a `const` assertion in Rust, `comptime` in Zig. Without such a feature, put the checks in a test that runs on every build. |
| No recursion | Bounded execution | Where recursion is the idiom, as in functional languages, use the bounded form in *Ban recursion, or bound it*. |
| Named arguments | Call sites that cannot be mixed up | Keyword arguments (Python, Swift, Kotlin), an options struct or object (C, Go, Rust, Zig, TypeScript), or a distinct wrapper type for each kind of value. |
| Pass by reference, build in place | No accidental copies | These apply to languages with value semantics, and the size threshold is a profile decision. With reference semantics the risk is the opposite one, accidental sharing: copy on purpose and do not alias mutable objects. |
| Run to completion | Preconditions hold throughout | The guarantee covers one turn of one thread. With `async`/`await`, coroutines or threads, the rules under Concurrency take over, starting with *Check and apply in one atomic step*. |
| Handle every error | No silent failure | With exceptions: no empty `catch`, and catch only what you can handle. With return codes: make an ignored result a compiler or linter error (`[[nodiscard]]`, `#[must_use]`, `errcheck`). |
| Rounding and overflow | Intent visible in arithmetic | Defaults differ between languages and even between build modes. Name the rounding and the overflow behavior wherever the language offers a choice. |
| Formatting and naming case | One look for all code | Where the ecosystem fixes a convention (`gofmt` tabs, Go's capitalized exports, Java's camelCase), it wins. The point is consistency and descriptive names. |

## Adopting the guide

Adopt the guide in six steps, and keep the result as a short profile file in the repository. The first step is required, because the guide is not meant to be applied whole.

1. **Select what applies.** List the specialized blocks and conditional rules that are in force for this project (*Switch on a specialized block by its trigger*). Do this before you choose any mechanism or number. A specialized block is out of scope only when its stated applicability condition is not met. Leaving it out of the profile does not waive an applicable rule.
2. **Choose the project profile.** Decide the mechanisms for what applies: the memory strategy, the recursion policy, the concurrency model and the assertion policy. Write each one down with its reason.
3. **Define the resource limits.** List every bounded resource with its limit (*Bound everything*).
4. **Set the reaction to each limit.** State what happens when it is exceeded (*Say what happens at the limit*).
5. **Automate the checks.** Put the formatter, the lint rules, the length limits and the tests for each limit into the build. A rule that depends on review alone will drift.
6. **Record exceptions and open questions.** Use the procedure in *Record every exception*, and keep open questions in the profile beside the exceptions.

For each limit, state the unit, the scope, where it is checked and what happens when it is exceeded. The values below are examples.

| Limit | Value and unit | Scope | Checked | On excess |
| --- | --- | --- | --- | --- |
| Request body | 1 MiB | Per request | At the parser, before allocation | Reject with an error |
| Queue depth | 1,024 items | Per connection | On enqueue | Apply backpressure |
| Retries | 3 attempts | Per operation | In the retry loop | Return the last error |
| Function length | 70 lines | Per function | By the linter, in the build | Fail the build |

When the guide instructs an AI agent, give it the profile as well. The agent then knows which rules are in force in this repository, and does not carry the whole systems-programming profile into every task.

## Examples

Five short cases in pseudocode show how the rules read in practice.

### Trust boundary

Untrusted bytes are parsed once, field by field, into types that cannot hold an invalid value. `parse_account_id` has the same shape as `parse_amount`.

```
// Contract: returns an Amount, an integer with 1 <= amount <= amount_max, or an error.
function parse_amount(raw): Amount | ParseError {
    if (!is_integer(raw)) return ParseError.amount_not_integer;
    if (raw < 1) return ParseError.amount_too_small;
    if (raw > amount_max) return ParseError.amount_too_large;
    return Amount(raw);
}

// At the boundary: bad input is an operating error, so it is returned, not asserted.
function parse_transfer(bytes): Transfer | ParseError {
    if (bytes.length > transfer_bytes_max) return ParseError.too_large;
    const fields = decode(bytes);
    if (fields == null) return ParseError.malformed;
    const account_id = parse_account_id(fields.account_id);
    if (is_error(account_id)) return account_id;
    const amount = parse_amount(fields.amount);
    if (is_error(amount)) return amount;
    return Transfer { account_id, amount };
}

// Behind the boundary: only parsed types are accepted. A bad value here is a bug.
function apply_transfer(transfer: Transfer) {
    assert(transfer.amount >= 1);
    assert(transfer.amount <= amount_max);
    // ...
}
```

### Assertions

An expected failure is handled. An internal invariant is asserted, one condition at a time, where the value is written and again where it is read.

```
function append(log: Log, record: Record): Ok | Error {
    // Operating error: a full log will happen, so the caller gets an error.
    if (log.count == log.count_max) return Error.log_full;

    // Programmer error: callers must pass a record that is already checked.
    assert(record.size > 0);
    assert(record.size <= record_size_max);

    log.records[log.count] = record;
    log.count += 1;
    return Ok;
}

function read(log: Log, index: u32): Record {
    assert(index < log.count);  // The index is unsigned, so this one bound is enough.
    const record = log.records[index];

    // The same two properties, asserted again where the record is read.
    assert(record.size > 0);
    assert(record.size <= record_size_max);
    return record;
}
```

### State around `await`

A check made before a suspension is stale after it. Do the slow work first, then check and apply in one atomic step.

```
// Wrong: the balance can change while the limit is fetched.
async function withdraw(account, amount: Amount) {
    if (account.balance < amount) return Error.insufficient_funds;
    const limit = await fetch_withdrawal_limit(account.id);  // Other tasks run here.
    if (amount > limit) return Error.over_limit;
    account.balance -= amount;                               // The balance check is stale.
    return Ok;
}

// Right: nothing suspends between the balance check and the write.
async function withdraw(account, amount: Amount) {
    // Policy: a per-withdrawal limit read at the start holds for this one operation.
    const limit = await fetch_withdrawal_limit(account.id);
    if (amount > limit) return Error.over_limit;
    return account.update(function (state) {
        if (state.balance < amount) return Error.insufficient_funds;
        state.balance -= amount;
        return Ok;
    });
}
```

`account.update` stands for the atomic step of the chosen model: a short critical section, a transaction or a single-threaded turn. A concurrent withdrawal then surfaces as an ordinary refusal, not as a failed assertion. A daily limit would need more: the amount already spent would be checked and updated inside the same atomic step.

### Return types

The type states every outcome the caller must handle, and no more.

```
// Hides an error: null means both "no such user" and "the store is down".
function find_user(id): User | null

// States the whole contract: three outcomes, each handled by the caller.
function find_user(id): User | NotFound | StoreError

// Cannot fail and always has an answer, so the type stays small.
function user_count_max(): u32
```

### Repeated requests

The protocol decides what a duplicate means. Here every attempt is audited, a repeat returns the first result, and the transfer is applied once. The same key with different content is a conflict.

```
// The idempotency key is scoped to one client: (client_id, request.id).
function handle_transfer(client_id, request) {
    audit_log.append(client_id, request);  // Every attempt is recorded, duplicates too.

    // One atomic step: look up the key, apply the transfer, remember the result.
    return store.transaction(function (tx) {
        const key = { client_id, request_id: request.id };
        const earlier = tx.results.get(key);
        if (earlier == null) {
            const result = tx.apply_transfer(request.transfer);
            tx.results.put(key, { transfer: request.transfer, result });
            return result;
        } else {
            if (earlier.transfer != request.transfer) return Error.key_conflict;
            return earlier.result;  // A duplicate is answered, not applied again.
        }
    });
}

test "a replayed transfer is applied once and audited twice" {
    const first = handle_transfer(client_id, request);
    const second = handle_transfer(client_id, request);

    assert(second == first);
    assert(balance(request.transfer.account_id) == balance_before - request.transfer.amount);
    assert(audit_log.count == audit_count_before + 2);
}
```

## Credits and sources

The style and most of its rules come from the TigerBeetle team. This guide is an independent adaptation and is not endorsed by them.

| Source | Used here for |
| --- | --- |
| [TigerStyle](https://github.com/tigerbeetle/tigerbeetle/blob/main/docs/TIGER_STYLE.md), TigerBeetle | The original guide: the three goals, zero technical debt, and most of the rules at the profile and convention levels. |
| [The Power of Ten – Rules for Developing Safety Critical Code](https://spinroot.com/gerard/pdf/P10.pdf), Gerard J. Holzmann, NASA/JPL | The roots of the safety rules: bounded loops, no recursion, no allocation after initialization, short functions, two assertions per function, smallest scope, strictest warnings. |
| [Simple Testing Can Prevent Most Critical Failures](https://www.usenix.org/system/files/conference/osdi14/osdi14-paper-yuan.pdf), Yuan et al., OSDI 2014 | The evidence behind handling every error and testing the error paths. |
| [Push Ifs Up And Fors Down](https://matklad.github.io/2023/11/15/push-ifs-up-and-fors-down.html), matklad, 2023 | Keeping every branch in the parent function. |
| [It Takes Two to Contract](https://tigerbeetle.com/blog/2023-12-27-it-takes-two-to-contract), matklad, 2023 | Pair assertions. |
| [Parse, don't validate](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/), Alexis King, 2019 | Parsing at the trust boundary. |
| [The Protection of Information in Computer Systems](https://web.mit.edu/Saltzer/www/publications/protection/), Saltzer and Schroeder, 1975 | Fail-safe defaults, least privilege, complete mediation and open design, which the rules on authority restate. |
