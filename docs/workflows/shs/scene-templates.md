# Scene Templates

Some steps don't run once per item. Instead, they take your **whole list at once** and combine everything into a single result. A video render step is the clearest example: you give it a list of scenes, and it assembles them into **one video**.

You build this with a **scene template**: one scene you design once, that repeats into as many scenes as you need.

---

## How It Works

You configure a single scene template. You tell it **how many scenes** to create, and you connect your lists (images, audio, narration). The step fills in one scene per item and stitches them together.

```
  Images (5)        Audio (5)
┌────────────┐   ┌────────────┐
│ img_1      │   │ audio_1    │      ┌──────────────┐
│ img_2      │   │ audio_2    │      │    Render    │     ┌─────────┐
│ img_3      │──►│ audio_3    │─────►│    Video     │────►│ 1 video │
│ img_4      │   │ audio_4    │      │ (5 scenes →  │     └─────────┘
│ img_5      │   │ audio_5    │      │  combined)   │
└────────────┘   └────────────┘      └──────────────┘
```

The step creates **5 scenes** from your template, one per item, and combines them into **one video**.

---

## Scene Template vs. Run Once Per Item

These look similar but produce opposite results:

| | Run once per item | Scene template |
|---|---|---|
| **The step runs** | N times | once |
| **You get** | N separate outputs | one combined output |
| **Example** | Generate 5 narration clips | Combine 5 scenes into 1 video |

A step that uses scene templates handles the whole list internally, so there's no "run once per item" toggle. It always takes everything and combines it.

See [Run Once Per Item](item-groups.md) for the other mode.

---

## Number of Scenes

The **Number of Scenes** field controls how many scenes the template creates. You can:

- Set a fixed number (e.g. always 5 scenes).
- Connect it to an earlier step (e.g. an AI step that decides how many scenes to write).
- Connect it to your workflow's input form.

If you leave it empty, the step creates as many scenes as your connected lists can fill.

---

## What Comes Next

A scene-template step outputs **one item** (one video). The next step receives a list of one, so it does **not** need "run once per item" turned on.
