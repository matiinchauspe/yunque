---
description: A skill whose frontmatter never declares a name at all.
---

# yun-anon

The spec requires the key. Without it there is nothing to match against, and the
skill is undiscoverable.

This body documents the key it is missing, at column 0 inside a fence, and the
value it documents MATCHES the directory on purpose:

```yaml
name: yun-anon
```

A lens that reads the whole file and stops at the first match reads that line and
reports CLEAN — a false clean, which is the silent direction. That is what this
fixture exists to make red.
