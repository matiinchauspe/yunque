---
name: yun-headless
description: A skill that carries no H1 at all, only section headings.
---

## Overview

This is `yun-research`'s real defect, fixed in `8671789`: the document carried no
top-level heading whatsoever.

## Why the section headings matter here

A lens matching `^#` instead of `^# ` also matches `##` and `###`. Against this
file it would find `## Overview`, decide the skill opens with the wrong name, and
report WRONG-H1 — a true defect described falsely. The right reading is NO-H1.
