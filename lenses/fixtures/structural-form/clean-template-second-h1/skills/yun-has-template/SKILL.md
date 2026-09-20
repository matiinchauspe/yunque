---
name: yun-has-template
description: A correct skill that also carries the H1 of the document it produces.
---

# yun-has-template

This skill opens with its own name, which is the rule. It also carries a template
for the document it writes, and that template has an H1 of its own:

# <NN> — <Ticket title>

That second H1 is the TITLE OF THE PRODUCED DOCUMENT, not of this skill, and it is
correct. `yun-slice-plan` and `yun-write-spec` both carry one for real.

A lens that checks "exactly one H1" reports this file — and those two real skills —
as broken. It is not. Only the FIRST H1 is the skill's identity.
