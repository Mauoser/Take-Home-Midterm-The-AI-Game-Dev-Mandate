# When the Lines Get Blurred
### Why Generative AI Fails at 16×16 Pixel Art and What to Do Instead

**CSYE 7270: Virtual Environments and Real-Time 3D — Take-Home Midterm — Lei Zhang**  
Category C — AI Tools Evaluation for Game Dev

Video walkthrough: https://youtu.be/vGs0m1Aqde0

---

> **Topic Claim:** After reading this piece, a practitioner will understand why general-purpose image generation models architecturally cannot produce reliable 16×16 pixel art well enough to choose the right pipeline for sprite production — without making the mistake of treating DALL-E or Gemini as pixel-accurate tools and shipping blurry, inconsistent assets into a Unity game.

---

## The Argument

![alt text](https://github.com/Mauoser/Take-Home-Midterm-The-AI-Game-Dev-Mandate/blob/main/output/05_full_pipeline_comparison.png?raw=true)

The blur you see when importing an AI-generated sprite into Unity is not one problem. It is **three distinct failures** — wrong model category, wrong Unity import settings, wrong sub-tool selection — that produce **identical symptoms** and require **different fixes**.

Iterating on your prompt fixes none of them. The problem is not the prompt. It is architectural.

This repository contains the technical essay, demo notebook, and author's note that demonstrate why.

---

## Repository Structure

```
.
├── essay.md                        # Technical essay (~2,200 words)
├── authors_note.md                 # 3-page reflection (design choices, tool usage, self-assessment)
├── pixel_art_pipeline_demo.ipynb   # Jupyter notebook — runnable demo
└── output/                         # Generated figures (produced by running the notebook)
    ├── 00_ground_truth.png       # Hand-crafted 8-color ground truth sprite
    ├── 01_failure_A.png                # Failure Mode 1: DALL-E + bilinear downscale
    ├── 02_failure_B.png                # Failure Mode 2: PixelLab 24×24 + manual scale
    ├── 02c_bilinear_annotated_grid.png # Figure 2: Bilinear interpolation 2×2 grid diagram
    ├── 03_failure_C_unity_settings.png # Failure Mode 3: correct sprite, wrong Unity import
    ├── 04_human_decision_node.png      # Human Decision Node: ACCEPT/REJECT scale ratio chart
    └── 05_full_pipeline_comparison.png # Main figure: all three failures vs. correct pipeline
```

---

## Quickstart

**Prerequisites:** Python 3.8+, Jupyter

```bash
pip install Pillow matplotlib numpy
jupyter notebook pixel_art_pipeline_demo.ipynb
```

No API keys required. Run cells top to bottom.

---

## The Demo

The notebook demonstrates all three failure modes programmatically using PIL — the same interpolation algorithms that DALL-E, Gemini, and Unity's texture importer use in production.

### What each cell produces

| Cell | What it does | Key output |
|---|---|---|
| 0 | Setup and display helpers | — |
| 1 | Builds ground-truth 16×16 sprite (8-color palette) | `00_ground_truth_16x16.png` |
| 2 | **Failure A** — DALL-E simulation → bilinear downscale | 8 → 142 colors (17.8× bleed) |
| 2b | Bilinear interpolation 2×2 grid diagram (Figure 2) | `02c_bilinear_annotated_grid.png` |
| 3 | **Failure B** — PixelLab 24×24 → manual scale-down | 24/16 = non-integer ratio |
| 4 | **Failure C** — correct sprite → wrong Unity import | Point vs. Bilinear filter comparison |
| 5 | **Human Decision Node** | ACCEPT/REJECT chart by scale ratio |
| 6 | Full pipeline comparison (main figure) | `05_full_pipeline_comparison.png` |
| 7 | **The Exercise** — trigger the failure yourself | Change one variable, observe result |
| 8 | Output file audit | Confirms all figures generated |

### The Exercise (Cell 7)

Change this one line and re-run:

```python
RESAMPLE_METHOD = Image.BILINEAR   # ← change to Image.NEAREST to fix
```

| Value | Result |
|---|---|
| `Image.NEAREST` | Point filter — pixel grid **preserved** |
| `Image.BILINEAR` | Unity default — pixel grid **destroyed** |
| `Image.BICUBIC` | Smoother blur — pixel grid **destroyed** |
| `Image.LANCZOS` | Sharpest blur — pixel grid **destroyed** |

Same source sprite. Same import settings. One variable determines whether the grid survives.

---

## The Three Failure Modes

All three produce an identical blurry sprite in Unity's scene view.

```
Failure A  →  DALL-E/Gemini  →  bilinear downscale  →  blurry
Failure B  →  PixelLab Character Creator 24×24  →  bilinear scale to 16×16  →  blurry
Failure C  →  PixelLab 32×32 + correct scale  →  Unity Filter Mode: Bilinear (default)  →  blurry
Correct    →  PixelLab 32×32  →  nearest-neighbor to 16×16  →  manual cleanup  →  Point filter  →  ✓
```

The causal chain for Failure B (the most dangerous, because it looks like the pixel-native pipeline failed):

> PixelLab Character Creator generates at 24×24 (minimum canvas) → scale to 16×16 → 24/16 = 1.5, a non-integer ratio → bilinear interpolation is mathematically unavoidable → discrete grid destroyed → result indistinguishable from Failure A

The fix: generate at 32×32 instead. 32→16 is exactly 2:1 — clean nearest-neighbor, predictable output. Then manually clean up the result in Aseprite, Pixelorama, or Photoshop before importing.

---

## Human Decision Node

```python
# ============================================
# MANDATORY HUMAN DECISION NODE
# AI proposed: Generate at 24×24 using PixelLab Character Creator
#   (24×24 is the minimum canvas; the tool produces valid pixel art here)
# I rejected/modified because: 24→16 is a 1.5:1 non-integer ratio.
#   Any resampling algorithm must interpolate — there is no clean mapping.
#   This introduces artifacts before the manual cleanup stage even begins,
#   requiring significantly more correction work than a clean 2:1 downscale.
# My decision: Generate at 32×32. The 32→16 ratio is exactly 2:1 — every
#   output pixel maps to four source pixels, nearest-neighbor makes clean
#   choices, and the cleanup step starts from a better baseline.
# ============================================
```

**Cell 5 in the notebook produces a chart showing why this decision is architectural:**
- 24×24 → 16×16: ratio = 0.667 → **REJECT** (non-integer, interpolation unavoidable)
- 32×32 → 16×16: ratio = 0.500 → **ACCEPT** (integer divisor, clean nearest-neighbor)
- 16×16 → 16×16: ratio = 1.000 → **ACCEPT** (native, no scaling needed)

---

## PixelLab Sub-Tool Reference

16×16 sits below the reliable working resolution for most PixelLab AI models. The correct pipeline for any 16×16 asset — characters, objects, tilesets, UI elements — is: generate at 32×32 → scale down → manual cleanup → import.

| Sub-tool | Generation size | Use for |
|---|---|---|
| Character Creator | **32×32** | Characters, enemies, NPCs — scale to 16×16, clean up in Aseprite/Pixelorama/Photoshop ✓ |
| Aseprite extension (Create Texture) | **32×32** | Tilesets, objects, UI — same pipeline ✓ |
| Rotate Tool | **32×32** | Animation workflows |
| Skeleton Animation | **32×32** | Animation workflows |

**Why 32×32 over 24×24:** Both require manual scale-down. But 32→16 is a 2:1 integer ratio — clean nearest-neighbor, predictable output, less manual correction. 24→16 is a 1.5:1 non-integer ratio — bilinear interpolation is unavoidable regardless of algorithm, more artifacts to fix before cleanup even begins.

---

## Unity Import Settings

For any pixel art sprite — regardless of generation tool:

| Setting | Wrong (default) | Correct |
|---|---|---|
| Filter Mode | Bilinear | **Point (no filter)** |
| Compression | Default | **None** |
| Pixels Per Unit | 100 | **Match canvas size (e.g. 16)** |

Tested on Unity 2022 LTS and later. The "Point (no filter)" option may appear as "Point" in older versions.

---

## The Transferable Principle

> The diagnostic question — *at which pipeline boundary was the discrete grid lost?* — is the question to ask before touching a prompt or a setting.

This generalizes beyond pixel art. When a language model produces code that passes unit tests but fails in production, the constraint was dropped at the boundary between text output and execution environment. When an AI-generated audio asset sounds correct in a DAW but wrong in-engine, the constraint was dropped at the sample rate conversion step. The prompt is usually the last place the problem lives.

**AI is a pipeline decision, not a magic layer. Where you put it — and how you constrain it — determines what your game becomes.**

---

## Figures

All figures are generated by running the notebook. The main comparison figure:

`output/05_full_pipeline_comparison.png` — three failures and the correct pipeline side by side, at pixel-grid zoom with grid overlay. This is the figure for the essay's Failure Case section and the video Show section.

`output/04_human_decision_node.png` — ACCEPT/REJECT chart by source canvas size. This is the Human Decision Node figure for the video.

---

## Deliverables

| File | Description |
|---|---|
| `essay.md` | Technical essay — five-section structure per CSYE 7270 rubric |
| `authors_note.md` | 3-page Author's Note (design choices / tool usage / self-assessment) |
| `pixel_art_pipeline_demo.ipynb` | Runnable demo — all three failure modes + Human Decision Node |
