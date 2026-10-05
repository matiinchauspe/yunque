---
name: yun-show-me
description: Re-pitch what did not land — a little context, plain words in the domain's own language, and the smallest view that shows it.
disable-model-invocation: true
argument-hint: "What did not land? (default: the last message)"
---

# yun-show-me

The last thing said did not land. **Re-pitch** it — the same content, built again for a reader
who lost the thread.

## Steps

1. **Find what did not land.** The user's argument names it; without one, it is your last
   message. Done when you can state, in one sentence, the point that failed to land.

2. **Rebuild the ground under it.** Open with a little context: where the work stands, what was
   already decided, and the question this point answers. Done when a reader who skipped the
   last few turns could follow the next sentence.

3. **Say it in Simplified Technical English.** Short sentences, one idea each, active voice,
   one word per meaning. Name domain things by their canonical term: `consult` the project's
   domain model through `skills/_shared/domain-convention.md` and use the glossary's word, never
   a synonym for it. Where there is no model, the plain word wins over the clever one.

4. **Show it.** Pick the smallest view from [`VISUALS.md`](VISUALS.md) that carries the point
   and put it beside the sentence it supports. Done when every view earns its place against
   that sentence.

Re-pitch only what is already on the table. Where rebuilding it exposes a hole — a step nobody
decided, a term used two ways — name the hole as a hole; it is often the reason the point did
not land.

## Close on the point

End with **one sentence**: the point, restated in the words step 3 chose, phrased so the user
can refute it at a glance. A re-pitch that ends without it has rebuilt the context and left the
point implied — the same miss as the message it replaced.

## Attribution

Adapted for this workspace from **Matt Pocock's `wait-what`** (github.com/mattpocock/skills, MIT)
and **Dex Horthy's `show-me`** (github.com/humanlayer/skills, MIT), merged into one skill: the
re-pitch in Simplified Technical English and the domain's language comes from the first, the
visual vocabulary — disclosed to `VISUALS.md` so `yun-write-pr` reads the same menu — from the
second. Changes: the glossary is reached through `skills/_shared/domain-convention.md` instead of
a fixed `GLOSSARY.md`; added the hole rule and the closing sentence.
