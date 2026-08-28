# UI branch — rival variants side by side

For "what should this look like?" — the day the user would otherwise spend picking between
three vague mockups in their head. Generate several **radically different** UI variations on a
single route, switchable from a floating bar, so they can be compared directly. A throwaway
route on its own is a vacuum: every variant looks fine in isolation; the value is seeing them
against each other.

## Where the variants live

- **Default — an existing page.** Gate the variants on an existing route behind a `?variant=`
  URL param, keeping the real data and auth. The comparison is honest because the content is
  real.
- **Last resort — a new throwaway route.** Only when the thing genuinely has no existing page
  to live inside, and following the project's existing routing conventions for it. This is
  the exception, not the reach-for.

## The variants

- **Three by default, capped at five.** More than that and the comparison blurs.
- **Structurally different** — different layout, different information hierarchy, different
  primary affordance. Not the same screen in different colours; that answers nothing.

## The switcher

- A **floating bottom bar** that cycles the variants — arrow buttons and `←` / `→` keys.
- **Updates the URL param**, so a variant is shareable and survives a reload.
- **Hidden in production builds**, so a stray prototype merge can't ship the bar to users.

## The payoff

The feedback you want is *"the header from B with the sidebar from C"* — the human composing
the real design out of the rivals. When you fold that decision in, **rewrite the winner
properly**; prototype UI code is not production code.
