---
layout: research_post
title: "Unlocking the Computation Inside Pretrained Models"
subtitle: "A second compatible neural improvement: cumulative development over pretrained components, complete results and an application test."
description: "Checkpoint 06 acquires a second compatible improvement over frozen pretrained components. Read the complete results, application test, paper and reproducible code."
author: Nathaniel Oh
date: 2026-11-13
last_modified_at: 2026-11-13T23:03:10+00:00
permalink: /research/shaping-gap-sustainable-ai/
image:
  path: https://calysteon.github.io/assets/images/shaping-gap-checkpoint06-social.png
  width: 1200
  height: 630
  alt: "Checkpoint 06: a second compatible neural improvement. Six protected program instances retained; paper and complete evidence public."
---

What if a pretrained model contains useful computation that its usual way of operating does not make accessible?

We normally encounter a language model through a familiar workflow: text enters, the transformer processes it, and a language-generation head produces token predictions. That workflow is a way to use the pretrained computation. It need not define everything we can do with it.

**Our research asks what happens when we separate the pretrained neural body from that usual workflow, and acquire a different execution system around it.**

Think of lifting a heavy rock and discovering the activity underneath. The rock, in this analogy, is the assumption that a pretrained model must always be used through the interface it arrived with. Lifting that assumption lets us investigate what other computations can be constructed from the same trained substrate.

With Dr. Paul Attie, I am pursuing this through [Shaping Gap Benchmark](https://github.com/calysteon/shaping-gap-benchmark). Checkpoint 04 demonstrated a learning-and-reuse cycle around a complete pretrained transformer body. Checkpoint 05 brought that cycle into a component runtime that executes no original transformer graph. **Checkpoint 06 now acquires a second neural improvement from that already improved runtime, while keeping earlier programs usable.**

The larger objective is **capability growth through better organization of computation we already have**. If useful improvements can accumulate economically, some capability growth could become less tightly coupled to bigger pretrained models and expanding data-centre capacity.

<div class="research-links">
<a class="research-button" href="https://github.com/calysteon/shaping-gap-benchmark/tree/734af7274a691fdffc3fe4ffe41c2b433d352f52/checkpoints/06-cumulative-contract-interfaces">Explore Checkpoint 06</a>
<a class="research-button research-button-secondary" href="https://github.com/calysteon/shaping-gap-benchmark/blob/734af7274a691fdffc3fe4ffe41c2b433d352f52/checkpoints/06-cumulative-contract-interfaces/paper/shaping-gap-benchmark-vi.pdf">Read Paper VI</a>
</div>

## November 13 update: a second improvement that keeps earlier programs usable

A first improvement asks whether a construction helps. A second asks whether the system can build on it without breaking what it already does.

**We have now demonstrated that second step in one 0.6B-derived component runtime.** Starting from the accepted Checkpoint 05 construction, the system recruits one additional original pretrained component and learns a shared interface against its earlier programs' requirements. Experience selects the execution/interface pair; two untouched terminal streams then test the retained construction.

| Nonlinear temporal-processing score | Already improved runtime | Second retained construction |
|:--|--:|--:|
| Terminal stream 1 | 1.38792 | **1.84921** |
| Terminal stream 2 | 1.13904 | **1.70404** |

Those gains are **33.24% and 49.60% on this finite processing measure**, with **6/6 protected program instances retained** across three frozen libraries. The feature-interface control also improves, by 20.52% and 27.61%; the interface-only control retains baseline. Each successful arm accepts one additional neural acquisition during five rounds. The stronger criterion of two additional acquisitions remains unmet.

<figure>
<img src="/assets/images/shaping-gap-checkpoint06.png" alt="Checkpoint 06 second retained processing gain with feature-interface control, plus full-cohort recruitment successes for 64, 128 and 512 candidates." width="2100" height="760" loading="lazy">
<figcaption>The left panel starts from the already improved Checkpoint 05 runtime. These scores measure finite numerical processing, rather than language-answer accuracy. The right panel reports the earlier recruitment study, including baseline fallbacks.</figcaption>
</figure>

The engineering distinction is how the system protects earlier work. Coordinate alignment tries to preserve every observed feature. The new **program-contract interface** learns a shared observation map using the numerical functions that the unchanged programs must continue to compute. Source targets train this map; new inputs assess whether the acquired pair actually improves while respecting those obligations.

Compatibility stays in **one 600-value map** as interfaces compose. The successful runtime retains **one recruited port** and 1,102,272 bytes of conservatively counted runtime/program arrays. The original pretrained weights remain fixed. Whole-process memory and the immutable component archive are separately reported; the active four-port cap is not yet full, so replacement has not been exercised.

### Screening more components, with the work charged

An initial study screened pools of 64, 128 and 512 candidates in fixed-size chunks. Fresh terminal gains occurred in 2, 3 and 4 of six inherited contexts. Random-128 and resident-bank controls each succeeded in three contexts. Larger inventories offered more opportunities but required more acquisition work; they did not produce a second gain on the already improved chain.

That failure identified the next experiment: acquire the interface against existing program requirements while retaining the same regression guards. The program-contract study then produced the second gain on new streams. Both studies, including rejected candidates and failed criteria, are in [Paper VI](https://github.com/calysteon/shaping-gap-benchmark/blob/734af7274a691fdffc3fe4ffe41c2b433d352f52/checkpoints/06-cumulative-contract-interfaces/paper/shaping-gap-benchmark-vi.pdf).

### A real forecast test locates the next commercial hurdle

We froze the resulting controllers and tested new readouts on a public chronological appliance-energy series. One-hour forecast RMSE fell from **83.1373 to 82.6682 Wh**, a **0.564% reduction**. The paired daily-block interval was 0.101–1.175%. Ten-minute error rose 2.220%; four-hour change was very small. A 64-lag ridge autoregression reference performed better at all three horizons.

The application acceptance criterion was not met. This gives the next acquisition a concrete job: **improve the longer-horizon forecast while protecting the short-horizon consumer and competing with the classical reference**. The study uses univariate consumption from one house; complete predictions, classical controls, preprocessing and the public dataset attribution are released.

The new foundation is a two-improvement chain with compatibility and full candidate accounting. Commercial impact now depends on turning that mechanism into competitive application quality and amortized lifecycle savings. The numerical recruitment, contract-acquisition and transfer studies took approximately 175, 49 and 5 seconds respectively in the recorded CPU environment; these are full-study times, not production request latencies. Pool extraction and peak memory are separately recorded in the [cost ledger](https://github.com/calysteon/shaping-gap-benchmark/blob/734af7274a691fdffc3fe4ffe41c2b433d352f52/checkpoints/06-cumulative-contract-interfaces/results/cost-ledger.json).

Independent replay checks 12,112 temporal score vectors, 300 acquired maps, 8,320 native response points and every application prediction. Fresh portable acquisition and forecasting reproduce the reported outcomes exactly. [Explore the complete checkpoint](https://github.com/calysteon/shaping-gap-benchmark/tree/734af7274a691fdffc3fe4ffe41c2b433d352f52/checkpoints/06-cumulative-contract-interfaces).

## Checkpoint 05: the learning cycle runs on neural components

The latest experiment uses **24 selected pretrained neural fragments per context** as its computational substrate. A new recurrent runtime connects them directly. It executes their native numerical functions without invoking transformer blocks, attention, tokenization or a language-generation head.

The system acquires response interfaces from component behavior and partial derivatives. It uses those interfaces to anticipate how one component's output changes the response of another. Actual neural execution produces the result; experience then selects an execution change together with a shared compatibility interface to retain.

In one 0.6B-derived context, scores on two untouched input streams improve:

| Numerical processing score | Before acquisition | Retained construction |
|:--|--:|--:|
| Fresh stream 1 | 0.75648 | **1.17401** |
| Fresh stream 2 | 0.79913 | **1.16017** |

The shared interface preserves **both eligible old programs**, versus one under direct reuse. Across all six pretrained contexts using integrated guidance, one update and five baseline fallbacks produce positive mean score changes in both streams, with **18/18** old program instances retained.

<figure>
<img src="/assets/images/shaping-gap-checkpoint05.png" alt="Checkpoint 05: one component-level retained improvement and results across all six contexts for each control." width="2120" height="760" loading="lazy">
<figcaption>Paper V reports all 24 model/context/control conditions. Absolute gains are shown because score scales differ across these experimental runtimes.</figcaption>
</figure>

The controls help locate the contribution. First-order guidance produces two strict successes; shuffled-conditioning and matched random-weight components produce one each. The result establishes an **acquisition and compatible-reuse cycle after replacing the original transformer graph**. The complete comparisons are in [Paper V](https://github.com/calysteon/shaping-gap-benchmark/blob/04870c5ff60838dde856e03687524721910cda5c/checkpoints/05-component-learning-cycle/paper/shaping-gap-benchmark-v.pdf).

The [checkpoint package](https://github.com/calysteon/shaping-gap-benchmark/tree/04870c5ff60838dde856e03687524721910cda5c/checkpoints/05-component-learning-cycle) includes extracted component banks, executable code, every experimental condition and an independent replay of 1,440 score vectors. It runs on a CPU without downloading the full source language models. Numerical materialization differences from the original mixed-precision components, preparation work and resource costs are documented alongside the results.

This brings the training-scaffold idea into an executable experiment: selected functions acquired during transformer pretraining can participate in a different computation, and that computation can undergo a measured learning-and-retention cycle. Checkpoint 06, above, takes the next acquisition from that accepted runtime and measures preservation of earlier work.

## Checkpoint 04: separating computation from language generation

In Checkpoint 04, tokenization, English prompts and the language-generation head are outside the execution path. Continuous numerical state enters the pretrained body directly. We observe its neural response and acquire connections that return that response as the next recurrent state.

**The complete transformer body still executes: all 28 blocks and final normalization.** Its attention and feed-forward operations, with their frozen pretrained weights, supply the computation. We have separated the substrate from its usual language-generation workflow; the experiment does not remove the transformer operations or their computational cost.

This distinction makes the opportunity concrete. We can change how the same neural body is driven, how its response feeds the next invocation, and how programs access the resulting activity.

The method goes beyond recording activations. It acquires return connections from measured neural response geometry, changes the recurrent computation, and evaluates whether the change is worth retaining. A shared observation interface is acquired from paired trajectories so that unchanged readout programs can continue to consume the reorganized computation.

Our central contribution is the acquired pair and the learning-and-reuse cycle around it:

> Acquire an executable reorganization of frozen pretrained computation together with a shared interface for existing programs; learn which pair to retain from experience; then validate improvement and backward compatibility on fresh inputs.

The retained object is the **execution change plus its compatibility interface**. The quality of the new computation and the usability of the old programs are tested together.

This builds on established work in [frozen transformer reuse](https://arxiv.org/abs/2103.05247), [learned execution morphology](https://arxiv.org/abs/2610.38382), and [backward-compatible representations](https://arxiv.org/abs/2206.03040). Our contribution is the native-response-derived execution/interface construction and its closed acquisition, selection and fresh-reuse protocol, with quality, retention and costs evaluated together.

## The Checkpoint 04 result

In Checkpoint 04, experiential feedback selected five update/interface pairs across twelve runtime contexts. Seven contexts retained the baseline. All choices were locked before two untouched validation streams; no pretrained weights, readout programs or observation interfaces were refitted during that final validation.

Two contexts met the strict joint criterion: at least 1% quality improvement on **both** fresh streams, with every eligible old program retained.

| Frozen model and context | Fresh-stream quality gains | Old programs with the shared interface | Direct reuse without the interface |
|:--|:--|:--|:--|
| 0.6B / 111005 | **+3.95%, +4.39%** | **13/13** | 9/13 |
| 1.7B / 111004 | **+5.09%, +4.96%** | **18/18** | 10/18 |

These percentages measure a finite **nonlinear temporal-processing score**: how well linear readouts recover nonlinear relationships across input history. The assay has 143 functions, including 109 multi-lag nonlinear targets. They are benchmark processing-quality gains, rather than percentages of chatbot intelligence or customer revenue.

<figure>
<img src="/assets/images/shaping-gap-checkpoint04.png" alt="Learning-and-reuse cycle, two strict success contexts, and old-program retention across all twelve contexts." width="1430" height="748" loading="lazy">
<figcaption>Two strict fresh-reuse successes, alongside retention for the full twelve-context cohort. The complete paper and data include every selected update, fallback and regression.</figcaption>
</figure>

Across **all twelve contexts**, pooled mean score changes were positive in both fresh streams, and **140 of 141 old program instances remained usable**, versus 116 without the shared interfaces. Four selected updates improved quality on both streams with full retention; two passed the stricter 1% threshold in both. One selected update regressed and lost a program. The smaller-model aggregate was mixed across the streams.

The positive result is learning and reuse within experienced runtime contexts. A separate predictor tested on unseen runtime geometry missed its aggregate transfer criterion. The selected updates also did not cross the stronger newly reliable nonlinear-function threshold across all four feedback and validation blocks. These distinctions are part of the [complete results](https://github.com/calysteon/shaping-gap-benchmark/blob/61fd80fbe76b83a87d465c5dea046fca65637834/checkpoints/04-experience-acquired-reuse/results/report/FEEDBACK-RESULTS.txt).

All neural execution was local, using ordinary four-bit pretrained models on an Apple M4 Max with 36 GiB of memory. Independent checks reproduced **5,040 saved native outputs bit exactly**, alongside 1,008 score vectors and 387 acquisition/retention seals. The publication includes the source, frozen programs, trajectories, protocols and replay commands.

## A different unit of commercial progress

The commercial opportunity follows from the separation: an organization could develop useful new execution over an installed pretrained core, while keeping the programs built on that core usable. The unit of improvement becomes a retained executable construction and its interface.

A model is often one component in a much larger installed system. A better representation can still be an expensive upgrade if every dependent consumer has to be retrained or recalibrated.

This is an established industrial problem. [Learning Backward Compatible Embeddings](https://arxiv.org/abs/2206.03040), from researchers at Amazon and Stanford, describes how embedding updates can force consumer teams to retrain their models or continue maintaining historical versions. The study evaluates compatibility on a real-world recommendation application.

Our commercial research hypothesis is that **capability acquisition and compatibility should be engineered as one upgrade process**.

That could create three kinds of value:

- **More useful work from an installed deployment.** Acquire additional processing ability while retaining the existing pretrained core and measuring the added runtime resources.
- **Lower upgrade and integration costs.** Keep existing consumers usable as the runtime improves, reducing the downstream work required to adopt an improvement.
- **Reusable capability packages.** Retain, version and deploy validated execution/interface constructions for particular tasks or operating contexts.

The product direction would be a capability-development runtime for organizations that control their neural deployments: an engine that discovers candidate constructions, evaluates their gains and compatibility, and maintains accepted improvements. Commercial viability would depend on the resulting workload economics, reliability and maintenance effort.

Checkpoint 06 supplies an initial chronological energy-forecast transfer test. The next application-directed acquisition can improve one forecast horizon while protecting another, then measure whether the resulting quality and lifecycle economics justify deployment.

## The sustainability opportunity

The physical stakes are substantial. The IEA's [2026 energy-and-AI outlook](https://www.iea.org/reports/key-questions-on-energy-and-ai/executive-summary) projects data-centre electricity demand rising from about 485 TWh in 2025 to 950 TWh in 2030. That projection covers data centres broadly, with AI-focused demand growing particularly quickly.

Our long-term objective is **capability growth through acquired organization over an existing substrate**. If successive improvements can be retained and their discovery costs amortized, some capability growth could come from reorganizing installed computation without requiring a larger pretrained model at every step.

That would provide another lever for sustainable AI: extend the useful work delivered by existing hardware, reuse the investment embodied in pretraining, and reduce the upgrade pressure that can accompany new capability demands.

The environmental claim has to be measured over the lifecycle. Unchanged model weights are not an electricity measurement. Discovery, unsuccessful trials, validation, extra execution, storage and maintenance all consume resources. Checkpoint 04 records **1,852,268 new two-position native sequences**, including independent verification, with historical acquisition and an additional 7,680 publication-validation sequences recorded separately. Electrical energy has not been measured in these studies. The complete [Checkpoint 05 ledger](https://github.com/calysteon/shaping-gap-benchmark/blob/04870c5ff60838dde856e03687524721910cda5c/checkpoints/05-component-learning-cycle/results/cost-ledger.json) separately records its component execution, preparation and validation work. The [Checkpoint 06 ledger](https://github.com/calysteon/shaping-gap-benchmark/blob/734af7274a691fdffc3fe4ffe41c2b433d352f52/checkpoints/06-cumulative-contract-interfaces/results/cost-ledger.json) adds recruitment, interface acquisition, failed trials and application transfer. Measured lifetime energy remains the sustainability criterion.

The commercial and sustainability target is therefore precise: **lower lifetime cost and energy per retained useful capability**, at the required quality and regression limits. Deployment volume matters too; a lower cost per capability does not by itself determine the total footprint as usage expands.

## Could a smaller substrate outperform a larger model?

That is a compelling hypothesis to test: a smaller pretrained body with an acquired execution system could outperform a larger model on specified capabilities. Model size and the quality of its execution system are different experimental variables.

The current result establishes improvement relative to the same frozen substrate under the benchmark's baseline runtime. The smaller-versus-larger question calls for its own comparison: matched tasks and quality criteria, with acquisition work, inference work, memory and energy accounted for on both sides.

Checkpoint 06 establishes a second compatible neural gain and keeps shared compatibility in a fixed-size map. The next cumulative threshold is repeated successful acquisition beyond that second gain, including replacement when the active bank fills, with discovery, rejected trials and maintenance charged throughout.

For stronger commercial evidence, the next acquisition must target the real workload directly, improve competitive task quality, and preserve established consumers. Peak memory, latency, energy and total acquisition cost then determine whether useful upgrades pay back against replacement and adaptation alternatives.

This is where the larger opportunity lives: **an AI installation whose useful capabilities can develop through experience, with retained improvements that make later development more valuable.** The model we trained becomes a substrate for further invention through execution.

The research question is whether that process can repeatedly increase what installed computation can do, while keeping the resource budget and existing obligations explicit. The public benchmark now includes first cycles at whole-body and component levels, a second compatible component acquisition, and an application test that defines the next quality and regression contracts.

If you operate a neural workload with expensive upgrades, established downstream consumers and measurable acceptance criteria, I would be interested in exploring that next experiment together.

## Inspect the work

- [Checkpoint 06: complete code, data and reproduction](https://github.com/calysteon/shaping-gap-benchmark/tree/734af7274a691fdffc3fe4ffe41c2b433d352f52/checkpoints/06-cumulative-contract-interfaces)
- [Paper VI: A Second Compatible Neural Acquisition Through Program-Contract Interfaces](https://github.com/calysteon/shaping-gap-benchmark/blob/734af7274a691fdffc3fe4ffe41c2b433d352f52/checkpoints/06-cumulative-contract-interfaces/paper/shaping-gap-benchmark-vi.pdf)
- [Checkpoint 06: complete findings and controls](https://github.com/calysteon/shaping-gap-benchmark/blob/734af7274a691fdffc3fe4ffe41c2b433d352f52/checkpoints/06-cumulative-contract-interfaces/results/FINDINGS.txt)
- [Checkpoint 05: component runtime, complete data and reproduction](https://github.com/calysteon/shaping-gap-benchmark/tree/04870c5ff60838dde856e03687524721910cda5c/checkpoints/05-component-learning-cycle)
- [Paper V: Learned Component Execution Beyond the Transformer Graph](https://github.com/calysteon/shaping-gap-benchmark/blob/04870c5ff60838dde856e03687524721910cda5c/checkpoints/05-component-learning-cycle/paper/shaping-gap-benchmark-v.pdf)
- [Checkpoint 05: all results and controls](https://github.com/calysteon/shaping-gap-benchmark/blob/04870c5ff60838dde856e03687524721910cda5c/checkpoints/05-component-learning-cycle/results/FINDINGS.txt)
- [Current benchmark and research history](https://github.com/calysteon/shaping-gap-benchmark)
- [Checkpoint 04: code, data, protocols and reproduction](https://github.com/calysteon/shaping-gap-benchmark/tree/61fd80fbe76b83a87d465c5dea046fca65637834/checkpoints/04-experience-acquired-reuse)
- [Paper IV: Experience-Acquired Execution and Compatibility Reuse](https://github.com/calysteon/shaping-gap-benchmark/blob/61fd80fbe76b83a87d465c5dea046fca65637834/checkpoints/04-experience-acquired-reuse/paper/shaping-gap-benchmark-iv.pdf)
- [All-context results](https://github.com/calysteon/shaping-gap-benchmark/blob/61fd80fbe76b83a87d465c5dea046fca65637834/checkpoints/04-experience-acquired-reuse/results/report/FEEDBACK-RESULTS.txt)
- [Acquisition and lifecycle cost ledger](https://github.com/calysteon/shaping-gap-benchmark/blob/61fd80fbe76b83a87d465c5dea046fca65637834/checkpoints/04-experience-acquired-reuse/results/cost-ledger.json)

*Research by Nathaniel Oh and Paul Attie. This post describes a research result and a commercial development hypothesis; the linked artifacts provide the precise experimental scope.*

## Change log

### November 13, 2026 — Checkpoint 06

- Added the second compatible neural acquisition, program-contract interface and matched controls.
- Included bounded recruitment scaling, the failed cumulative baseline, complete lifecycle accounting and independent/fresh reproduction.
- Added chronological energy-forecast transfer, its short-horizon regression and stronger classical reference; identified application-directed acquisition as the next threshold.
- Updated the latest paper/code links, homepage excerpt and social preview; preserved earlier checkpoint evidence.
- Research version: [Checkpoint 06 publication](https://github.com/calysteon/shaping-gap-benchmark/commit/734af7274a691fdffc3fe4ffe41c2b433d352f52).


### November 13, 2026 — Checkpoint 05

- Added the component-level learning-and-reuse experiment, its fresh results and all controls.
- Updated the latest paper/code links and social preview. Retained the dated Checkpoint 04 evidence and its original links.
- Documented numerical fidelity and preparation costs; clarified the next acquisition and energy-measurement questions.
- Research version: [Checkpoint 05 publication](https://github.com/calysteon/shaping-gap-benchmark/commit/04870c5ff60838dde856e03687524721910cda5c).

### November 13, 2026 — Initial publication

- Published the explanation of pretrained computation, the Checkpoint 04 result, and its commercial and sustainability research direction.
- Original article version: [de62ced](https://github.com/calysteon/calysteon.github.io/commit/de62ceddf106e75d7fafd1b5aa5d9e5a0e3e89cb).
