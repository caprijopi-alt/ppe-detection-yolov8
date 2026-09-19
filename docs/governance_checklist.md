# AECO Governance Checklist

Project: PPE detection on construction imagery (YOLOv8) · Team: [FILL] · Date: 2026-09-19

## 1. Data provenance

- **Source:** `construction-safety-gsnvb`, part of the Roboflow-100 public benchmark, forked to
  `caprijopi-hotmail-com/construction-safety-gsnvb-oz6um` and used at **v1**
- **Link:** https://universe.roboflow.com/caprijopi-hotmail-com/construction-safety-gsnvb-oz6um/dataset/1
- **Owner of the raw images:** the original dataset publishers, per the Roboflow-100 benchmark
- **Licence:** **CC BY 4.0** — use permitted with attribution, which is given in the README
- **Collection date:** not stated by the publisher [FILL if the Universe page records it]
- **Our own images:** none. No client, employer or data-centre site photography is included in
  this project, by deliberate choice.

## 2. Privacy, consent and PII handling

- **Faces present?** **Yes.** This is construction-site photography of real workers; faces are
  visible in an unknown proportion of images.
- **Consent basis:** none held by us. We rely on the publisher's CC BY 4.0 release. We have no
  evidence that the individuals photographed consented to ML training use, and we cannot obtain it.
  This is a genuine weakness of using a public dataset and is recorded here rather than glossed.
- **Protection strategy applied:** none at training time — faces were not blurred, because the
  dataset was used unmodified to stay comparable with the published baseline.
- **Mitigation applied at publication:** [FILL — do this before pushing] every screenshot committed
  to `results/evidence/` is checked individually; any image with a clearly identifiable face is
  either blurred or swapped for another. Training on faces is one exposure; republishing them in a
  public repository is a second, avoidable one.
- **If this moved beyond coursework:** apply Roboflow's blur augmentation to faces before training,
  and obtain a Data Processing Agreement with each subcontractor whose workers appear.

## 3. Data minimisation

- The model answers "**is** there a person without a helmet in this frame", never "**who** is it".
- No identity, no tracking, no attendance or productivity measurement, no linking of detections to
  named individuals.
- Under the **EU AI Act**, biometric identification in the workplace is prohibited or high-risk.
  This system is scoped to stay outside that: it classifies PPE state, not people. Adding
  re-identification would change its regulatory category entirely and require a fresh assessment.
- The `person` class exists in the source schema and we kept it, but it is not used to count,
  follow or identify anyone.

## 4. Risk statement

- **High-impact false negative:** an unhelmeted worker is not flagged and no one reviews the frame.
  **Measured rate: recall 0.545 on `no-helmet` — roughly half of cases missed** (5 of 11 in
  validation). Severity: potential head injury or fatality. Mitigation: 100% human review of all
  imagery is **not** replaced by this tool; the model reorders the queue, it does not clear frames.
  The supervisor's existing walk-round is unchanged.
- **High-impact false positive:** a bystander or a bucket is flagged. **Measured: `no-vest`
  precision 0.764**, so about a quarter of those alerts are wrong. Severity: 30 seconds of
  supervisor time; at volume, alert fatigue and loss of trust in the tool.
- **Which error we tuned for:** recall-first, because a missed hazard is unrecoverable and a false
  alarm is not. The current model does **not** achieve this on the class that matters — stated in
  the README results section, not buried.
- **Known failure conditions:** distance (PPE classes fail well before `person` does), occlusion,
  small instances at `imgsz=640`, and any site whose appearance differs from the training data.

## 5. Limitations statement — when *not* to use this

This model performs **screening**, not **certification**.

- **May be used to:** pre-filter site imagery, rank frames for supervisor review, produce an
  indicative flag for a human to verify.
- **Must NOT be used to:** certify PPE compliance, close out a safety inspection, trigger automated
  enforcement or disciplinary action against any worker, serve as evidence that a site was
  compliant at a point in time, or replace a competent person's inspection.
- **Not validated on:** night-time or low-light imagery, thermal or IR, indoor data-hall
  environments, drone altitude imagery, or any camera system other than those in the source
  dataset.
- **Rare-class caveat:** the `no-helmet` metrics rest on 11 validation instances. They indicate a
  problem; they do not precisely measure its size.
- Performance figures hold only for imagery resembling the validation split. Performance on a new
  site is unknown until re-validated there.

## 6. Human-in-the-loop

- **Review process:** every flagged frame is reviewed by a site supervisor before any action. The
  model output is a worklist, never a decision.
- **Decision owner:** [FILL: role, e.g. site safety supervisor]
- **No adverse action** is taken against an individual on the basis of a model output alone.
- **Monitoring:** [FILL: e.g. a sample of N frames per week re-checked manually to catch drift as
  site conditions and the workforce change.]

## 7. Licence and rights

- **Code in this repository:** MIT — see `LICENSE`
- **Dataset rights:** CC BY 4.0, attribution given. Used unmodified.
- **Model weights:** fine-tuned from Ultralytics YOLOv8, which is **AGPL-3.0**. Distributing
  `best.pt`, or running a network service on code that imports `ultralytics`, carries AGPL-3.0
  obligations unless an Ultralytics Enterprise Licence is obtained. Recorded so a downstream user
  is not surprised.
- **Evidence screenshots:** derived from the CC BY 4.0 dataset; redistributable under the same
  terms, subject to the face check in §2.

## 8. Sign-off

| Item | Owner | Date |
|---|---|---|
| Data provenance and licence verified | [FILL] | [FILL] |
| PII check completed on all published screenshots | [FILL] | [FILL] |
| Limitations statement present in README | [FILL] | [FILL] |
| LICENSE file present | [FILL] | [FILL] |
