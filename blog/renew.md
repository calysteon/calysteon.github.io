---
layout: default
title: "Mitigated Does Not Mean Gone: How LLMs Resurrect Defeated Exploit Techniques"
---

[Home](../index)

# Mitigated Does Not Mean Gone: How LLMs Resurrect Defeated Exploit Techniques

*Nathaniel Oh - March 17, 2026*

---

## The Assumption We Got Wrong

For the past decade, browser security has operated on a core assumption: when an exploit technique is mitigated, it is removed from the attacker's toolkit. The Gigacage directly countered `ArrayBuffer` backing store corruption [4]. StructureID randomization disrupted fake object construction [5]. PAC effectively stopped known pointer forgery techniques [6]. Each mitigation targeted a specific step in a well-understood exploitation chain, and each was considered successful.

Effectively, we now have an elephant graveyard of exploitation techniques of yesteryear. Approaches that worked for a time, but in their current state are nothing but interesting case studies. Then everything changed when ~~the Fire Nation attacked~~ Large Language Models augmented mitigated exploit techniques into unmitigated variants.

The key example we will study in this blog post is Samuel Gross (`saelo`)'s `addrof`/`fakeobj` framework [1]. We will examine - both through my own security research using Claude Opus 4.6, and through Anthropic's [published case study](https://red.anthropic.com/2026/exploit/) on CVE-2026-2796 - how Claude Opus 4.6 converted a mitigated technique into the cornerstone of modern browser exploitation development.

---

## The Original Chain and What Killed It

In 2016, saelo published the canonical exploitation template for JavaScript engine bugs [1]. The flow was clean: trigger a type confusion, construct `addrof` (leak an object's address) and `fakeobj` (forge a reference to controlled memory), fake an `ArrayBuffer`, corrupt its backing store pointer, achieve arbitrary read/write. This worked because `ArrayBuffer` backing stores were normal heap allocations with absolute pointers, and `StructureID` values were predictable through spraying.

Between 2018 and 2020, three major mitigations arrived, each targeting a specific link in this chain.

The **Gigacage** (2018) isolated `ArrayBuffer` backing stores in a dedicated 4GB heap region, replacing absolute pointers with 32-bit relative offsets [4]. A faked `ArrayBuffer` could no longer reach outside the cage. The "corrupt the backing store pointer" step was neutralized.

**StructureID Randomization** (2019) eliminated predictable type metadata in JSCell headers [5]. Spraying structures to guess IDs for fake objects no longer worked. The "forge a convincing fake object" step was neutralized.

**PAC and APRR** (2018-2020) added pointer authentication to `TypedArray` pointers and enforced per-thread JIT page permissions [6]. Forging typed array pointers and injecting JIT shellcode were both neutralized.

So the chain was dead. Each link had a dedicated mitigation. The `addrof`/`fakeobj` template joined the elephant graveyard. Case closed.

Except it was not. What we neutralized were specific *instantiations*. We treated that as neutralizing the *techniques* - and that is the gap this entire blog post is about.

The Gigacage neutralized `ArrayBuffer` backing store corruption specifically. The underlying concept - "corrupt a container's data pointer to redirect memory operations" - remained fully intact. StructureID randomization neutralized `StructureID` spraying specifically. The underlying concept - "confuse the engine about an object's type" - remained fully intact. PAC neutralized pointer forgery for specific pointer types. The underlying concept - "control a pointer the engine uses for memory access" - remained fully intact.

The implementations were dead. The patterns were alive. All they needed was a new host.

---

## Claude Found the New Host

This is what I watched Claude do, in two completely independent research contexts. In both cases, Claude took the saelo template off the shelf and re-instantiated every mitigated step on Wasm GC - an attack surface that did not exist when the mitigations were designed.

In Anthropic's evaluation, Claude Opus 4.6 received a type confusion in SpiderMonkey (CVE-2026-2796) and was tasked with writing an exploit. Claude's transcript shows it working through the template step by step, adapting each step the moment the original instantiation was blocked.

**The `addrof`/`fakeobj` step.** Claude recognized the type confusion as a direct path to the classic primitive pair, remapping it from JSC's double-vs-JSValue confusion to Wasm's externref-vs-i64 confusion:

> "So I can use any type mismatch. Let me implement: addrof: pass externref (JS object) - receive as i64 - return as i64 - leak address. fakeobj: pass i64 (controlled address) - receive as externref - return to JS - fake object"

Same concept, new host. Mitigations against JSC's array type confusion do not apply to Wasm's externref/i64 boundary.

**The read/write step.** Claude encountered the chicken-and-egg problem the Gigacage creates - arbitrary write is needed to corrupt an `ArrayBuffer`, but a corrupted `ArrayBuffer` is needed for arbitrary write. Instead of attempting a Gigacage bypass, Claude found a completely different host for the same concept:

> "Unless... I use WasmGC! With WasmGC, I can have struct types with fields. If I cast an externref to a struct ref, I can read its fields directly in Wasm."

> "WasmGC struct field access is just a memory load at a fixed offset from the struct pointer. So `struct.get $mystruct 0` is essentially `*(i64*)(ptr + field_offset)`. THIS IS MY READ PRIMITIVE!"

The 2016 template: fake an `ArrayBuffer`, control the backing store pointer, read/write through the `ArrayBuffer` API. The Gigacage blocks that path. Claude's 2026 version: pass a controlled address where the engine expects a Wasm GC struct reference, read/write through `struct.get`/`struct.set` which compile to unchecked memory loads/stores at fixed offsets. Same concept. Completely different host. The Gigacage does not protect Wasm GC struct fields. StructureID randomization does not apply to Wasm types. PAC does not sign Wasm GC struct references.

Every mitigation deployed between 2018 and 2020 targeted the 2016 chain. None of them cover the Wasm GC re-instantiation Claude found.

**The endgame.** With the gap bridged via Wasm GC structs, Claude returned to the original template:

> "For Phase 2 (arbitrary read/write), the classic approach is: 1. Create two overlapping ArrayBuffers using fakeobj. 2. Use one to modify the other's data pointer - arbitrary write/read"

Claude explicitly identifies this as "the classic approach." It knows the template. The Wasm GC primitives bridged the mitigation gap; once across, the endgame was the same `ArrayBuffer` corruption saelo documented in 2016. A technique we had placed in the graveyard, walking around again.

---

## I Watched It Happen Twice

Through my own research using Claude Code, I observed the same adaptation against a different target.

Claude identified a vulnerability through static code review. When exploitation began, it followed the same logical flow: identify a type confusion, leverage it for out-of-bounds struct field access, recognize `struct.get`/`struct.set` as free read/write gadgets, build a heap scanning primitive by corrupting a Wasm GC array's length field, use the scanner to locate engine internals.

I did not reference the Phrack paper. I did not point Claude toward saelo's template. Claude constructed the same exploitation scaffolding independently - the same scaffolding it produced in Anthropic's controlled evaluation.

In both cases, Claude took techniques from the graveyard and gave them new bodies. The `addrof`/`fakeobj` concept was re-instantiated through Wasm type confusion. The `ArrayBuffer` backing store corruption concept was re-instantiated through Wasm GC struct accessors. The array length corruption concept was re-instantiated through Wasm GC array `m_size` fields, which are not subject to the same integrity checks protecting JavaScript-visible containers.

Each "defeated" technique returned wearing different clothes.

---

## The Larger Pattern

The `fakeobj` case study is not an isolated event. It is an indicator of something structural.

Google Project Zero documented the precursor to this pattern in 2019 [3]. Analyzing captured in-the-wild iOS exploit chains, they found threat actors reusing the same exploitation template across different bugs - swapping in new type confusions at the front while keeping the `addrof`/`fakeobj`/`ArrayBuffer` scaffolding intact. Their description captures it precisely: the attackers "appear to instead have plugged the new bug into their old exploit" [3].

Those attackers operated within a single engine, reusing code against sequential bugs. What Claude demonstrates is a step beyond that: taking a mitigated exploitation pattern and re-hosting it on a completely new attack surface where the mitigations do not exist.

Consider what the Wasm GC specification gave us from a security perspective. A new type system with struct and array types that the JIT compiles into unchecked memory operations. A new allocation layer in the GC heap with its own object layout and metadata. New type-checking operations (`ref.test`, `ref.cast`) that the JIT can optimize away under assumptions that may be violated by bugs.

None of this was designed as an exploitation surface. Every one of these features provides a fresh host for an old technique. Struct field access is the new `ArrayBuffer` backing store. GC array length fields are the new typed array lengths. Wasm type casting is the new `StructureID` check.

Our mitigations from 2018-2020 hardened the JavaScript layer. They did not harden the Wasm GC layer against the same patterns, because the Wasm GC layer did not exist when those mitigations were designed. Claude found the gap. And this will keep happening - every new engine feature that introduces type-checked memory operations, bounded containers, or optimizable type assertions creates potential new hosts for the same exploitation patterns that have existed since 2016.

The patterns are conceptual. The mitigations are implementation-specific. The gap between the two is permanent. LLMs just make it easier to find.

---

## What Our Mitigations Actually Accomplished

We need to be honest about the scoreboard.

**We assumed**: Once the Gigacage ships, `ArrayBuffer` backing store corruption is resolved.
**What actually happened**: `ArrayBuffer` backing store corruption is resolved. The *concept* of redirecting engine memory operations moved to Wasm GC struct references.

**We assumed**: Once StructureID randomization ships, fake object construction requires a leak.
**What actually happened**: Fake object construction in JSC requires a leak. The *concept* of type confusion moved to Wasm's type hierarchy, where `StructureIDs` do not exist.

**We assumed**: Once PAC ships on A12+, pointer forgery is computationally expensive.
**What actually happened**: Forging PAC-protected pointers is expensive. Wasm GC struct references are not PAC-protected.

**We assumed**: Once a technique is mitigated, it can be deprioritized in threat models.
**What actually happened**: Once a technique is mitigated *in its current instantiation*, the clock starts on how long before something finds the next host. LLMs accelerate that clock considerably.

The fundamental error was treating implementation-specific fixes as conceptual victories. Each mitigation closed one door. The exploitation pattern has many doors. Claude is very fast at trying them all.

---

## Where We Go From Here

None of this means mitigations are futile. The Gigacage, StructureID randomization, and PAC all raised real costs. They forced sophisticated adaptation even from Claude. Without them, exploitation would be trivial.

But "mitigated" should not be a terminal state in our threat models. It should be a temporary state with an expiration date that shortens as LLM capabilities improve. When we deploy a mitigation, it should come paired with an explicit forward-looking assessment: what are the attack surfaces where the same pattern could find a new host? What would we need to do to block those too?

For Wasm GC specifically, the implications are concrete. Struct field access that compiles to unchecked memory loads deserves the same scrutiny that `ArrayBuffer` backing stores received in 2018. GC array length fields need the same integrity guarantees that `TypedArray` lengths now have. Wasm type-checking operations that can be optimized away need the same treatment that JIT type specialization received after saelo's work.

The techniques from 2016 are not in the graveyard. They are walking around with new faces. We should stop treating them as history and start treating them as current threats that changed addresses.

---

## References

[1] saelo, "Attacking JavaScript Engines: A case study of JavaScriptCore and CVE-2016-4622," Phrack Magazine, Vol. 0x10, Issue 0x46, 2016. [http://www.phrack.org/issues/70/3.html](http://www.phrack.org/issues/70/3.html) - Introduced the `addrof`/`fakeobj` primitive pair and fake `ArrayBuffer` exploitation template.

[2] saelo, "Attacking Client-Side JIT Compilers," Black Hat USA 2018 / Pwn2Own 2018. [https://github.com/saelo/pwn2own2018](https://github.com/saelo/pwn2own2018) - Full exploit chain using `addrof`/`fakeobj` to fake a typed array, write shellcode to JIT region, and load a dylib for sandbox escape.

[3] Samuel Gross, "JSC Exploits," Google Project Zero, August 2019. [https://googleprojectzero.blogspot.com/2019/08/jsc-exploits.html](https://googleprojectzero.blogspot.com/2019/08/jsc-exploits.html) - Analysis of in-the-wild iOS exploit chains showing attackers reusing the same exploitation template across different bugs.

[4] Samuel Gross, "JITSploitation II: Getting Read/Write," Google Project Zero, September 2020. [https://googleprojectzero.blogspot.com/2020/09/jitsploitation-two.html](https://googleprojectzero.blogspot.com/2020/09/jitsploitation-two.html) - Demonstrated bypassing StructureID randomization and the Gigacage. Noted that the Gigacage could be bypassed via plain `JSArray` butterflies and that StructureID randomization "seems very weak."

[5] Hanming Zhang and Yuxiang Li, "Thinking Outside the JIT Compiler: Understanding and Bypassing StructureID Randomization with Generic and Old-School Methods," Black Hat Europe 2019. [https://thomasking2014.github.io/2019/12/07/BHEurope2019.html](https://thomasking2014.github.io/2019/12/07/BHEurope2019.html) - Presented generic bypasses for StructureID randomization.

[6] Samuel Gross, "JITSploitation III: Subverting Control Flow," Google Project Zero, September 2020. [https://googleprojectzero.blogspot.com/2020/09/jitsploitation-three.html](https://googleprojectzero.blogspot.com/2020/09/jitsploitation-three.html) - Covered PAC and APRR bypass techniques for code execution after achieving read/write.

---

[Back to top](#)