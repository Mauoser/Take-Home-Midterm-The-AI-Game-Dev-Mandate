# Author's Note
## *When the Lines Get Blurred: Why Generative AI Fails at 16×16 Pixel Art and What to Do Instead*
CSYE 7270: Virtual Environments and Real-Time 3D — Take-Home Midterm — Lei Zhang

---

## Page 1 — Design Choices

### Why this topic?

This topic chose itself. Three weeks into building a top-down RPG in Unity for another course, I spent an afternoon generating 16×16 knight sprites in DALL-E and Gemini. Every output looked plausible at generation preview size. Every output dissolved in Unity's scene view. The failure was repeatable, consistent, and completely opaque — iterating on prompts made no meaningful difference.

That opacity is what made it a good essay topic. A failure that looks like a prompting problem but is actually an architectural problem is exactly the kind of thing the course's master claim is designed to expose: AI is a pipeline decision, not a magic layer. The 16×16 case is the master claim made visible at the pixel level.

### What I left out and why

The essay does not cover Stable Diffusion LoRA setup in depth. A pixel art LoRA for SD represents a third generation approach alongside PixelLab, and it has its own sub-tool configuration complexity (checkpoint selection, CFG scale, step count). Including it would have required a separate mechanism section and pushed the essay past 2,500 words. PixelLab's Character Creator at 32×32 is the most accessible pixel-native starting point for a solo Unity developer, and it cleanly demonstrates the mechanism. SD LoRA is the correct follow-up topic for a reader who wants to go deeper.

The essay also does not address sprite animation. Skeleton Animation and the Rotate Tool in PixelLab both support generation at 32×32, but they serve animation workflows, not static sprite generation. Including them would have fragmented the essay's focus on the single-sprite production decision that most developers encounter first.

### How the essay instances the master claim

The master claim — AI is a pipeline decision, not a magic layer — is demonstrated through three failure modes that share one symptom and have three different pipeline locations. Each failure mode is the result of a specific pipeline decision (model selection, import settings, sub-tool selection), and each fix is a pipeline correction, not a prompt correction. The essay's exercise makes this concrete: a reader can change a single variable at each step and observe the failure appear or disappear, without touching a prompt. The prompt never changes across all four exercise configurations. The pipeline does.

---

## Page 2 — Tool Usage

### What Bookie generated and where I corrected it

Bookie produced a strong initial draft with two structural problems and one factual error about PixelLab. First, the section order was wrong: Bookie placed "Choosing the Right Pipeline" (the Design Decision section) after "Three Failures That Look Like One" (the Failure Case section). The rubric order is Mechanism → Design Decision → Failure Case → Exercise. Second, Bookie's PixelLab description was generic and incorrect — it described the Character Creator as the wrong sub-tool for 16×16, and suggested the BitForge canvas in Simple Creator as the correct alternative. Testing PixelLab revealed this is wrong: 16×16 is below the reliable working resolution for most of PixelLab's AI models, and the correct pipeline requires generating at 32×32 and scaling down manually.

My corrections:

- Moved the Design Decision section before the Failure Case, restoring the rubric's causal logic: here is how it works → here is what you choose → here is what happens when you choose wrong.
- Replaced Bookie's incorrect PixelLab recommendation entirely. Bookie wrote: "Use PixelLab's Simple Creator (BitForge canvas) which supports native 16×16 generation." My correction: the correct pipeline is Character Creator at 32×32 → scale to 16×16 with nearest-neighbor → manual cleanup in Aseprite. No PixelLab tool reliably produces usable 16×16 output directly.
- Rewrote Failure Mode 3 entirely. Bookie's version framed it as choosing the wrong sub-tool (Character Creator vs. BitForge). The actual failure is skipping the manual cleanup step after a correct 32×32 → 16×16 downscale. The causal chain is: generate at 32×32 (correct) → scale nearest-neighbor to 16×16 (correct) → skip manual pixel editing (failure) → sprite reads as broken in game context.

### What Eddy flagged

Eddy's review was thorough and mostly correct. Five line edits were accepted without modification:

- "fully legible" → "makes sense" (academic register removed)
- Redundant sentence after bilinear formula cut
- "at a glance / at a glance" repetition resolved to single clause with brushwork analogy
- "The team iterates on game mechanics, on level design, on audio" cut (over-explained)
- "honest recommendation" → removed "honest" (verbal hedging tic)

Eddy's Substack-specific recommendations were reviewed individually. The Problem Set was removed after verifying it does not appear in the rubric — Bookie added it because the essay reads like a course chapter, but it is not a required deliverable. The TL;DR block and CTA were also omitted — neither appears in the rubric's required essay structure.

> *Eddy: "The Problem Set at the end, the academic register, the lack of a CTA, the section headers that describe rather than propel — all of it signals 'read this on paper in a course packet,' not 'open this on your phone between commits.'"*

My response: Eddy was correct about the Problem Set — it was removed once I confirmed it is not a rubric requirement. The other Substack recommendations (TL;DR, CTA, alternative headline) were not adopted because the rubric specifies five essay sections and a graded academic format, not a newsletter format. The academic register and original headline were kept.

Eddy did not flag the section order problem (Design Decision placed after Failure Case). This is the highest-risk structural issue for the rubric and I caught it independently.

### What Figure Architect produced that I used

Figure Architect identified six figure zones and ranked them by priority. Zone 3 (Three Failures, One Symptom) was ranked #1 with the language:

> *"The chapter's central claim — three architecturally distinct failures produce identical symptoms — is completely unverifiable without a side-by-side diagnostic figure. A reader cannot confirm that FM1, FM2, FM3 look identical without seeing them side-by-side. This is the highest-value Verification Gap in the text."*

This confirmed that the Jupyter notebook's Cell 6 (full pipeline comparison figure) was the correct computational approach. The notebook produces this figure programmatically, demonstrating the failures rather than describing them. Figure Architect's Zone 6 (Structured Exercise Sequence) was implemented as the notebook's Cell 7.

Zone 4 (Pipeline Boundary Principle abstract diagram) and Zone 5 (Decision Framework) were noted by Figure Architect as Important but deprioritized. The essay's prose and the PixelLab sub-tool table cover this content, and adding these figures would have pushed the deliverable past the word limit. This is documented as a deliberate scope decision, not an oversight.

---

## Page 3 — Self-Assessment

### Rubric scoring

| Criterion | Points | Self-Score |
|---|---|---|
| Argumentative Rigor — specific claim, mechanistic failure case, failure triggered in demo | 35 | 34 / 35 |
| Technical Implementation — real-world messiness, Human Decision Node present, failure reproducible | 25 | 25 / 25 |
| Clarity — five-section structure, every claim has a mechanism, no premature jargon | 20 | 20 / 20 |
| Relative Quality (Top 25%) | 20 | 19 / 20 |
| **TOTAL** | **100** | **98 / 100** |

### Top 25% test — five questions

**1. Can I state the master claim and explain how my essay instances it?**

Yes. The master claim is: AI is a pipeline decision, not a magic layer. The essay instances it by demonstrating that three different pipeline failures — wrong model category, wrong import settings, skipped manual cleanup — each produce the same blurry output and each requires a different pipeline correction. Critically, the correct pipeline includes a mandatory human editing stage: no AI tool reliably produces usable 16×16 output directly, so the human cleanup step is not optional polish but a required pipeline decision.

**2. Does my essay name a failure mode, trace the causal chain, and show it triggering?**

Yes, for all three failure modes. Failure Mode 3's causal chain is explicit: generate at 32×32 (correct) → scale to 16×16 nearest-neighbor (correct) → skip manual pixel editing (failure) → sprite reads as broken in game context. Failure Mode 1's causal chain is quantified in the notebook: 8 colors in, 142 colors out — a 17.8× color explosion from bilinear averaging of ~1,024 source pixels per output pixel.

**3. Is there a visible Human Decision Node — in the demo and on camera?**

Yes. The Human Decision Node documents the choice of 32×32 over 24×24 as the generation target. The reasoning is architectural: 32→16 is a 2:1 integer ratio — every output pixel maps to exactly four source pixels, and nearest-neighbor makes clean, predictable choices. 24→16 is a 1.5:1 non-integer ratio — interpolation is unavoidable, introducing artifacts before the manual cleanup stage even begins. The notebook's Cell 5 produces a decision summary chart showing ACCEPT/REJECT by scale ratio. This is the 30-second camera moment in the video.

**4. Does every section have the scenario, mechanism, decision, and failure?**

Yes. The Scenario names the game type (top-down RPG), pipeline stage (sprite generation), and creative problem (no pixel artist). The Mechanism traces from training corpus through latent space to bilinear downscale with the formula. The Design Decision gives the PixelLab sub-tool table and Unity import settings. The Failure Case gives three distinct failure modes with causal chains.

**5. Can a classmate reproduce the failure mode from a fresh clone?**

Yes. The Exercise section gives four step-by-step configurations requiring only DALL-E (free tier), PixelLab (free tier), Unity (student license), and Aseprite (free). The notebook automates all three failure modes using only PIL and matplotlib — no API keys required. The deliberate trigger in the notebook's Cell 7 is a single variable change: `RESAMPLE_METHOD = Image.BILINEAR` → `Image.NEAREST`. A classmate can reproduce the failure mode in under two minutes from a fresh clone.

### What my failure case does and does not demonstrate

**Does demonstrate:** that the same visual symptom (blurry or low-quality 16×16 sprite) has three distinct root causes at three different pipeline stages, and that fixing one stage does not fix the others. The notebook's color count measurement (8 → 142) provides a quantitative, not just visual, measure of grid destruction for Failure Mode 1.

**Does not demonstrate:** the manual cleanup step in a live workflow. The notebook simulates the generation and downscale stages programmatically, but cannot simulate the judgment a developer makes when editing pixels in Aseprite. Failure Mode 3 — the skipped cleanup failure — is described and its upstream conditions are demonstrated in the notebook, but the cleanup work itself is a human action that exists outside the automated demo. This is an honest limitation: the most important stage in the correct pipeline is the one the notebook cannot run.
