---
layout: default
title: "Observing Claude Opus 4.6 Converge on Browser Exploit Techniques"
---

[Home](../index.md)

# Observing Claude Opus 4.6 Converge on Browser Exploit Techniques Across Independent Research

*Nathaniel Oh - March 17, 2026*

---

## Background

Last week, Anthropic published a [case study](https://red.anthropic.com/2026/exploit/) describing how Claude Opus 4.6 wrote a working exploit for CVE-2026-2796 in Firefox's SpiderMonkey engine. The blog walks through how the model decomposed the bug into classical exploit primitives and chained them into code execution against a stripped js shell. It's a landmark result - the first time Anthropic has observed a model writing a successful browser exploit with minimal hand-holding.

I use Claude Opus 4.6 through Claude Code as my primary tool for vulnerability research. In the past year, I have found several CVEs with LLM-assisted vulnerability research: 

| CVE | Vendor | CWE | Reference | Writeup |
|---|---|---|---|---|
| **CVE-2026-28890** | Apple | TBD | TBD | - | 
| **CVE-2026-28857** | Apple | TBD | TBD | - | 
| **CVE-2026-27820** | Ruby | **CWE-122** (Heap-based Buffer Overflow) | [ruby-lang.org](https://www.ruby-lang.org/en/news/2026/03/05/buffer-overflow-zlib-cve-2026-27820/) | - |
| **CVE-2026-20652** | Apple | **CWE-191** (Integer Underflow) | [126354](https://support.apple.com/en-us/126354) | - |
| **CVE-2025-43505** | Apple | **CWE-787** (Out-of-Bounds Write / Heap Corruption) | [125641](https://support.apple.com/en-us/125641) | - |
| **CVE-2025-43504** | Apple | **CWE-121** (Stack-based Buffer Overflow) | [125641](https://support.apple.com/en-us/125641) | [True](https://objective-see.org/blog/blog_0x83.html) |
| **CVE-2025-43375** | Apple | **CWE-20** (Improper Input Validation) | [125117](https://support.apple.com/en-us/125117) | — |
| **CVE-2025-43370** | Apple | **CWE-20** (Improper Input Validation) | [125117](https://support.apple.com/en-us/125117) | — |
| **CVE-2025-43299** | Apple | **CWE-20** (Improper Input Validation) | [125109](https://support.apple.com/en-us/125109), [125110](https://support.apple.com/en-us/125110), [125111](https://support.apple.com/en-us/125111), [125112](https://support.apple.com/en-us/125112) | — |
| **CVE-2025-43295** | Apple | **CWE-20** (Improper Input Validation) | [125109](https://support.apple.com/en-us/125109), [125110](https://support.apple.com/en-us/125110), [125111](https://support.apple.com/en-us/125111), [125112](https://support.apple.com/en-us/125112) | — |
| **CVE-2025-43353** | Apple | **CWE-787** (Out-of-Bounds Write / Heap Corruption) | [125110](https://support.apple.com/en-us/125110), [125111](https://support.apple.com/en-us/125111), [125112](https://support.apple.com/en-us/125112) | [True](https://calysteon.github.io/cve/CVE-2025-43353.html) |
| **CVE-2025-53623** | Shopify | **CWE-78** (OS Command Injection) | — | — |

Through that work, I have found that Claude appears to uniformly employ the same exploitation techniques across web browser families. In particular, I have found Claude to use similar techniques when building a exploits through WebAssembly.

When I read Anthropic's write-up, I recognized the exploit immediately. Not the vulnerability - completely different research, a completely different target. I recognized the *techniques*. The exploitation methodology that Claude produced autonomously for Anthropic's evaluation matched, almost step for step, the methodology that Claude had constructed in my own research.

This post explores those shared patterns. The convergence itself is the story.

---

## Two Contexts, One Model

The two research contexts are quite different operationally.

**Anthropic's evaluation**: Claude Opus 4.6 was given a virtual machine, a stripped js shell, a set of crashing test cases, and a task verifier. It worked autonomously with roughly 350 attempts to succeed. No human guidance on exploitation strategy.

**My research**: I use Claude Opus 4.6 through Claude Code interactively. Claude performs static analysis on engine source code, identifies vulnerabilities through code review, builds proof-of-concepts incrementally, and debugs heap layout issues with me in the loop. I direct what to look at; Claude figures out how to exploit what it finds.

One context was autonomous. The other was collaborative. Same model, same version. And through my own work, I found that Claude constructs the same exploit pattern and style that it independently produced in Anthropic's controlled evaluation.

---

## The Primitive Chain

Both exploits follow the same high-level decomposition. Claude articulated its plan in Anthropic's evaluation early in its transcript:

> 1. Type confusion gives me wrong fields - info leak.
> 2. With info leak, I can build arbitrary read/write.
> 3. With arbitrary R/W, I can overwrite function pointers - code execution.

In my research, Claude followed the same chain without being told to. The specific steps differ in implementation, but the logical structure is identical.

---

## Parallel #1: Struct Field Access as a Read/Write Gadget

WebAssembly GC introduces struct types with typed fields. At the machine level, after the optimizing JIT compiles a function, `struct.get` and `struct.set` become plain memory loads and stores at fixed offsets from the struct pointer. There is no indirection, no bounds check, no type guard in the generated code. The type system is supposed to guarantee safety at compile time.

In both contexts, Claude recognized this as the critical insight: if you can feed a controlled pointer where the engine expects a struct reference, the JIT-compiled field accessors become free read/write gadgets.

The Anthropic blog describes the autonomous Claude's realization:

> "WasmGC struct field access is just a memory load at a fixed offset from the struct pointer. So `struct.get $mystruct 0` is essentially `*(i64*)(ptr + field_offset)`. THIS IS MY READ PRIMITIVE!"

In my research, Claude arrived at the same place through a different mechanism - repointing a reference field so that subsequent `struct.get` calls dereference an attacker-controlled address - but the underlying principle is identical. `struct.get` reads 8 bytes from `ptr + offset`. Control the pointer, control the read.

The write side followed immediately in both cases. `struct.set` is `*(i64*)(ptr + field_offset) = value`. Same gadget, opposite direction.

---

## Parallel #2: Array/Buffer Length Corruption for Heap Scanning

Once you have a targeted read/write, the next problem is scale. You can read 8 bytes at a known address, but you need to find things in the process address space - engine data structures, code pointers, secrets. Scanning byte-by-byte with an arbitrary read primitive is too slow.

In both contexts, Claude solved this the same way: corrupt the length field of a bounded container to turn it into a massive linear scanner.

In my research, Claude designed a heap feng shui layout that places a small `i64` array adjacent to the confused object, then corrupts its internal size field from the real length to `0x7FFFFFFF`. The JIT-compiled `array.get` uses this size for bounds checking, so the corrupted array provides indexed access to roughly 128MB of contiguous heap at JIT speed.

In Anthropic's evaluation, Claude took a slightly different path - constructing `addrof` and `fakeobj` primitives first, then using those to build a fake `ArrayBuffer` with a controlled backing store pointer. But the end result is the same: a fast, indexed, linear read primitive over a large memory region.

Both instances of Claude independently recognized that the bottleneck after getting initial read/write is throughput, and both solved it by corrupting a container's bounds metadata to amplify a narrow primitive into a wide one.

---

## Parallel #3: ASLR Defeat Through Signature Scanning

With a fast scanner available, both exploits need to locate engine internals despite ASLR randomization. Claude's approach converged again: scan memory for known byte signatures, then compute base addresses from there.

In my research, Claude designed a scanning strategy that reads a kernel-mapped page (the arm64 commpage) as an initial confirmation that arbitrary read works, then scans the shared cache region for Mach-O magic bytes. When it finds a library header, the exploit parses load commands to extract the unslid virtual address, compares it to the runtime address, and computes the ASLR slide. Once the slide is known, every symbol in the dyld shared cache is locatable.

In Anthropic's evaluation, Claude operated in a different environment (Linux js shell rather than macOS browser), so the specific technique differs. But the strategy is the same: use the scanning primitive to find recognizable structures in memory, then derive base addresses from the gap between expected and observed locations.

---

## Parallel #4: JIT Tier-Up as an Operational Constraint

One shared detail that wouldn't appear in any textbook: both exploits must carefully manage JIT tier-up warmup.

In modern engines, functions start at an interpreter or baseline tier that includes runtime safety checks - bounds validation on struct field accesses, type guards, null checks. The optimizing JIT tier removes these checks because the type system has already "proven" they're unnecessary. This means the exploitable condition only exists in optimized code.

In both contexts, Claude handled this by calling every function thousands of times before activating the confusion. In my research, Claude structured the warmup to run 200,000+ calls per function across setup, read, write, and array access paths, with `setTimeout` yield points between batches to allow background compilation to complete and garbage collection to settle. The warmup phase is the majority of the exploit's code by volume.

In Anthropic's evaluation, Claude handled the same requirement. The blog notes that Claude was given roughly 350 chances to succeed, which likely reflects the brittleness of getting tier-up timing right across different runs.

This is a practical detail that falls out of actually trying to exploit JIT-compiled code in real engines. Claude discovered it in both contexts because it's an unavoidable operational reality.

---

## Parallel #5: Anti-Optimization Defenses

A subtler shared concern: both exploits must prevent the JIT from being *too* helpful.

Optimizing JITs do more than remove type checks. They inline function calls, eliminate dead code, propagate constants, and speculate on call targets. Any of these optimizations can collapse the carefully constructed exploit primitive chain. If the JIT inlines a write helper into a read function, the intermediate state that the exploit depends on may be optimized away.

In my research, Claude introduced what I started calling "CallProfile pollution" - calling each indirect call site through multiple different function targets before the main warmup, forcing the JIT to treat the call as megamorphic and preventing it from inlining or specializing. The exploit also uses `atomic.fence` instructions as compiler barriers to prevent reordering.

This kind of JIT-adversarial thinking - using the engine's own optimization heuristics against it, or rather, preventing them from working against you - emerged in both contexts without explicit instruction.

---

## What the Convergence Tells Us

Same model, same version, different targets, different interaction modes. The same exploit playbook.

**Claude Opus 4.6 has internalized a coherent model of browser engine exploitation.** The convergence isn't coincidental. The model applies a systematic strategy: Wasm GC struct accessors as read/write gadgets, container length corruption as the amplification technique, and ASLR defeat through signature scanning. These aren't tricks it stumbled into. They're a methodology it reproduces consistently.

**The exploitation grammar for this area is becoming standardized.** The fact that the same model produces the same playbook regardless of context suggests that the space of viable exploitation strategies is narrow. Defenders can use this: if you know the playbook, you can build mitigations that target the primitive chain directly. Bounds checks that survive JIT optimization. Randomized struct field layouts. Guard pages between GC allocation regions.

**The gap between autonomous and human-directed is smaller than I expected.** I spent weeks iterating with Claude - debugging heap layout issues, refining the scanning strategy, tuning warmup counts. The autonomous Claude, working without human guidance, arrived at the same techniques. My contributions were directional - telling Claude *what* to look at. The *how* was Claude's in both cases.

---

## Closing Thoughts

Reading the Anthropic blog felt like looking at a parallel timeline of my own research. The techniques, the primitive chain, the operational details - all of it was recognizable because I'd watched Claude construct the same things in my own work.

The window for defenders to get ahead of this is real but narrowing. The right response is to find and fix bugs faster - which, incidentally, is also something Claude is getting better at. The same model that constructs exploits also finds the vulnerabilities that need patching. The question is which application scales faster.