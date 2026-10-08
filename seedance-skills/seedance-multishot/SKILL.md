---
name: seedance-multishot
description: Compose ONE Seedance 2.5 prompt carrying several beats as staged events with a Maintain block, so a script's worth of shots is tested in a single generation instead of N separate runs. Use for "multishot prompt", "multiple keyframes in one generation", "test several shots at once", "stringout", "animatic". For a single shot use seedance-single-shot; to hold a character across shots use seedance-consistent-refs.
---

# Seedance multishot

A script plus a set of keyframes, turned into one generation that reads as a rough animatic. The fast way to see whether a sequence *flows* before committing to per-shot runs.

**Load seedance-prompt-grammar first.** Slots, brackets, references and limits live there.

## When this is right

- Early exploration: you want a read on flow, not a finished shot
- Testing several shot ideas at once against one style
- Judging pacing before per-shot generation

Not for hero shots. A multishot pass trades per-shot control for breadth. Winners get re-run individually through seedance-single-shot.

## Inputs

1. **Beat list.** What happens, in order.
2. **Keyframes.** In playback order. Two to six beats is the sweet spot.
3. **Style anchor.** The locked style frame. Attach it as a reference, never describe it in prose.
4. **Length.** How long the whole clip runs.

## Stage by event, not by clock

Timecode scaffolds like `[0:00–0:06 | Image 1]` are treated as guidance, not frame-accurate cuts, and drift about half a second per boundary. ByteDance's own guidance is to stage by event. Do that, and the drift stops being something you tolerate.

```
Generation goal: <the whole clip in one line>

Stage 1: <one main change>. Ends with <clear end state>. Anchor @Image 1.
Stage 2: <one main change>. Ends with <clear end state>. Anchor @Image 2.
Stage 3: <one main change>. Ends with <clear end state>. Anchor @Image 3.

Maintain: <how many people, what they wear, who holds what, spacing>

(<music or ambient bed>)
<sound effects, short and physical>
```

**One change per stage.** A stage holding two events forces the model to choose which one gets the motion budget, and it chooses wrong. If a beat genuinely has two changes, it is two stages.

**End state is the handoff.** The next stage starts from where the last one ended. A vague end state is where sequences fall apart.

**The Maintain block is not optional.** It is the only thing holding continuity across stages. Name the count, the clothing, the props.

## Anchoring keyframes

Each stage names its anchor as `@Image N`, and says what to take and leave, exactly as in seedance-prompt-grammar.

The first image dominates global style. Order so the strongest style anchor leads.

## When the material overflows one generation

Split into sequential generations **sized to the actual remainder, never padded to the maximum.**

- A 35 second script becomes one 30 second generation and one 5 second generation. Not two 30 second ones.
- Split at a stage boundary, never mid-stage.
- Keep the style line, the entity names and the Maintain block identical across parts.
- Part 2 opens on the shot that follows part 1's last stage.
- Label them `part 1 of 2` in the filename. Each part's staging restarts at Stage 1, because the model does not know it is part 2.
- Padding a part to full length just makes the model invent material the script does not have.

## Reviewing the result

A multishot generation is judged for flow, so watch it whole, then find where it broke.

Lay each beat's keyframe above the matching stretch of the generation. That is how you tell whether a stage failed because the prompt was wrong or because the keyframe was.

## What done looks like

- [ ] Every stage holds exactly one change
- [ ] Every stage names a clear end state
- [ ] A Maintain block exists and names count, clothing and props
- [ ] Each stage anchors to a tagged image with an exclusion
- [ ] Audio channels are filled
- [ ] The output was treated as an animatic, not a deliverable

## When to send it back

- A beat cannot be reduced to one change. Back to the script; it is two beats
- Keyframes are mid-motion. Back to keyframes
- More than eight distinct subjects. Back to the script, the model will not hold them
