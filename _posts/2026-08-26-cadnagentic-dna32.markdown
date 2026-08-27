---
layout: post
title: "DNA32: What a Coding Agent Can't Learn From Source Code"
---

![Target shape to verified DNA origami design](/images/2026-08-26/cadnagentic_gallery.png)

Earlier this month we were in Fayetteville for [DNA32](https://isnsce.org/dna32-august-3-7-2026/) (August 3–7, University of Arkansas). The Track B talk was *Coding Agents as a Mechanism for Formalizing and Transferring Domain Knowledge in DNA Origami Design*, with Yonggang Ke.

## The problem is not the tools

There is no shortage of DNA origami design software. caDNAno, DAEDALUS, PERDIX, TALOS, DNAxiS, autobreak, tacoxDNA, oxDNA — each one is open source, and each one is individually fine. What's missing is continuity. Every tool addresses one shape class or one subtask through its own interface, and the rules for composing them correctly aren't written down in any of the codebases. They live in tutorials, lab conventions, and people's heads.

The practical consequence is adoption friction. Most designers default to the single environment they already know, even when a better tool exists for the task in front of them. That's the bottleneck I wanted to attack.

## The first architecture didn't work

The obvious move is to embed an LLM in the caDNAno GUI as a tool-caller: the user types an instruction, the model parses it and calls predefined methods that modify the design. Decompose origami manipulation into enough primitives — identify crossovers, move them, place strands, break staples — and the model composes them.

It doesn't. Scaffold routing hit **0% success across GPT-class models**. Every primitive needed contextual parameters the schema couldn't carry: half crossover or full crossover, left edge or right edge or interior, 5′→3′ or 3′→5′ on the target helix. Encoding those distinctions into the API surface just produced a combinatorial explosion of tool variants without moving success at all. Reinforcement learning with a design verifier as the reward signal didn't rescue it either — a local model (Qwen 1.7B) never discovered a correct action sequence by exploration, and there were no successful GPT-class trajectories to distill from in the first place.

A predefined tool API encodes necessary but insufficient knowledge. The model can see the available actions but not the reasoning that governs when to compose them.

## Source code access buys two things

Swapping to a coding agent that reads caDNAno's source and writes Python got two capabilities the tool API couldn't express.

**Discovery.** caDNAno's public `createXover` splits strands, which fragments the scaffold. The agent read the strand model internals and found `setConnection3p/5p`, which preserves connectivity. That method isn't in any documentation.

**Composition.** It wired caDNAno → tacoxDNA → oxDNA into one automated pipeline by reading each tool's source for its format requirements, rather than by my navigating three interfaces and two file formats by hand.

## But source code is not domain knowledge

This is the part of the talk I care about. The agent fails at exactly the points where a rule is required and absent from the codebase, and each failure exposes something an experienced designer applies without noticing.

**"2-layer" means two grid rows.** Honeycomb packing puts a single row of helices at two distinct y-coordinates, because the vertices of a hexagon don't sit on one horizontal line. Asked for a 2-layer rectangle, the agent read that as four rows and built a 4×7 grid. What it did next is the thing I keep repeating to people: it drew its own multiple-choice diagram of 1-, 2-, 3- and 4-layer cross-sections and asked me to point at the right one. I labeled it once. It never made that error again in any subsequent session, because the answer went into the pipeline as a constraint instead of into a chat log.

**It invented its own verification.** caDNAno's 2D routing view tells you nothing about the 3D conformation. Asked to check that a design was actually flat, the agent converted candidates to 3D coordinates through tacoxDNA, ran PCA on the nucleotide positions, and scored cross-sectional circularity — flat sheet ≈ 0.05, an incorrect 2×3 grid ≈ 0.84 — and rejected wrong helix arrangements without asking me anything. It was told to verify flatness, not how to.

The same pattern ran through scaffold routing (the single-midseam pattern that yields one continuous scaffold oligo), crossover parity, and cavity orientation. Correct once, formalize, move on.

![From failure to autonomous design, and parametric cavity variants](/images/2026-08-26/cadnagentic_abstract.png)
*(A) Early agent output, user-provided template, and autonomous 2-by-22 output. (B) Cavity variants at 20, 30 and 40 nm — tacoxDNA conversion on top, oxDNA-relaxed below.*

The proof of concept: 2-layer rectangles with tunable cavities at 20, 30 and 40 nm, generated parametrically and taken end to end through stapling, format conversion and MD relaxation with no user interaction.

## The part I got wrong

My first draft of the talk was science-first — here are the structures, here is the result. Wrong subject. The structures are *evidence*; the contribution is the mechanism. A correction only becomes transferable when it lands as an artifact — a verifier function, a parametric script, a documented failure catalog — instead of as a conversation that ends when the session does. Rebuilding the deck around that thesis took one pass, mostly because the slides were generated by a script rather than clicked together.

## What came after the abstract

Scale, which is the other half of the argument. Once the tools sit behind one interface, the agent can compose techniques no single tool supports — splicing a DNAxiS curved shell onto a DAEDALUS faceted mesh in one scaffold — and it can simulate the whole library instead of the handful a person would get through.

Running that backlog through oxDNA turned up a design rule I didn't go looking for: **topology beats technique**. A curved single shell sheds essentially none of its staples over 10⁶ steps; a two-body splice sheds ~24%; a multi-body splice ~32%. The seam is the instability locus, not the technique that built either half. That one probably deserves its own post.

## Links

- Preprint: [https://www.biorxiv.org/content/10.64898/2026.04.11.717962v1](https://www.biorxiv.org/content/10.64898/2026.04.11.717962v1)
- caDNAgentic: [https://github.com/dfu99/caDNAgentic](https://github.com/dfu99/caDNAgentic)
- DNA32: [https://isnsce.org/dna32-august-3-7-2026/](https://isnsce.org/dna32-august-3-7-2026/)
