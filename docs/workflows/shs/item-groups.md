# Run Once Per Item

When a step receives a **list** as input, you decide how to handle it with the **Run once per item** toggle.

---

## The Two Modes

### Off - step runs once

The step receives the full list but only uses the **first item**. Everything else is ignored.

```
  Prompt (3)            Style (3)
┌──────────────┐      ┌──────────────┐      ┌────────────┐     ┌─────────┐
│ ► "sunset"   │      │ ► cinematic  │      │  Generate  │────►│ image_1 │
│   "forest"   │─────►│   watercolor │─────►│   Image    │     └─────────┘
│   "city"     │      │   neon       │      └────────────┘
└──────────────┘      └──────────────┘          1 run
```

### On - step runs once per item

The step runs **N times**, once for each item. Multiple list inputs are paired together by position.

```
  Prompt (3)       Style (3)                                  Output
┌────────────┐   ┌────────────┐                            ┌─────────┐
│ "sunset"   │   │ cinematic  │                            │ image_1 │
│ "forest"   │──►│ watercolor │──► Generate Image (3×) ─-─►│ image_2 │
│ "city"     │   │ neon       │                            │ image_3 │
└────────────┘   └────────────┘                            └─────────┘
```

Use this when you want one output per item - for example, generating one image per prompt.

---

## When Lists Have Different Lengths

### Chops at the shortest

By default, the step stops when the **shortest list runs out**.

```
  Prompt (3)       Style (2)                                  Output
┌────────────┐   ┌────────────┐                            ┌─────────┐
│ "sunset"   │   │ cinematic  │                            │ image_1 │
│ "forest"   │──►│ watercolor │──► Generate Image (2×) ─-─►│ image_2 │
│ "city" [✗] │   └────────────┘                            └─────────┘
└────────────┘
```

### Loop - cycles the shorter list

Enable **Loop** on a field to cycle it back to the start instead of stopping.

```
  Prompt (3)       Style (2, loop ↺)                         Output
┌────────────┐   ┌────────────┐                            ┌─────────┐
│ "sunset"   │   │ cinematic  │                            │ image_1 │
│ "forest"   │──►│ watercolor │──► Generate Image (3×) ─-─►│ image_2 │
│ "city"     │   │ cinematic ↺│                            │ image_3 │
└────────────┘   └────────────┘                            └─────────┘
```

Loop is toggled **per field** using the ↺ button next to the field selector. One input can loop while another chops.

---

## All Scenarios

| # | Prompt | Style | Runs | Result |
|---|--------|-------|------|--------|
| 1 | single value | single value | 1 | 1 output |
| 2 | list (3) | single value | 3 | 3 outputs - static value repeats each run |
| 3 | list (3) | list (3) | 3 | 3 outputs - paired by position |
| 4 | list (3) | list (2) | 2 | 2 outputs - stops when shorter runs out |
| 5 | list (3) | list (2) loop ↺ | 3 | 3 outputs - style cycles back to start |
| 6 | list (3) | list (5) | 3 | 3 outputs - longer list chops to match shorter |

---

## Exception: steps that combine instead of iterate

A few steps, like **video render**, don't offer this toggle. They take your **whole list at once** and combine everything into **one result** (e.g. a list of scenes → one video). You build these with a **scene template**: one scene you design, repeated N times, all assembled into a single output.

The next step then receives a list of one, so it does **not** need "Run once per item" turned on.
