---
name: yun-doc-the-flag
description: A model-invoked skill whose BODY documents the user-invocation flag. Its frontmatter is clean.
---

# yun-doc-the-flag

This skill is model-invoked. Its frontmatter carries no invocation flag.

Its body, however, explains the flag to whoever is authoring a skill. It does so in
the two shapes that prose actually takes, and each one defeats a different broken lens:

- A **user-invoked** skill sets `disable-model-invocation: true` in its frontmatter.
  Inline, inside backticks — this defeats an UNANCHORED match.
- A **model-invoked** skill omits it. Written out, the frontmatter of a user-invoked
  skill looks like this:

```yaml
---
name: yun-example
disable-model-invocation: true
---
```

That fenced line sits at column 0 and matches an anchored pattern exactly — so it
defeats a lens that reads the whole file even when its regex is anchored.

Both forms are here on purpose. A lens that reports this skill as user-invoked is
broken; the skill is model-invoked and its frontmatter is clean. That false positive
is friction #23, measured twice in the wild, and this fixture is what stops a third.
