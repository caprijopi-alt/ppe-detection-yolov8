# SAM 3 exploration — what helped, what did not

Session 2 asked us to test Meta's Segment Anything (SAM 3) as a smart-labelling aid alongside
manual Roboflow annotation. This is the honest record of that experiment. SAM was used **only as
an annotation accelerator**; it is not part of the trained model or the inference pipeline.

## What we tried

- **Tool:** SAM 3 via [FILL: the Meta demo at aidemos.meta.com/segment-anything / Roboflow's
  Smart Polygon / Auto Label]
- **Images tested:** [FILL: N] images from our set, chosen as [FILL: a mix of clean and cluttered
  frames]
- **Prompting approach:** [FILL: single click on the object / box prompt / text prompt "<class>"]

## What helped

- [FILL: e.g. "Large, high-contrast objects against a plain background were segmented in one click;
  converting the mask to a bounding box was faster than drawing it by hand."]
- [FILL: e.g. "Useful for finding instances we had missed in busy frames — the mask proposals acted
  as a second pair of eyes."]

## What failed — the "kryptonite" check

- [FILL: e.g. "Thin, low-contrast targets (hairline cracks) were either missed or merged with
  surrounding texture."]
- [FILL: e.g. "Occluded instances were split into two separate masks, which converts into two wrong
  boxes rather than one right one."]
- [FILL: e.g. "SAM segments *a thing*, but has no idea which of our classes it is — every mask
  still needed a human to assign the class, so the time saving was smaller than it first looked."]

## The geometry tax

SAM returns **masks** (segmentation); YOLOv8 detection training needs **boxes**. Converting a mask
to its bounding box loses the shape information and, for diagonal or irregular objects, produces a
box containing a lot of background. [FILL: state whether this mattered for your classes — it
matters most for long thin diagonal objects.]

## Verdict

[FILL: e.g. "We used SAM for roughly N of our images, mainly for <case>, and annotated the rest
manually. Net time saving: modest. We would use it again for <case> and not for <case>."]

No SAM output was accepted without human review. Every box in the final dataset was checked by a
team member against the rules in `class_definitions.md`.
