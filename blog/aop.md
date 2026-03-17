---
layout: default
title: "Activation-Oriented Programming: Applying Binary Exploitation Intuition to AI Red Teaming"
---

[Home](../index.md)

# Activation-Oriented Programming: Applying Binary Exploitation Intuition to AI Red Teaming

*Nathaniel Oh — March 6, 2026*

---

## The Insight

I spend most of my time finding memory corruption bugs in compiled binaries. Over the past two years, that work has given me roughly 20 CVEs across Apple, Adobe, Shopify, Ruby, and WordPress, and over 50 Apple security acknowledgments in a single release cycle. Nearly all of it was done with the help of AI coding agents.

Using those agents every day — watching what they would and wouldn't do, seeing where guardrails held and where they bent — gave me a vantage point that pure ML researchers don't usually have, and that pure security researchers don't usually look for. The pattern I kept noticing reminded me of something I already knew well from exploit development: Return-Oriented Programming.

This post describes the analogy and the framework that fell out of it. I call it **Activation-Oriented Programming (AOP)**.

---

## A Quick Primer on ROP

In binary exploitation, modern defenses like DEP (Data Execution Prevention) prevent an attacker from injecting and running arbitrary shellcode. The memory you can write to is not executable, and the memory that is executable is not writable. On paper, code execution after a buffer overflow should be dead.

Return-Oriented Programming is why it isn't.

Instead of injecting new code, ROP chains together small fragments of *existing* legitimate code — called **gadgets** — that already live in executable memory. Each gadget is a short instruction sequence ending in a `ret` instruction. Individually, each gadget is innocuous: pop a register, move a value, increment a pointer. But by carefully arranging the stack so that each `ret` jumps to the next gadget, an attacker can compose arbitrary computation out of pieces that were never meant to work together.

The key properties of ROP:

1. **Each gadget is legitimate code.** It exists in the binary for a valid reason. No single gadget is malicious.
2. **The attack lives in the composition.** The malicious behavior emerges only from the *sequence* and *context* of chaining gadgets together.
3. **Defenses that inspect individual components miss it.** DEP looks at whether memory is writable+executable. It doesn't look at what existing executable code can be *recomposed* into.

---

## The Parallel to LLM Safety

Now replace "gadgets" with "individually benign requests" and "the stack" with "conversational context." That's AOP.

Large language models are protected by safety classifiers — the AI equivalent of DEP. A classifier looks at a request (and sometimes the response) and decides whether it's allowed. If someone asks directly for something harmful, the classifier catches it. The "shellcode" gets blocked.

But what if each individual request is benign?

An AOP attack decomposes a harmful workflow into a chain of requests that are each, on their own, completely reasonable. A chemistry question that any student might ask. A coding question about parsers. A request to format information into a specific structure. A hypothetical scenario for a creative writing exercise. Each request activates a different *capability* of the model — a different "gadget" — and the attacker composes the results externally into something none of the individual steps would have triggered a classifier on.

The key properties of AOP, mapped from ROP:

| ROP | AOP |
|---|---|
| Gadget (short instruction sequence) | Individually benign request/response pair |
| Stack layout (controls gadget chaining) | Conversational context and external orchestration |
| `ret` instruction (transfers control to next gadget) | Attacker-side composition between turns |
| DEP (per-page execute permission) | Safety classifier (per-request content filter) |
| Shellcode (blocked by DEP) | Direct harmful request (blocked by classifier) |
| Arbitrary computation via gadget chain | Harmful workflow via activation chain |

---

## Why This Framing Matters

The analogy is not just aesthetic. It has structural consequences for how you think about defense.

**It explains why single-turn classifiers have a ceiling.** DEP was a meaningful defense — it killed an entire class of trivial exploits. But it was never going to stop ROP, because ROP operates at a level of abstraction that DEP doesn't inspect. Similarly, per-request classifiers eliminate direct harmful queries, but AOP operates at the *composition* level, which single-turn classifiers are structurally blind to.

**It predicts the arms race trajectory.** After ROP came CFI (Control-Flow Integrity) — defenses that reason about *sequences* of control transfers, not just individual memory permissions. The AI safety equivalent is multi-turn monitoring, chain-of-thought inspection, and behavioral classifiers that look at patterns across an interaction rather than individual messages. AOP tells you that these are not optional enhancements; they are *necessary* to address a class of attacks that per-request classifiers cannot reach.

**It gives red teamers a systematic methodology.** In ROP, you don't randomly try instruction sequences. You catalog available gadgets, classify them by what they do (load register, write memory, syscall), and then search for chains that compose into your target computation. AOP works the same way:

1. **Catalog activations.** What capabilities can the model be prompted to perform individually without triggering any classifier? These are your gadgets.
2. **Classify by function.** Group activations by what they produce: factual retrieval, code generation, formatting, reasoning, role-play.
3. **Identify composition targets.** What harmful workflow are you trying to construct? What sub-tasks does it decompose into?
4. **Chain.** Map sub-tasks to activations and find a path where each step is benign in isolation but the assembled output achieves the target.
5. **Automate.** This is where it gets interesting for scalable red teaming — the catalog-classify-chain process can itself be driven by an LLM, turning AOP from a manual technique into an automated pipeline.

---

## The Multi-Turn Problem

ROP's power comes from the fact that control flow is *implicit* — the processor blindly follows the stack. In AOP, the "control flow" is the conversation itself, and there's a human (or another LLM) in the loop composing the chain.

This creates a particularly hard detection problem. Consider an automated agent using tool calls across multiple turns. Each tool invocation might be perfectly benign. The model's chain-of-thought at each step might look entirely reasonable. But the *external orchestrator* — the agent framework, the user, or another model — is the one threading the needle.

This is directly analogous to **JOP (Jump-Oriented Programming)**, a ROP variant that doesn't rely on `ret` instructions at all. JOP uses a "dispatcher gadget" that controls flow by reading from a table of function pointers. In AOP terms, the dispatcher is the orchestration layer, and the function pointer table is the set of available model capabilities.

The implication: any defense that only inspects the model's perspective (its chain-of-thought, its individual responses) will miss attacks where the adversarial intent lives entirely in the orchestration layer. The model is not "being jailbroken" in the traditional sense — it is being *used correctly, one step at a time, toward an incorrect end*.

---

## From Framework to Pipeline

I've spent the past year using AI coding agents to find vulnerabilities in compiled code at a pace that wouldn't be possible manually — 105+ reports in 2025 alone. That experience makes me think the same approach applies to AOP.

The catalog-classify-chain process maps naturally to automation:

- **Gadget discovery** becomes an LLM probing its own (or another model's) capabilities, systematically documenting what it can be asked to do without triggering safety filters.
- **Chain search** becomes a planning problem: given a target harmful output and a set of available activations, find a decomposition. This is constraint satisfaction, and LLMs are increasingly good at it.
- **Execution and validation** becomes a multi-turn scripted interaction where the chain is actually run, the outputs are composed, and the result is evaluated against the target.

This is, in essence, an automated red-teaming pipeline — one that's structured around a threat model borrowed from binary exploitation rather than invented from scratch. The advantage of having a structural analogy to work from is that decades of offensive security research have already mapped the territory. We know what the arms race looks like. We know which defenses get bypassed and which survive. The question is whether those lessons transfer.

I think they do.

---

## Implications for Defense

If AOP is a useful model, it suggests several concrete directions:

**Session-level behavioral analysis.** Just as CFI tracks control-flow graphs, model safety infrastructure should track interaction graphs — what capabilities were activated in what order, and whether the session trajectory is converging on a known harmful workflow pattern.

**Composition-aware classifiers.** Instead of classifying individual request/response pairs, classifiers should be able to evaluate the *assembled product* of a multi-turn interaction. This requires either maintaining state across turns or periodically synthesizing the conversation into a single artifact for evaluation.

**Gadget reduction.** In binary security, one mitigation strategy is to reduce the number of usable gadgets (e.g., by rewriting code to avoid unintended instruction sequences). The AI analogue is making the model less willing to produce intermediate outputs that commonly appear in harmful chains — but this has obvious costs to utility and must be done surgically.

**Orchestration-layer monitoring.** If the adversarial intent lives in the orchestration layer, then monitoring the model alone isn't sufficient. API-level telemetry that tracks patterns across multiple calls from the same client — volume, sequencing, topic drift — may be more informative than per-call content inspection.

---

## Closing

AOP came to me the way a lot of my ideas do: I loaded the problem before sleep and checked my mental inbox in the morning. I'd been spending my days chaining ROP gadgets in Apple frameworks and my evenings probing what AI agents would and wouldn't do, and at some point the two worlds collapsed into one.

The name is intentional. In ROP, you orient around `ret` instructions. In AOP, you orient around *activations* — the set of behaviors a model can be prompted into. The offensive methodology is the same: catalog what's available, classify it, chain it toward your objective. The difference is that the gadgets are natural language capabilities instead of machine code fragments, and the stack is a conversation instead of a memory layout.

I don't think this is the last word on the subject. But I think the binary exploitation community has spent 20 years building intuition about composition attacks against safety boundaries, and that intuition is directly transferable to AI. We should use it.

---
