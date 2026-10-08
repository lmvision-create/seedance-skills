---
name: seedance-guide
description: Write Seedance 2.5 prompts the way ByteDance's official prompt guide says to, plus field lessons from real production runs. Covers locked versus unlocked tasks, editing real footage (swap a background, add a crowd, keep the actors), reference videos, keyframes and extensions, ready-to-fill edit and reference templates, input preparation, and a table of known failure fixes. Use for "edit this footage in seedance", "swap the background", "keep the footage locked", "unlocked reference", "write me a seedance prompt".
---

# Seedance 2.5 guide

Source of truth: ByteDance's **Dreamina Seedance 2.5 prompt guide** (published in the BytePlus ModelArk docs). If anything below disagrees with it, the guide wins. The field lessons at the bottom come from real production runs editing live-action ad footage.

For slot order, bracket channels and limits, load seedance-prompt-grammar.

Every job runs in five steps: **pick the task type, build the reference table, write the prompt from the template, prepare the inputs, show the prompt and generate only on an explicit go.**

## Step 1. Pick the task type (this is what "locked" and "unlocked" mean)

Seedance 2.5 sorts every job into two kinds. This is a property of the task, not a word you sprinkle in the prompt.

| Kind | Task | What happens | Must set |
|---|---|---|---|
| **Locked** | **Edit** an existing video | The input video sits on the timeline; output keeps its shape and length (within 0.3 s) | aspect ratio `adaptive`, duration `-1`, and the prompt contains an edit trigger: *edit, add, insert, remove, delete, modify, replace, change to* |
| **Locked** | **First frame / first + last frame** | Output starts (and ends) on the image, shape follows the image | image role first frame / last frame, aspect ratio `adaptive`, both frames the same shape |
| **Locked** | **Extend** forward or backward | Output continues the video | aspect ratio `adaptive`, trigger: *extend forward, extend backward, continue* |
| **Unlocked** | **Reference** (images, videos, audio used for meaning only) | You choose shape and length; the model re-creates | normal aspect ratio and duration |
| **Unlocked** | **Keyframes** ("Use Images 1 to N in order as keyframes") | Follows the frames fairly strictly | first sentence names the keyframes |

**"Keep my footage, change the place"** = an **Edit** task (the footage is locked) with the location as a **reference** (unlocked). Say both in the prompt in plain words: "Editing task: edit @Video 1 … @Image 1 is an unlocked reference, use it only for …".

## Step 2. Build the reference table before writing a word

One row per file, in upload order. Tags are positions: `@Video 1`, `@Image 1`, `@Image 2`. Every row needs a "do not take" cell.

| Tag | File | Role | Take from it | Do NOT take |
|---|---|---|---|---|
| @Video 1 | the footage | locked edit source | everything not named below | nothing is invented over it |
| @Image 1 | location plate | unlocked reference | what the place is | its framing, camera angle, sharpness, colour grade |

Rules from the guide:
- Bind every asset in the prompt text, by number and by job ("the plaza in @Image 1"). Never rely on text written inside an image.
- Say which **part** of an asset to use ("Refer to Image 1 for lighting and filters").
- If a reference is accurate, **refer to it and stop**. Do not also describe it at length; that gives the model two sources.
- Upload subjects in the order they first appear, so numbering matches.
- 1 to 5 reference images for an edit is the stable range; 6 to 8 gets shaky.

## Step 3. Write the prompt from a template

Use positive descriptions. Negatives only for subtitles and audio.

### Edit template (locked footage, unlocked reference)

```
Editing task: edit @Video 1. Keep @Video 1 unchanged: <what stays - people, faces, performances, every cut and its timing, framing, focus, grade, and the LIGHTING DIRECTION on the people, e.g. warm key light from frame left>.
Change only <the scope>: replace <A> with <B from @Image 1>.
@Image 1 is an unlocked reference: use it only for <what>, not for its framing, camera angle, sharpness or colour.
<Any addition, described as A -> B, e.g. add more people behind the people already there, matching them in clothing, lighting and focus.>
<Same-camera line: the new background is shot on the same lens as @Video 1 - anamorphic, shallow depth of field, soft oval bokeh, gentle horizontal flare, film grain, the same grade.>
No subtitles, no new text. <audio line, see below>
```

For a partial edit, scope it with whole-second timestamps: "from 4 to 6 seconds in Video 1 … leave the rest unchanged".

### Reference / new-shot template (unlocked)

```
<One-sentence summary: subject + location + event + style + camera.>
<Asset bindings: "The man in Image 1 is the host; Image 2 is the location; follow the camera move in Video 1.">
0-3s: <what we see, camera, sound>
3-7s: <...>
<Recurring details that stay constant: camera angle, environment, atmosphere, sound.>
```

"Shot N" works instead of timestamps. Keyframes: open with "Use Images 1 to N in order as keyframes."

### Timestamps

- Whole seconds only, no gaps ("0-3s … 3-7s … 7-10s").
- Too little in a range and the model improvises; **too much in a range and it adds cuts or drops parts**. Never list sub-second shots.
- Do not time high-frequency actions ("nods three times a second").

### Camera and action language

Plain terms work: shot size, push in, pull out, pan, track, orbit, tilt, handheld, low angle, overhead, long take, dolly zoom, speed ramp. For a niche term, write the term plus what it does ("rack focus: the foreground goes soft as the man behind comes into focus"). Transitions need a time and a method ("at 5s, a left wipe with a dissolve"). Keep action descriptions general; spell out only the one or two memorable moves.

### Audio

- If you want foley only (common for ads, where music is laid in the edit), know that the model sneaks music in. **Enumerate and repeat**: put "No music of any kind: no music, background music, BGM, score, instrumental, melody, synth effects or ambient pad. Only <voices / room sound / action sounds>." at the **start and the end** of the prompt.
- If you will lay the original audio back in the edit, still write the audio line; it keeps the generated track clean.
- Water, waves and echoes in the prompt raise the odds of bubbly audio artifacts.

### Known failure fixes (from the guide's FAQ)

| Problem | Fix |
|---|---|
| Complex job (several edits and references at once) comes back unstable | **Split it** into simpler passes, one Edit or Reference per pass |
| Edit output ~0.33 s short | Make the input **8n+1 frames** (e.g. 97 at 24 fps), or use Reference mode with a set duration |
| Changing the aspect ratio in an edit invents extra picture | Use Reference mode with a shot-by-shot description instead |
| Subtitles appear anyway | Say "No subtitles"; do not attach tone or action notes to individual spoken words |
| Glowing eyes | "normal human eyes; no glowing eyes" first; use calm emotion words ("amazed", not "fanatical") |
| Wrong face on the wrong character | Upload references in order of first appearance and renumber |
| Misspelled on-screen text | Put the text in a reference image, or spell it letter by letter; safest is still to add graphics in post |
| Fingerprint texture on grass or foliage | Resize the reference image to no bigger than the output |
| JPG rejected (HEIC decode error) | Re-save as a standard PNG or 4:2:0 JPG with ffmpeg |

## Step 4. Prepare the inputs (these are rejected before rendering, so check first)

- **Edit source video: 4 to 30 s** (the guide recommends under 20 s). Shorter? Hold the last frame with ffmpeg's `tpad` filter.
- **Reference video: between 407,696 and 8,295,044 pixels** per frame and **aspect 0.4 to 2.5**. A 2.67:1 anamorphic frame must be cropped to 2.39 or narrower; 720x480 is too small, use 600p or more.
- **Anamorphic footage: de-squeeze first.** Check the sample aspect ratio with ffprobe (`sample_aspect_ratio=3:2` means 1.5x); scale with `scale=iw*sar:ih,setsar=1`.
- No letterbox bars and no burned-in labels in the source; crop them out.
- MOV output is recommended for edits and extensions (better colour and brightness match).
- If you call Seedance through an API, reference files usually have to be publicly reachable links. Host them somewhere temporary and remove them afterwards.

## Step 5. Show the prompt, then generate on an explicit go

Show the user the full prompt and the reference table. Generation runs only after an explicit go from the user. A plan is not approval.

Save the request (prompt, inputs, settings) and the job id before waiting on the result, so a dropped connection never means running the same job twice.

To lay the original sound back over an edited result:

```
ffmpeg -i result.mp4 -i original.mp4 -map 0:v -map 1:a -c:v copy -shortest result_with_audio.mp4
```

## Field lessons from real runs

These come from editing live-action ad footage with Seedance 2.5: background swaps, crowd additions, partial edits.

1. **For a background swap, describe the place in words and send no plate image.** Sending the plate made Seedance copy its sharpness, angle and light and paste it flat behind the people. The best backgrounds came from a text-only edit that described what the plate shows: "historic plaza at street level, lamp on the left, old brick buildings with arched warm windows, upper floors fading into a dark sky". A bare city name gives you a generic skyline, so describe the look, not the city.
2. **The plate's light wins unless you pin the light.** Write where the key light comes from on the people ("warm key light from frame left") and flip any reference so its lamp or practical sits on the same side.
3. **Same-camera line.** New backgrounds come out sharper and cleaner than real footage unless you say "same anamorphic lens, same depth of field, softness, grain and grade, never sharper than the people".
4. **Montages do not edit evenly.** A 4 s clip with six cuts came back with some shots untouched and others replaced, whether the prompt listed the shots or not. Edit each shot on its own (with at least 4 s of handles) and recut to the original timing.
5. **A wrong shot list is worse than none.** If you do describe shots, check them against the actual file (`ffmpeg -i clip.mp4 -vf "select='gt(scene,0.2)',showinfo" -f null -` prints the cut times).
6. **Seed is random by default (-1).** The same prompt will not reproduce a result; save the output, not just the prompt.
7. **Lay the original audio back** after an edit when the performance is real.
8. **Scope a partial edit with whole-second timestamps.** "Edit only the first shot (0-1s) and the last shot (3-4s); leave 1-3s exactly unchanged" kept two shots untouched while the other two got the new background.
9. **The background recipe that held across a cut-heavy clip:** an Edit task with the footage locked, the place described in words, plus ONE unlocked reference image that is a frame from an earlier output the director liked, used "for the background only: its buildings, lit windows, sky and street depth; not its people, signs, framing or sharpness". It was the first approach to put the new background behind all six shots of the clip.
10. **Never name the thing you do not want.** "No new shot, no wide shot" produced a wide shot in three of six shots. Say what to keep in positive terms ("the same six shots in the same order") and leave the unwanted noun out.
11. **Do not relight with an edit pass.** A locked edit asked only to relight one shot came back unchanged. Fix exposure and colour in a grade (ffmpeg curves on that shot's frame range, or your editor), not in Seedance.
12. **A background reference must contain no people.** A liked frame containing a person, a crowd and signs re-staged a different clip to copy that crowd and those signs. The same recipe with an empty frame of the same location kept the clip's own people and signs exactly. On a clip whose people already matched the frame it was harmless, but do not count on that.
