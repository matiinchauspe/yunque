---
name: yun-fenced-only
description: A skill with no H1 of its own, whose only heading sits inside a code fence.
---

This document never opens with its own name. Its only `#` line is an example of
what a skill looks like, shown inside a fence:

```markdown
---
name: yun-fenced-only
---

# yun-fenced-only
```

A fence-blind lens finds that line, matches it against the skill's name, and
reports CLEAN. **The document has no H1 at all.** `yun-model-domain` carries a
`# Event-sourced orders` inside a fence for real, which is where this bug lives.
