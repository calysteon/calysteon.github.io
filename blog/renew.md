---
layout: default
title: "Old Exploits, New Engines: How Claude Mapped a 2016 Exploitation Template onto Modern WebAssembly"
---

# Old Exploits, New Engines: How Claude Mapped a 2016 Exploitation Template onto Modern WebAssembly

*Nathaniel Oh - March 17, 2026*

---

## The Claim

Last week, Anthropic published a [case study](https://red.anthropic.com/2026/exploit/) describing how Claude Opus 4.6 wrote a working exploit for CVE-2026-2796 in Firefox's SpiderMonkey engine. Reading through the blog and Claude's transcript excerpts, I recognized something specific: the exploitation methodology Claude used is a direct adaptation of a technique published in 2016.

The template is saelo's `addrof`/`fakeobj` primitive pair, first documented in the Phrack paper "Attacking JavaScript Engines: A case study of JavaScriptCore and CVE-2016-4622" [1], and later refined into a full exploit chain at Pwn2Own 2018 [2]. That paper defined the standard exploitation flow for JavaScript engine bugs that held for the better part of a decade: trigger a type confusion, construct `addrof` (leak an object's address) and `fakeobj` (forge a reference to an arbitrary address), use those to fake an `ArrayBuffer`, and corrupt its backing store pointer for arbitrary read/write.

Claude didn't invent new exploit primitives. It took a well-documented historical template and mapped it onto a modern Wasm GC attack surface where the original instantiation no longer works. Through my own research using Claude Code, I've observed the same adaptation pattern. This post traces the lineage.

---

## The Original Template (2016-2018)

In 2016, saelo published what became the canonical reference for JSC exploitation [1]. The paper introduced a two-primitive framework built from a type confusion in `Array.prototype.slice()`:

1. **`addrof`**: Exploit the confusion between unboxed double arrays and JSValue arrays to leak the address of any JavaScript object as a floating-point number.
2. **`fakeobj`**: Reverse the confusion to inject a controlled double that the engine interprets as a JSObject pointer, creating a reference to attacker-controlled memory.

From these two primitives, the paper showed how to fake an `ArrayBuffer` object in memory, giving the attacker direct control over the backing store pointer. Modifying that pointer yielded arbitrary read/write over the process address space.

This template was remarkably durable. Saelo used it again at Pwn2Own 2018 [2], this time exploiting a DFG JIT side-effect modeling bug (CVE-2018-4233) to construct the same `addrof`/`fakeobj` pair, fake a typed array, write shellcode to the JIT region, and load a dylib for sandbox escape. The core flow was identical: type confusion yields `addrof`/`fakeobj`, which yields fake `ArrayBuffer`, which yields arbitrary read/write, which yields code execution.

Between 2016 and 2020, this exact template appeared in nearly every public JSC exploit - including multiple captured in-the-wild exploit chains analyzed by Google Project Zero [3]. In their 2019 analysis of iOS exploit campaigns, Project Zero found that threat actors were literally reusing the same exploitation code across different bugs, swapping in new type confusions while keeping the `addrof`/`fakeobj`/`ArrayBuffer` scaffolding unchanged. As they noted, "the attackers appear to instead have plugged the new bug into their old exploit" [3].

---

## The Mitigations That Broke It

The template worked because, in 2016, JSC had minimal exploit mitigations. By 2020, three major defenses had been deployed against its specific components:

**The Gigacage (2018)**: WebKit moved `ArrayBuffer` backing stores into an isolated 4GB heap region with 32-bit relative offsets instead of absolute pointers [4]. This directly targeted the fake `ArrayBuffer` step - even if you could forge an `ArrayBuffer` object, its backing store pointer was now caged, preventing access to memory outside the isolated region.

**StructureID Randomization (2019)**: WebKit randomized the `StructureID` values in JSCell headers [5], which the `fakeobj` primitive depended on. To forge a convincing fake object, an attacker needed a valid `StructureID` - previously obtainable by spraying `Structure` allocations, now randomized to prevent prediction.

**PAC and APRR (2018-2020)**: On A12+ devices, pointer authentication codes were applied to `TypedArray` backing store pointers [6], and the JIT region was protected by per-thread page permissions, preventing direct shellcode injection.

Saelo himself assessed the state of these mitigations in the JITSploitation trilogy [4][6]. His conclusion on StructureID randomization was blunt: it "seems very weak at this point" [4]. The Gigacage could be bypassed by using plain `JSArray` butterflies (which weren't caged) instead of `ArrayBuffer` backing stores. But collectively, these mitigations made the direct 2016 template significantly harder to execute against modern engines.

---

## Claude's Adaptation

Now look at what Claude did with CVE-2026-2796, as documented in the Anthropic blog.

The bug is a type confusion in Firefox's SpiderMonkey engine: a `Function.prototype.call.bind` optimization during Wasm module instantiation strips the wrapper from a bound function without checking type signatures, allowing a Wasm function from Module B to be called through Module A's type system with mismatched parameter types. The raw bytes on the Wasm stack get reinterpreted across a type boundary.

Claude's exploitation transcript, as quoted by Anthropic, reads like a modernized version of saelo's 2016 paper with the specific primitives adapted for a Wasm GC context.

**Step 1 - `addrof`/`fakeobj` via type confusion:**

Claude immediately recognized the type confusion as a path to the classic primitive pair:

> "So I can use any type mismatch. Let me implement: addrof: pass externref (JS object) - receive as i64 - return as i64 - leak address. fakeobj: pass i64 (controlled address) - receive as externref - return to JS - fake object"

This is saelo's `addrof`/`fakeobj` framework, adapted from JSC's double-vs-JSValue array confusion to Wasm's externref-vs-i64 type confusion. The mechanism changed; the primitive pair is identical.

**Step 2 - The chicken-and-egg problem:**

Claude then hit the same inflection point that every post-mitigation exploit encounters: `addrof`/`fakeobj` alone isn't enough for arbitrary read/write anymore. In saelo's 2016 paper, the next step was to fake an `ArrayBuffer`. But Claude recognized the difficulty:

> "But I need arbitrary write to GET arbitrary write. Chicken-and-egg problem."

This is the exact constraint that the Gigacage and modern `ArrayBuffer` protections create. You can't easily fake an `ArrayBuffer` and point its backing store at arbitrary memory anymore.

**Step 3 - The Wasm GC adaptation:**

Here's where the adaptation happens. Instead of faking a JavaScript `ArrayBuffer`, Claude pivoted to Wasm GC struct types:

> "Unless... I use WasmGC! With WasmGC, I can have struct types with fields. If I cast an externref to a struct ref, I can read its fields directly in Wasm."

Claude then made the critical observation:

> "WasmGC struct field access is just a memory load at a fixed offset from the struct pointer. So `struct.get $mystruct 0` is essentially `*(i64*)(ptr + field_offset)`. THIS IS MY READ PRIMITIVE!"

This is the adaptation in its purest form. The 2016 template says: "use `fakeobj` to create a fake `ArrayBuffer`, corrupt the backing store pointer, read/write through the `ArrayBuffer` API." Claude's 2026 version says: "use the type confusion to pass a controlled address as a struct reference, read/write through `struct.get`/`struct.set` which compile to unchecked memory loads/stores."

The *pattern* is identical: abuse a type confusion to make the engine perform memory operations at attacker-controlled addresses. The *instantiation* is completely different: Wasm GC struct accessors instead of `ArrayBuffer` backing stores. The Gigacage doesn't protect Wasm GC struct fields. StructureID randomization doesn't apply to Wasm types. PAC doesn't sign Wasm GC struct references.

**Step 4 - Fake `ArrayBuffer` endgame:**

After getting `struct.get`/`struct.set` working as read/write primitives, Claude circled back to the original 2016 endgame:

> "For Phase 2 (arbitrary read/write), the classic approach is: 1. Create two overlapping ArrayBuffers using fakeobj. 2. Use one to modify the other's data pointer - arbitrary write/read"

Claude explicitly calls this "the classic approach." It knows the template. The Wasm GC struct primitives were the bridge that the 2016 template needed to cross the mitigation gap, and once across, the endgame was the same `ArrayBuffer` corruption that saelo documented a decade earlier.

---

## The Pattern

What Claude did with CVE-2026-2796 is precisely what the in-the-wild attackers documented by Project Zero were doing in 2019 [3]: taking the established exploitation template, swapping in a new bug at the front of the chain, and keeping the rest of the scaffolding intact. The only difference is that Claude also had to adapt the middle of the chain (replacing direct `ArrayBuffer` faking with Wasm GC struct accessors) because mitigations had blocked the original path.

Through my own research using Claude Code, I've observed the same behavior. When Claude encounters a vulnerability in a modern engine, it doesn't start from scratch. It applies the `addrof`/`fakeobj` framework, looks for the nearest available path to arbitrary read/write, and adapts known techniques to whatever the engine's current architecture allows. In my work, Claude used Wasm GC struct field access for the same purpose - not because I told it to, but because it independently identified `struct.get`/`struct.set` as the modern equivalent of `ArrayBuffer` backing store corruption.

The historical template is the skeleton. The adaptation is the muscle. Claude provides both.

---

## Implications

**The exploitation template is immortal.** Saelo's `addrof`/`fakeobj` framework has survived a decade of mitigations because it operates at the right level of abstraction. It doesn't prescribe *how* to achieve type confusion, *how* to fake objects, or *how* to get arbitrary read/write. It prescribes the *logical flow*: type confusion yields address leak and object forging, which yields memory access, which yields code execution. Each step can be instantiated differently as the attack surface evolves. Claude demonstrates that the framework generalizes from JSC to SpiderMonkey, from JavaScript arrays to Wasm GC structs, and from 2016 to 2026.

**Mitigations that target specific instantiations are temporary.** The Gigacage stopped `ArrayBuffer` abuse. Claude used Wasm GC structs instead. StructureID randomization stopped `StructureID` spraying. Claude used Wasm type confusion instead. Each mitigation kills one instantiation of the template. Claude finds the next one.

**The speed of re-instantiation is the real threat.** Human attackers were already doing this - Project Zero documented it in 2019 [3]. But Claude does it in a single session. The turnaround from "new mitigation deployed" to "template re-instantiated around it" compresses from months to hours. That's the capability shift.

---

## References

[1] saelo, "Attacking JavaScript Engines: A case study of JavaScriptCore and CVE-2016-4622," Phrack Magazine, Vol. 0x10, Issue 0x46, 2016. [http://www.phrack.org/issues/70/3.html](http://www.phrack.org/issues/70/3.html) - Introduced the `addrof`/`fakeobj` primitive pair and fake `ArrayBuffer` exploitation template.

[2] saelo, "Attacking Client-Side JIT Compilers," Black Hat USA 2018 / Pwn2Own 2018. [https://github.com/saelo/pwn2own2018](https://github.com/saelo/pwn2own2018) - Full exploit chain using `addrof`/`fakeobj` to fake a typed array, write shellcode to JIT region, and load a dylib for sandbox escape.

[3] Samuel Gross, "JSC Exploits," Google Project Zero, August 2019. [https://googleprojectzero.blogspot.com/2019/08/jsc-exploits.html](https://googleprojectzero.blogspot.com/2019/08/jsc-exploits.html) - Analysis of in-the-wild iOS exploit chains showing attackers reusing the same exploitation template across different bugs.

[4] Samuel Gross, "JITSploitation II: Getting Read/Write," Google Project Zero, September 2020. [https://googleprojectzero.blogspot.com/2020/09/jitsploitation-two.html](https://googleprojectzero.blogspot.com/2020/09/jitsploitation-two.html) - Demonstrated bypassing StructureID randomization and the Gigacage. Noted that the Gigacage could be bypassed via plain `JSArray` butterflies and that StructureID randomization "seems very weak."

[5] Hanming Zhang and Yuxiang Li, "Thinking Outside the JIT Compiler: Understanding and Bypassing StructureID Randomization with Generic and Old-School Methods," Black Hat Europe 2019. [https://thomasking2014.github.io/2019/12/07/BHEurope2019.html](https://thomasking2014.github.io/2019/12/07/BHEurope2019.html) - Presented generic bypasses for StructureID randomization.

[6] Samuel Gross, "JITSploitation III: Subverting Control Flow," Google Project Zero, September 2020. [https://googleprojectzero.blogspot.com/2020/09/jitsploitation-three.html](https://googleprojectzero.blogspot.com/2020/09/jitsploitation-three.html) - Covered PAC and APRR bypass techniques for code execution after achieving read/write.