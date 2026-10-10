---
layout: post
title: "DNA32: Building a DNA Origami Coding Agent"
---

![Target shape to verified DNA origami design](/images/2026-08-26/cadnagentic_gallery.png)

Earlier this month I was in Fayetteville, Arkansas for [DNA32](https://isnsce.org/dna32-august-3-7-2026/). I gave a talk on the agent harness I've been developing for DNA origami design. The whole thing was sort of spontaneous. I had the idea something like 10 weeks ahead of the submission deadline - January 24, going by the commit history, and that was starting with developing an internal tool-calling agent, which ultimately didn't even work.

I hadn't done a talk in a *long* time. Definitely overloaded the presentation and had to rush at the end, oops. On the plus side, I saw some old labmates from John's (my PhD advisor) lab I hadn't seen in forever. Met some new folk. There was a celebration for John's 75th with John giving a special talk (that I unfortunately missed because I'm a corporate drone now). All in all, very fun for the few days I was there. I should write and submit spontaneous papers more often. It's worth the practice.

## So, the DNA origami design agent

*I choose a lazy person to do a hard job, because a lazy person will find an easy way to do it. -Bill Gates*

![The 2x12 rectangle with a cavity, the structure I had edited too many times by hand](/images/2026-08-26/rect_2x12_cavity_template.png)

Yes, but easy jobs qualify too, if you're lazy enough. I made maybe 50 iterations of this shape, which, I'll admit, isn't a lot. But I had an aching premonition that I was going to design another 50, and after that, I wasn't sure if there was another 50 waiting for me.

For one specific structure without much variation, you can parameterize the things you keep changing and throw together a Python script pretty quickly. It'll save some time, but typically only works for your narrow parameterization of one structure. If you expect to update a design only a few times, scripting it might be fun, but not really worth it.

Early on, that was typical - *ad hoc* scripts. Write enough of them, though, and you hit a critical point where you need to consolidate. So after awhile, lots of groups started looking for more general desig principles, and identifying archetypal structures (polyhedral, curved 3D). This meant writing what was essentially a light CAD program.

But as the variety of archetypal DNA nanostructures has grown, I'm not sure people realize that the goalposts have already moved. Labs now habitually release their own in-house software with a frequency like the *ad hoc* scripts of before. We've reached another critical point.

## So, agents

To me, agents are the next step after a CAD program per archetype. Instead of baking one archetype's rules into one more tool, an agent can carry design principles across archetypes. Over the course of a couple weeks (it's a bit ridiculous how fast things move now) Claude was generating design files from text descriptions, or editing them, from plain text commands.

![From failure to autonomous design, and parametric cavity variants](/images/2026-08-26/cadnagentic_abstract.png)
*Top: before and after I gave it a template. Bottom: 20/30/40 nm cavities, before and after relaxing in oxDNA.*

As we scaled, emerging problems led to more engineering. Memory only helped so much, so next was a harness, with evals to actually keep the drift in check. And then the harness needed engineering of its own.

As a final curiosity before the conference, I set up a loop engineering pipeline to see if it could rediscover optimization rules that already exist, like the staple-breaking rules from [Autobreak](https://doi.org/10.1073/pnas.2406769121). The real follow-up results will be whether it can discover new ones, like what the frontier AI labs are supposedly doing with math proofs (although I don't 100% believe the hype.)

Unfortunately, I'm not part of a lab anymore. Some folks expressed interest at the conference, but to really see this work end-to-end, the designs have to go into the wet lab, and I'm not in a setting to do that these days. Would be great to have that happen. Nothing's in the works yet, but who knows what happens spontaneously now.

## Links

- Preprint: [https://www.biorxiv.org/content/10.64898/2026.04.11.717962v1](https://www.biorxiv.org/content/10.64898/2026.04.11.717962v1) [*This needs to get updated*]
- caDNAgentic: [https://github.com/dfu99/caDNAgentic](https://github.com/dfu99/caDNAgentic)
- DNA32: [https://isnsce.org/dna32-august-3-7-2026/](https://isnsce.org/dna32-august-3-7-2026/)
