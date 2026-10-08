# Seedance Skills

Five Claude Code skills for prompting **Seedance 2.5**, ByteDance's video model, when making AI video ads. They encode the prompt grammar, a single-shot iteration loop, multi-beat staging, reference locking for consistent characters and products, and a field guide to editing real footage.

Provider neutral: the skills teach the prompt craft, not any particular app or API. Use them wherever you run Seedance 2.5.

By MatterVision.

## The skills

| Skill | What it does |
|---|---|
| `seedance-prompt-grammar` | The spec: six slots in order, four bracket channels for music, sound, dialogue and subtitles, `@Image N` references with exclusions, limits and locked parameters. |
| `seedance-single-shot` | One keyframe to a usable clip: three variants that differ in exactly one slot, low-res pass first, winners re-run high. |
| `seedance-multishot` | Several beats in one generation, staged by event with a Maintain block, for fast animatics. |
| `seedance-consistent-refs` | Keep a person, product or place identical across a batch with a take / do-not-take reference table. |
| `seedance-guide` | Locked versus unlocked tasks, edit and reference templates, input prep, known failure fixes, and field lessons from editing real footage. |

The four workflow skills load `seedance-prompt-grammar` first, so install all five together.

## Install in Claude Code

Copy the skill folders into your personal skills directory (available in every project):

```sh
mkdir -p ~/.claude/skills
cp -R seedance-skills/* ~/.claude/skills/
```

Or into one project only:

```sh
mkdir -p .claude/skills
cp -R seedance-skills/* .claude/skills/
```

Restart Claude Code. The skills load on their own when you ask for a Seedance prompt ("animate this keyframe", "swap the background in this footage", "keep the character consistent"), or you can name one directly.

## Credits

The task types, edit and reference templates, timestamp rules, input limits and FAQ fixes in `seedance-guide`, and the slot and bracket grammar in `seedance-prompt-grammar`, follow ByteDance's **Dreamina Seedance 2.5 prompt guide** (BytePlus ModelArk documentation). All credit for the specification goes to ByteDance. Where these skills and the official guide disagree, the guide wins. The field lessons are our own.

## License

MIT. See [LICENSE](LICENSE).
