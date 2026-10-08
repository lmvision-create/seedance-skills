---
name: seedance-consistent-refs
description: Hold a character, product or location identical across a batch of Seedance 2.5 generations using a reference table of tagged references with explicit exclusions. Use for "keep the character consistent", "same product every shot", "lock the look across the batch", "the face keeps changing", "reference bleed". For a single shot use seedance-single-shot; for one clip carrying several beats use seedance-multishot.
---

# Seedance consistent references

The job this skill exists for: **the same person, product or place has to look the same in shot 1 and shot 9.**

Before 2.5, reference images were passed positionally and you hoped. 2.5 lets you name each file inside the prompt and say what to take from it and what to ignore. That second half is the whole trick, and it is the fix for the failure everyone knows: you attach a reference for the jacket and the model also copies the sky, the parked cars and the time of day.

Load seedance-prompt-grammar first. Everything here assumes it.

## When this is the right skill

- A spokesperson or character appears in more than one shot
- A product has to be recognisably the same object across a batch
- A location repeats and has to read as one place
- Something is drifting and you cannot work out which reference is causing it

Not for one-off shots with nothing to hold. Use seedance-single-shot.

## Build the reference table first

Before writing any prompt, write this. It is the artifact, and it lives next to the shots.

| Tag | File | Take from it | Do NOT take |
|---|---|---|---|
| @Image 1 | `ref/talent_face_neutral.png` | Face, hair, skin | Background, clothing, lighting |
| @Image 2 | `ref/talent_wardrobe.png` | Jacket, collar, colour | Face, pose, room |
| @Image 3 | `ref/kitchen_wide.png` | Room, counter, window light | Any person in frame |
| @Video 1 | `ref/walk_cycle.mp4` | Gait and pace only | Wardrobe, face, location |

Two rules make this work:

**One element, one file.** If two files could plausibly supply the jacket, the model will average them and you get a third jacket. Decide which file owns each element.

**Every row needs a "do not" cell.** A blank exclusion column is how bleed gets in. If you cannot think of what to exclude from a file, look at it again; there is always a background.

Upload references in the order their subjects first appear, so the numbering matches the action.

## Writing the prompt

Slots as normal, with the reference sentences sitting in the slot they belong to.

```
The talent stands at the counter and turns to camera as she finishes a sentence.
Use her face and hair from @Image 1. Do not use its background or lighting.
Her jacket is the one in @Image 2. Do not use its face or pose.
The room is @Image 3, morning light through the window on the left.
Do not use any person from @Image 3.
Locked-off medium, eye level, no camera move.
{So that is the part nobody tells you.}
(room tone only)
```

Note what is absent: no adjectives describing her face, no prose about the jacket. The references carry the look. Describing a referenced thing in words gives the model two sources for one element, which is the same failure as two files.

## Naming

Name the subject identically in every prompt in the batch. "The talent", always. Not "she" in one prompt, "the presenter" in the next and "our host" in the third. The label is what ties the shots together in the model's head as much as the image does.

## Batch procedure

1. Build the reference table. Save it as `refs.md` next to the shots.
2. Write shot 1. Generate once, at the lowest settings.
3. **Look at it before writing shot 2.** If the face is wrong at shot 1 it is wrong at shot 9 and you have wasted nine generations.
4. Once shot 1 holds, the reference block is frozen. Copy it verbatim into every other shot. Change only the action, the camera and the audio.
5. Generate the rest.
6. Review as a batch, not one at a time. Drift is invisible in isolation and obvious in a row. Put every shot side by side, and for a strict head to head line up every version of one shot together.

## When something drifts

Work down this list. Do not change two things at once.

1. **Is the same element claimed by two files?** Most common cause by far.
2. **Is the element also described in prose?** Delete the words, keep the reference.
3. **Is the exclusion missing?** Add it and re-run the one shot.
4. **Are there too many subjects?** The stable range is 1 to 8. A crowd scene will not hold a face.
5. **Is the reference itself ambiguous?** A three-quarter face at low resolution is a weak anchor. Regenerate the reference before blaming the prompt.

If it still drifts after all five, the reference set is wrong and no prompt will save it. Go back and make better references.

## What done looks like

- [ ] `refs.md` exists, every row has a take and a do-not
- [ ] No element is claimed by two files
- [ ] Nothing referenced is also described in words
- [ ] The subject is named identically in every prompt
- [ ] Shot 1 was approved before the batch ran
- [ ] The batch was reviewed in a row, not one at a time

## When to send it back

- References are low resolution or ambiguous. Back to whoever made the keyframes
- The script needs more than 8 distinct subjects in one shot. Back to the script, because it is a script problem
- The same element genuinely needs two sources. Back to the script; the shot is asking for something the model cannot hold
