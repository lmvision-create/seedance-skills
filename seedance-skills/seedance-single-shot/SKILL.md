---
name: seedance-single-shot
description: Develop one motion shot from a keyframe with Seedance 2.5. Confirm the references, pick the input mode, draft three prompt variants that differ in exactly one slot, run a low-resolution pass, pick winners, re-run winners high. Use for "animate this keyframe", "turn this frame into a shot", "seedance this image". For one clip carrying several beats use seedance-multishot; to hold a character or product across a batch use seedance-consistent-refs.
---

# Seedance single shot

One keyframe plus a description of the motion, turned into a usable clip. Low-resolution exploratory pass first, high-resolution re-render on the winners only.

**Load seedance-prompt-grammar before writing any prompt.** The slot order, the bracket channels and the reference syntax live there and are not repeated here.

## The loop

### 1. The user describes the shot and drops references

One or more keyframes, plus plain language about what happens, where the camera goes, and what lands when.

### 2. Confirm the references out loud

**Open and look at every image.** Do not write a prompt for a frame you have not looked at.

Say back in one or two sentences what is actually in each frame. This catches the wrong file before it wastes a generation.

Then say which keyframe is playing which role, and which input mode that implies.

### 3. Pick the input mode

| Situation | Mode and inputs | Watch out |
|---|---|---|
| One keyframe, motion starts from it | Reference mode, keyframe as first frame | Aspect locks to the image |
| Two keyframes, motion travels between them | Reference mode, first frame + last frame | Both frames need the same shape |
| Keyframes are style anchors, not literal frames | Reference mode, images as references | Tag them `@Image N` in the prompt and say what to exclude |
| Nothing to reference at all | Text to video | Rejects any reference media |
| Continuing an existing clip | Video extension, forward or backward | The direction is required |

A keyframe means reference mode, never text to video. That is the mistake to expect.

### 4. Draft three variants that differ in ONE slot

The old habit was three whole prompts in three different voices: literal, directorial, atmospheric. That was a sensible hedge when Seedance had no published grammar. It is the wrong experiment now, because when one wins you cannot tell which part of it won.

**Hold the grammar constant. Change one slot.** Then a loss is diagnostic.

The slot worth varying is usually camera, sometimes action, occasionally style. Almost never all three.

```
V1  ... Slow push in, eye level. ...
V2  ... Locked off, no camera move. ...
V3  ... Handheld, drifting slightly right. ...
```

Everything else in those three prompts is identical, character for character.

Every variant ends with its audio channels filled, because audio generates by default and an undirected model picks something generic:

```
(low room tone)  <bottle set down on wood>
```

Present all three in one message and wait for approval.

### 5. Run the low-resolution pass

**Never generate without explicit approval.** A plan is not authorisation. Wait for a clear go from the user.

Then run the three variants together at the lowest resolution (480p), same duration, same inputs. Save them in one dated folder with a small `metadata.json` beside them recording every prompt, job id and setting, so a winner can be reproduced. Seeds are random by default, so save the output, not just the prompt.

### 6. Review as a set, not one at a time

Differences between variants are invisible in isolation and obvious in a row. Never judge from files opened one by one. Put the three side by side (a contact sheet, a grid player, or a split-screen render) and compare there.

### 7. Re-run the winners high

Only the ones the user picked. Same prompt, same inputs, higher resolution (720p or 1080p).

## What done looks like

- [ ] Every reference was looked at before it was prompted
- [ ] The three variants differ in exactly one slot
- [ ] Audio channels are filled on every variant
- [ ] The low-resolution pass ran before anything high-resolution
- [ ] Nothing was generated without an explicit go
- [ ] Winners were picked side by side, not from memory
- [ ] Prompts, ids and settings are saved next to the outputs

## When to send it back

- The keyframe is mid-motion. Back to keyframes; stillness first
- The shot needs something the keyframe does not contain. Back to the script or the keyframe
- Three variants all fail the same way. The prompt is not the problem, the frame is
