---
name: seedance-prompt-grammar
description: The Seedance 2.5 prompt specification - six slots in a fixed order, four bracket channels that route music, sound effects, dialogue and subtitles, @-tagged references with exclusions, input limits, and the parameters that lock themselves per mode. Load this before writing any Seedance 2.5 prompt. Reference only, not a workflow; the workflows are seedance-single-shot, seedance-multishot, seedance-consistent-refs and seedance-guide.
---

# Seedance 2.5 prompt grammar

ByteDance published an actual prompt specification with Seedance 2.5. This matters more than it sounds. Before 2.5, everyone invented their own conventions because there were none. Now there is a grammar, and prompts that ignore it lose channels the model is listening on.

This page is the single copy. The other Seedance skills in this pack load it rather than restating it, so it cannot drift.

> **Read this first.** This is reconstructed from published write-ups of ByteDance's Seedance prompt guide plus our own generations. Rules are marked **confirmed** (generated with it and seen it work) or **unverified** (believed, not yet tested). Do not treat unverified rules as settled, and do not rewrite a whole prompt library around them until a test clip comes back.

## The six slots, in this order

| # | Slot | Required | What goes in |
|---|---|---|---|
| 1 | Subject | yes | Who or what. Name it, do not describe it vaguely |
| 2 | Action or event | yes | The one thing that happens |
| 3 | Scene and environment | no | Where, and the light |
| 4 | Visual style | no | The look, if a reference is not carrying it |
| 5 | Camera movement or cut | no | How the frame moves |
| 6 | Audio | no | See the bracket channels below |

Only the first two are required. Fill the slots you care about and leave the rest out. Two or three sentences beats a paragraph.

**Order is not decoration.** Write them in this sequence. Moving camera direction above the action changes what the model spends its motion budget on.

## The four bracket channels

Brackets are routing, not punctuation. Each type sends its contents to a different channel.

| Bracket | Goes to | Example |
|---|---|---|
| `( )` | Music and ambient beds | `(low cello drone)` |
| `< >` | Sound effects | `<door latch, rain on glass>` |
| `{ }` | Spoken dialogue | `{We should go.}` |
| `【 】` | On-screen subtitles | `【We should go.】` |

**Dialogue and subtitle are separate channels.** A line you want heard *and* seen goes in both. Putting it in one does not fill the other.

Keep sound effects short and physical. `<boot on gravel>` works. A dense list of per-object foley does not.

Audio generates by default on most Seedance 2.5 interfaces, so an undirected prompt gets generic sound. Fill the channels you care about.

## References are tagged in the prompt

Files are numbered in upload order and called by name inside the prompt text:

```
@Image 1   @Image 2   @Video 1   @Audio 1
```

**Say what to take and what to leave.** This is the part everyone skips and it is the part that fixes reference bleed:

```
Use the jacket and the hair from @Image 2. Do not use its sky or the parked cars.
```

Tie each element to exactly one file. Two files claiming the same element is how identity drifts.

## Limits

- 30 images, counting the first frame and last frame
- 10 videos, 10 audio clips
- 50 reference media items in total
- Reference videos must run **2 to 30 seconds**; anything shorter or longer is rejected outright. Conform short shots first (slow them down with `setpts=PTS*N` in ffmpeg, or loop them)
- Stable range is **1 to 8 distinct subjects**, which counts subjects, not files

The media counts and the video length range are **confirmed**: generation interfaces enforce them and reject the job rather than degrading quietly. The subject range is unverified guidance.

The 50-item ceiling is a ceiling, not a target. More references is not better; more *unambiguous* references is better.

## Parameters that lock themselves

| Mode | Aspect ratio | Duration |
|---|---|---|
| Video editing | Locked to the input | Locked to input, ±0.3 sec |
| First frame or image to video | Locked to the image | Free |
| Video extension | Locked to the input | Free |

If you set an aspect ratio and get something else back, this is why. Check the mode before you argue with the output.

### Modes, and what each one allows

| Mode | Use it for | Constraint |
|---|---|---|
| Text to video | No references at all | Rejects any reference media |
| Reference (omni) | Anything with references | Needs at least one; the mode that accepts a first frame and last frame |
| Video edit | Changing an existing clip | Exactly one video reference |
| Video extension | Continuing a clip | At least one video reference, plus a direction (forward or backward) |

A keyframe goes in as a first frame, which means reference mode, not text to video. That is the mistake to expect.

Resolution is typically 480p, 720p or 1080p, with no 4K. Duration is whole seconds.

## Staging a long clip

For anything past a single beat, stage by **event**, not by clock.

```
Generation goal: <one line, the whole clip>

Stage 1: <one main change> ends with <clear end state>
Stage 2: <one main change> ends with <clear end state>
Stage 3: <one main change> ends with <clear end state>

Maintain: <character count, clothing, who holds which prop, spacing>
```

One change per stage. The moment a stage holds two events the model has to choose which one to spend motion on, and it will choose the wrong one.

The Maintain block is what stops drift between stages. Name the count, the clothes, and who is holding what.

**Why not timecodes.** Scaffolding like `[0:00–0:06 | Image 1]` is treated as strong guidance, not frame-accurate cuts, and we measured roughly half a second of drift per boundary. Event staging says the same thing in the language the model actually follows. (Whole-second ranges still have their place for edits and reference shots; see seedance-guide.)

## What is still unverified

- Whether the six-slot order and the four bracket channels behave exactly as documented. They come from write-ups of ByteDance's guide, and one test clip settles them for your use case.
- Whether `【 】` subtitles render cleanly enough to trust. Until you have tested it, keep on-screen text out of the generation and add graphics in post.

## Craft rules that still apply

These come from real production runs and are not superseded by anything above.

- **Brand colours are law.** If a brand has a palette, use its hex values verbatim in style references, never approximate names.
- **Refs beat prose.** A look lives in the keyframe, not in adjectives. If a segment needs a paragraph of description, the keyframe is wrong.
- **Entity-first.** Name the recurring subject identically every time. Never rotate "the courier" into "he" into "our hero". With a real person on camera, pick one label ("the talent") and keep it.
- **Keyframe stillness.** Feed stable poses, never mid-motion frames.
- **Visual language only.** No strategy words, no brand adjectives, no words a camera cannot see.
- **Cheap pass first.** Explore at low resolution, re-run winners high. An exploratory generation is never the deliverable.
- **Direct the voice.** If anyone speaks, decide the delivery (pace, energy, emotion) before writing the `{ }` line, and keep tone notes off individual words.
