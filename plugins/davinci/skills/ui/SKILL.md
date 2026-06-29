---
name: ui
description: >-
  Use when implementing or changing anything in the ui layer of a Layer Cake
  project — building a component, adding markup, reaching for a button, input,
  text, icon, or any visual building block, or reviewing presentation code.
  Establishes that the ui layer is built out of primitives: bare building blocks
  that live in ui/primitives/ (one file each, re-exported through a barrel) and
  are consumed from that barrel rather than pulled raw from a component library
  or hand-rolled as raw elements. Trigger before writing JSX/markup in ui,
  whenever you'd otherwise type a raw <button>/<input>/<span> or import straight
  from a component library, and whenever you spot a component reaching past the
  primitives barrel. Trigger even when the user only says "build the UI", "make
  this component", or names an element without saying "primitive".
---

# UI — build it out of primitives

This is the **implementation skill for the `ui` layer** of the Layer Cake. The
[[layer-cake]] skill tells you *what belongs* in `ui` (isolated presentational
components, props typed from models, no server imports, no data shaping). This
skill tells you *how the layer is built*: out of **primitives**.

We grow this skill over time. Right now it covers one thing, and it's the first
thing to get right — **the primitives**.

## What a primitive is

A **primitive** is a *bare building block* of the interface: a `Button`, `Input`,
`Text`, `Icon`, `Stack`, `Spinner` — the smallest visual units everything else is
composed from. It is bare on purpose: it carries no domain knowledge, no
business data, no feature-specific behavior. A `UserCard` or a `CheckoutForm` is
**not** a primitive — those are composed `ui` components built *from* primitives.
The tell: a primitive is reusable in any app; a composed component only makes
sense inside one feature.

There is exactly **one exception** to "everything is a primitive": **semantic
HTML structural elements** — `div`, `main`, `header`, `aside`, `section`, `nav`,
`footer`, `ul`/`li`, and the like. Use those raw. They carry *structure and
meaning*, not styling or behavior, so wrapping each one in a primitive would add
a layer that buys nothing. Everything with a *visual* identity — anything you'd
style, theme, or attach behavior to — goes through a primitive instead of a raw
element.

## Where primitives live

```
ui/
  primitives/
    index.ts        # the barrel — the single import surface
    button.tsx      # one file per primitive
    input.tsx
    text.tsx
    icon.tsx
```

Two rules hold this together:

1. **One file per primitive**, all re-exported through the **barrel**
   (`primitives/index`). The barrel is the layer's *single import surface* for
   building blocks.
2. **The rest of `ui` imports primitives from the barrel — never from the
   component library directly, and never as a raw styled element.** The library
   (if any) is an implementation detail that only the primitive files know
   about.

### Why route everything through the barrel

The barrel makes `primitives/` the **single seam** between the interface and
whatever powers the building blocks. Swap the component library, restyle the
button, change the underlying element — you touch the one primitive file, and
every consumer is unaffected because they all import `Button`, not the library.
Import the library directly from a feature component and that seam is gone: the
swap now means a codebase-wide find-and-replace. The barrel is cheap insurance
that keeps presentation decoupled from the toolkit rendering it.

## With or without a component library

The primitives concept is the same either way — only what's *inside* each
primitive file changes. Resolve which case you're in from the project (the same
way [[layer-cake]] resolves its bindings):

- **Using a component library** (shadcn, Radix, Mantine, MUI…): the primitive
  file **re-exports the library's component** (re-export as-is, or wrap to rename
  / fix props / pin a default). The point of the file is to *surface* that
  component through the local barrel so `ui` consumes it from one place. The
  library name appears only inside `primitives/`.
- **No component library**: the primitive file **defines** the building block
  itself — a styled element with its props. Same barrel, same single-seam
  consumption; the implementation is hand-rolled instead of re-exported.

## Adding a primitive

When a component needs a building block that isn't surfaced yet — you're about
to type a raw `<button>`, or import a library component directly — that's the
signal to **add a primitive**, not to reach past the seam:

1. Create `primitives/<name>.tsx` — re-export the library component, or define
   the element if there's no library.
2. Export it from the barrel (`primitives/index`).
3. Import it from the barrel at the use site.

A raw styled element or a direct library import in a feature component is the
smell that a primitive is missing. The fix is always to surface it as a
primitive first, then consume it.

## Smells

- A raw `<button>`, `<input>`, `<span>`, or other *visual* element styled inline
  in a feature component → it wants to be a **primitive**. (Raw `div`/`main`/
  `header`/`aside` and friends are fine — that's the semantic-HTML exception.)
- A component **importing straight from the component library** (`from
  "@radix-ui/..."`, `from "@mui/..."`) instead of from the primitives barrel →
  route it through `primitives/` so the library stays a single-seam detail.
- A **primitive carrying domain data or feature logic** → it's not a primitive,
  it's a composed `ui` component; move the domain part out and keep the
  primitive bare.
- A primitive file that **isn't exported from the barrel** → consumers will
  reach into `primitives/<name>` directly or duplicate it; wire it through the
  barrel so there's one import surface.
