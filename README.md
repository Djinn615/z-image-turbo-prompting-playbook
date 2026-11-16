# Z-Image Turbo Prompting Playbook

A field guide to prompting Z-Image Turbo, the open-weights image model
(Apache-2.0), written from research and then corrected by production.
Everything here was tested by shipping a real personalized card on a
local ComfyUI stack (AMD Strix Halo, 128 GB unified memory) on
2026-11-16. No cloud generation.

Merged from a sourced web research pass and an independent cross-check.
Where the two disagreed, the sourced pass won and the disagreement is
noted. The final chapter is production rules learned the expensive way:
small text, measured compositing, and QC at zoom.

By Djinn615.

## The one law

The positive prompt is the only steering wheel. Turbo is distilled with
guidance baked in: CFG is locked at 1, the negative channel is zeroed and
inert, and there is no second pass to rescue a weak prompt. Prompt density
is the whole game.

## Locked house defaults (do not improvise away from these)

- 8 steps, CFG 1, res_multistep sampler, simple scheduler, shift 3.0, ~1 MP.
- Never set KSampler CFG to 0 (that ignores the prompt entirely; 0.0 is a
  Diffusers-only value). Never raise CFG above 1: it burns contrast, breaks
  skin, and doubles runtime.
- Never use karras or exponential schedulers at 8 steps: karras gives soft
  orange mush, exponential gives a screen-door blur. Beta is the one
  scheduler worth A/B testing against simple (slightly more texture in
  leather/bronze in a controlled test).
- Steps: 8 to 10 is the sweet spot. 4 is waxy. 12 to 16 adds grain as a
  deliberate texture experiment. 25 turns blotchy and invents objects.
- Samplers: res_multistep, euler, and dpmpp_2m produce nearly the same
  picture. Ancestral/SDE samplers change the picture without improving it.
  LCM runs soft. Sampler choice is taste, not quality.

## Prompt shape

Dense short concrete clauses, not tag soup and not flowing prose. Commas and
periods between phrases are fine; bare keyword lists are not. Structure:

1. Picture type / medium FIRST ("a medium-shot phone selfie photo",
   "vertical digital illustration", "product photograph"). This locks the
   style family before anything else can pull it.
2. Subject + action, with 2 to 3 identifying specifics. Every additional
   person or object gets its OWN clause with its own clothes, colours, props.
3. Subject detail grouped by region: garment, face, hair, held objects.
4. Spatial relations said out loud ("on the left", "above her extended left
   palm", "along the top edge"). Left implicit, layout is ignored.
5. Environment: location plus 1 to 2 supporting elements, foreground /
   middle / background layers. Named places work well.
6. Lighting: direction, quality, colour temperature, time of day. For
   photorealism this is the single strongest lever.
7. Camera/rendering + palette as a compact closing run (shot size, lens,
   aperture, film stock, dominant colours, medium).

Length: 80 to 200 words for a single subject; posters with several text
blocks legitimately run to ~300. Hard ceiling near 504 prompt tokens; if a
prompt must shrink, cut the middle, never the closing attributes. Front-load
whatever must be right.

Syntax rules:
- No weighting: (word:1.3), [brackets], --no flags are plain text to the
  Qwen3 encoder. Emphasis comes from position and from describing a thing
  in more concrete detail.
- No quality meta-tags: "masterpiece, 8k, ultra-detailed, award winning"
  are explicitly forbidden by Tongyi's own Prompt Enhancer. They spend
  attention and buy nothing.
- No metaphors or similes: the model draws them literally ("fawn" the colour
  became a baby deer). Plain visual description only.
- One value per attribute. No "or", "maybe", "various": the model draws
  both or neither.
- English or Chinese prompts only; those are the trained languages.

## Negatives do not exist here

- The negative prompt is mathematically inert (ConditioningZeroOut). Do not
  waste a word in it.
- Writing "no X" INSIDE the positive prompt was measured in a controlled
  test (189 images): it fixed 4 of 54 errors, the same as doing nothing,
  and scored 0 of 18 in a second round. It can backfire: "no robot arms"
  made the robot arms sharper. The model draws negated nouns.
- The fix that works is rewording: never name the unwanted thing; describe
  the desired state positively.

| Avoid naming | Write instead |
|---|---|
| watermark / text | clean unbroken surface, plain untouched background |
| extra people | a single figure alone in an empty room |
| blur | every edge crisp, uniform sharp focus across the frame |
| clutter | bare tabletop, one object centred, wide empty margins |
| cartoon look | photograph, natural skin texture, visible pores, real fabric weave |
| harsh shadows | soft even diffuse light, gentle shadow falloff |
| logos / branding | plain unbranded surface |

## Rendered text (a genuine strength)

1. Quote every exact string in double quotes: "OPEN 8-6". Unquoted text
   garbles or vanishes. The model renders what is quoted, typos included,
   so proofread the quotes.
2. For each string, state placement, relative size, and typeface in words:
   "Massive headline 'TRAIL CUP' across the upper third, heavy geometric
   sans-serif, warm white on deep navy."
3. Reliability zone: up to 3 short strings. 1 to 3 words per element is the
   comfortable range; ALL CAPS helps short labels. Split long copy into
   separate quoted strings, each with its own placement and typeface.
4. English and Chinese render, including mixed in one image. Other scripts
   are unsupported: proof-render, or generate clean and set type in post.
5. Dense multi-line posters do work in demos, but for anything that must be
   letter-perfect, keep strings short, large, and few, and budget seeds.

## Photorealism: the three-layer stack

The default prior is glossy beauty-stock. "Realistic" as an adjective does
nothing. Use all three layers:

1. Name the capture device or film stock: "point-and-shoot film camera",
   "handheld iPhone snapshot", "disposable camera aesthetic, slight
   overexposure", "Leica M6, Kodak Portra 400", "Sony A7R IV, 85mm f/1.4".
   Each maps to a learned visual fingerprint. "Photorealistic, 4K" maps to
   nothing.
2. Break the symmetry with one non-idealised feature: "three-day stubble",
   "slightly crooked nose", "deep smile lines", "thinning hair", "a small
   mole below the left jaw". Skipping this layer is the main cause of the
   plastic look.
3. Name material texture: "visible pores", "fabric weave", "film grain",
   "wet wooden planks", "salt crystals on his jacket".

Plus: specific directional lighting, a concrete lens/aperture for depth,
candid framing language ("candid unstaged photograph", "slight motion
softness"), and for angled shots anchor the gaze on a concrete object away
from the camera ("completely unaware of the camera") or the model tilts the
chin back to eye contact.

Weak: "Realistic photo of an average middle-aged French man in a cafe."
Strong: "Middle-aged French man, thinning hair, three-day stubble, deep
smile lines, worn grey wool coat. Sits at a small marble cafe table, elbow
on the surface, holding a small espresso cup. Half-empty carafe and folded
newspaper beside him. Overcast midday light through a large window, soft
directional shadows from the left. Point-and-shoot film camera, mild grain,
medium depth of field, face in sharp focus, muted warm-grey palette, candid
unstaged photograph."

Portraits/skin: dropping shift from 3 to 2 restored pores at no speed cost
in one controlled single-prompt test. It is a cheap first try for close-up
skin specifically; A/B it per prompt, do not adopt it globally. (The cross-check said
shift should rise with resolution; the sourced tests found shift barely
matters between 2.5 and 4.0. Trust the tests.)

## Weaknesses and what actually works

- Counting: enumerate with distinct attributes: "exactly three: one white,
  one black and one brown" fixed the count 3 of 3. Line subjects up and
  differentiate them. Head counts stay seed-dependent even then, so render
  several seeds and pick. Never write "not four, not two": it made counts
  worse.
- Multi-subject: one clause per subject, own clothes/colours/props,
  explicit left/right or foreground/background anchors, maximally distinct
  descriptions. Attribute bleed and same-face duplication are the failure
  modes; merged bodies were not observed.
- Hands: better than the model's reputation (zero finger errors across 144
  visible-hand images in one study), but complex contact poses still fail.
  Give hands a simple supported state and a concrete action ("resting flat
  on the table", "holding a mug by its handle in her right hand"), keep
  them reasonably large in frame, and never paste SD anatomy boilerplate
  ("perfect anatomy, correct number of fingers"): the negation half is
  inert and the rest is wasted tokens.
- Small/distant faces: a pixel budget problem, not a wording problem.
  Frame closer ("medium close-up", "face in sharp focus"), or fix in the
  workflow with an upscale/detail pass.
- Low diversity: officially rated Low. Re-rolling the seed returns nearly
  the same picture. Vary the prompt instead (one axis per variant: camera,
  light direction, wardrobe, time of day) and move the varied element early.
  Change aspect ratio to force a new composition. Some errors are bound to
  a seed; those are fixed by changing the seed, not by more wording.
- Style priors override wording: anime pulls faces toward East Asian
  features; some styles carry artefacts (a red seal stamp in minhwa).
  Describe visible features directly, fill the space where the artefact
  appears with described content, and accept occasional post-editing.

## Resolution

- Sweet spot is ~1 MP total: 1024x1024, 832x1216, 1216x832, 1344x768 and
  their inverses. Dimensions divisible by 16. Any aspect ratio works;
  aspect is a composition lever, and a taller ratio is the correct fix for
  cropped heads/feet, not more wording.
- Direct generation up to ~1.5 to 2 MP is workable but scene-dependent.
  The reliable route to 2K: generate near 1 MP, then an upscale pass
  (RealESRGAN pre-upscale, re-sample at denoise 0.33 shipped default;
  0.15 to 0.25 stays close to source and suits faces and text; above 0.35
  needs a detailed caption or it invents detail).

## Styles in one line each

- Illustration/anime: medium + technique up front ("cel shading, clean line
  art", "soft watercolor", "screentone shading"). One style family only;
  contradictions blend into mush.
- Product: product + clean backdrop + softbox/studio light + shadow style
  + materials + commercial look. Turbo is strong here.
- Logos/posters: "flat vector, flat solid colours" stated positively,
  exact quoted text, named palette only ("only dark forest green and warm
  cream"), layout and margins described, print idiom named.

## What does NOT transfer from SDXL/FLUX

CFG 6 to 7; long negative prompts; DPM++ 2M Karras at 25 steps; (word:1.3)
emphasis; booru/score tags; seed-rolling for variety; base Z-Image settings
(28 to 50 steps, CFG 3 to 5, real negatives) — a different checkpoint's
recipe. FLUX-style flowing prose is closer but still off: this model's
tuned shape is dense short clauses with photographic terms as the closing
run.

## Sources

- Tongyi-MAI Z-Image README and Prompt Enhancer rules (official):
  github.com/tongyi-mai/z-image
- ComfyUI Z-Image Turbo tutorial (verified live 2026-11-16):
  docs.comfy.org/tutorials/image/z-image/z-image-turbo
- Controlled settings test, 76 images, one variable at a time:
  robertcreative.com/blog/z-image-turbo-best-settings and the same author's
  skin test at /blog/z-image-turbo-realistic-skin
- Negation controlled test, 189 images: Medium, @iamsupcat, "Fixing AI
  Image Errors With Prompts: Z-Image vs Qwen Image 2.1"
- fal Z-Image Turbo prompt guide: fal.ai/learn/devs/z-image-turbo-prompt-guide
- Community skills synthesised from official sources:
  github.com/maciejdzierzek/z-image-prompt-generator,
  github.com/domoarigatomrburato/agent-skills (z-image-prompt)
- Independent cross-check (model knowledge, unverified): agreed on prompt form,
  inert negatives, weighting syntax, counting by enumeration; overruled by
  the sourced tests on steps-for-text, shift-vs-resolution, and the
  "name the opposite" phrasing (positive-state only, never name the flaw).


---

## Chapter: Production rules, learned by shipping a real card

Everything above came from research. This chapter came from building a
personalized birthday card end to end on this stack on 2026-11-16 and
having a human art-director reject the failures. These rules generalize
past Z-Image Turbo.

**1. Small text is never left to the model.** At signature size the
model garbled a fixed wordmark two different ways on two renders
("DIINN615", "DJIN615"). At nameplate size it embossed a 10-character
handle perfectly, three renders out of three. Working rule: the model
renders large display lettering only. Anything small, or anything whose
spelling must be guaranteed, is composited in post, where it is correct
by construction.

**2. Generation is the art layer, not the accuracy layer.** The
pipeline that shipped: generate the scene (plate, embossing, lighting),
composite the personalization deterministically (a profile photo seated
in a rendered porthole), stamp the small text, upscale last
(RealESRGAN 4x). Anything that has to be exactly right is code after
the render, never a more hopeful prompt.

**3. Measure, do not estimate.** Three estimation methods failed in one
afternoon: a bounding box over all dark pixels (caught shadows, disc
rendered at nearly double size), a longest-dark-run fit (a shadow
bridge shifted the center), and per-character text widths (understated
the drawn ink by about 15 percent, so the name ran off its plate).
What passed: find the target region as the largest connected dark
component after erosion, take its centroid and bounding box; and fit
text by drawing it, measuring the real ink bounding box, and rescaling
until it fits the measured band. If a value decides placement, measure
it from the pixels.

**4. QC at zoom, before delivery.** Every failure above was invisible
at full frame and obvious at 200 percent crop. Inspect crops of the
detail regions (text, faces, hands, seating edges) before calling a
render done. A failed check is a silent rerun, not a delivery with an
apology attached.

**5. Formed text beats printed text.** Asking for lettering physically
formed in the material (embossed, engraved, stamped) renders more
convincingly than surface graphics, and it hides the model's text
weaknesses better, because slight irregularity reads as craft. This
became a design doctrine on the project that produced this chapter:
words set in metal, in the Mistborn sense. "I write these words in
steel, for anything not set in metal cannot be trusted."
