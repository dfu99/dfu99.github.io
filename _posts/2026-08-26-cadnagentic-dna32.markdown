---
layout: post
title: "DNA32: What a Coding Agent Can't Learn From Source Code"
---

![Target shape to verified DNA origami design](/images/2026-08-26/cadnagentic_gallery.png)

Earlier this month we were in Fayetteville for [DNA32](https://isnsce.org/dna32-august-3-7-2026/) (August 3–7, University of Arkansas). The Track B talk was *Coding Agents as a Mechanism for Formalizing and Transferring Domain Knowledge in DNA Origami Design*, with Yonggang Ke.

## The fragmentation problem

There is no shortage of DNA origami design software. caDNAno, DAEDALUS, PERDIX, TALOS, DNAxiS, autobreak, tacoxDNA, oxDNA — each is open source and individually capable. What is missing is continuity. Every tool addresses one shape class or one subtask through its own interface, and the rules for composing them correctly aren't written down in any of the codebases. They live in tutorials, lab conventions, and people's heads.

The practical consequence is adoption friction: most designers default to the single environment they already know, even when a better tool exists for the task.

## Tool-calling: insufficient

The first architecture embedded an LLM in the caDNAno GUI as a tool-caller: the user types an instruction, the model parses it and calls predefined methods that modify the design. The premise was that origami manipulation decomposes into a sufficient set of primitives — identify crossovers, move them, place strands, break staples — which the model then composes.

Multi-step scaffold routing achieved **0% success across GPT-class models**. Every primitive needed contextual parameters the schema couldn't carry: half crossover or full crossover, left edge or right edge or interior, 5′→3′ or 3′→5′ on the target helix. Encoding those distinctions into the API surface produced a combinatorial expansion of tool variants without improving task success. Reinforcement learning with a design verifier as the reward signal also failed: a local model (Qwen 1.7B) did not discover correct action sequences by exploration, and no successful GPT-class trajectories existed to distill from.

A predefined tool API encodes necessary but insufficient knowledge. The model can see the available actions but not the reasoning that governs when to compose them.

## Code generation: two capabilities the API cannot express

A coding agent that reads caDNAno's source and writes Python provides two things the tool-calling architecture did not.

**Discovery.** caDNAno's public `createXover` splits strands, which fragments the scaffold. The agent read the strand model internals and found `setConnection3p/5p`, which preserves connectivity. That method isn't in any documentation.

**Composition.** The agent integrated caDNAno, tacoxDNA and oxDNA into one automated pipeline by reading each tool's source for its format requirements. The manual equivalent requires navigating three separate interfaces, file formats and documentation sets.

## Source code is not domain knowledge

The agent fails at exactly the points where a rule is required and absent from the codebase, and each failure exposes something an experienced designer applies without noticing.

**"2-layer" means two grid rows.** Honeycomb packing puts a single row of helices at two distinct y-coordinates, because the vertices of a hexagon don't sit on one horizontal line. Asked for a 2-layer rectangle, the agent read that as four rows and built a 4×7 grid. It then generated a multiple-choice diagram of 1-, 2-, 3- and 4-layer cross-sections and asked the designer to identify the correct one. Labeled once, the error did not recur in any subsequent session, because the answer went into the pipeline as a constraint rather than into a chat log.

**Unprompted geometric verification.** caDNAno's 2D routing view tells you nothing about the 3D conformation. Asked to check that a design was actually flat, the agent converted candidates to 3D coordinates through tacoxDNA, ran PCA on the nucleotide positions, and scored cross-sectional circularity — flat sheet ≈ 0.05, an incorrect 2×3 grid ≈ 0.84 — and rejected incorrect helix arrangements without human feedback. It was asked to verify flatness, not told how.

The same pattern held for scaffold routing (the single-midseam pattern that yields one continuous scaffold oligo), crossover parity, and cavity orientation: correct once, formalize, and the rule persists.

![From failure to autonomous design, and parametric cavity variants](/images/2026-08-26/cadnagentic_abstract.png)
*(A) Early agent output, user-provided template, and autonomous 2-by-22 output. (B) Cavity variants at 20, 30 and 40 nm — tacoxDNA conversion on top, oxDNA-relaxed below.*

The proof of concept: 2-layer rectangles with tunable cavities at 20, 30 and 40 nm, generated parametrically and taken end to end through stapling, format conversion and MD relaxation with no user interaction.

## Scaling the pipeline

Once the tools sit behind one interface, the agent can compose techniques no single tool supports — splicing a DNAxiS curved shell onto a DAEDALUS faceted mesh in one scaffold — and it can simulate the whole library rather than the handful a person would get through by hand.

Running that backlog through oxDNA gives a design rule: **topology beats technique**. Over 10⁶ steps, a curved single shell sheds essentially none of its staples (n = 22); a two-body splice sheds ~24% (n = 25); a multi-body splice ~32% (n = 47). The seam is the instability locus, not the technique that built either half.

## Links

- Preprint: [https://www.biorxiv.org/content/10.64898/2026.04.11.717962v1](https://www.biorxiv.org/content/10.64898/2026.04.11.717962v1)
- caDNAgentic: [https://github.com/dfu99/caDNAgentic](https://github.com/dfu99/caDNAgentic)
- DNA32: [https://isnsce.org/dna32-august-3-7-2026/](https://isnsce.org/dna32-august-3-7-2026/)
