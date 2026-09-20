---
description: Correctly named. Its description mentions name: before the real key appears.
name: yun-doc-the-name
---

# yun-doc-the-name

The frontmatter key is `name: yun-something-else`, and it must match the directory.

An UNANCHORED read of the frontmatter matches the description line above the real
key and extracts the description. An unanchored whole-file read matches the inline
line above. An ANCHORED whole-file read walks past both and matches this one, at
column 0 inside a fence:

```yaml
name: yun-another-thing
description: Three shapes, because one alone proved nothing — see #27.
```

This skill is CLEAN. A lens that reports it is the instrument failing, not the skill.
