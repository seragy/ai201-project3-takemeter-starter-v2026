# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

> These five criteria, their targets and their reasons were written by Claude
> (Anthropic's AI assistant) before the first training run, not by the student.
> See "How I Used AI" in the README.

**All five are yours this time.** None are given. You've had two projects of
practice.

An acceptance criterion names a number. *"The model is accurate"* is an
opinion. *"Every label has an F1 of at least 0.60 on the held-out set"* is a
criterion.

Under each, write a sentence or two on **why that number**. A reason that says
something about your data or your taxonomy earns credit — *"I picked 0.60 F1
for `reaction` because it's my smallest label and I only have about 50
examples of it"*. A reason that could be attached to any project does not.

> Missing your own targets next unit costs you nothing. Setting a target so
> easy you can't miss it does.

---

## Pick numbers you can defend

Not numbers that sound impressive. Three labels means a coin-flip guesser gets
about 33%, so a target of 0.40 is barely a target. Your number should sit
somewhere you'd honestly call useful.

**Cover at least three of these five areas.** They're here as prompts, not as a
form to fill in — a criterion that fits none of them is fine if it names a
number.

| Area | A question it could answer |
|---|---|
| Overall accuracy | How often does it need to be right to be worth using? |
| Per-label performance | Is one label allowed to be much worse than the others? |
| Balance | How lopsided can your label counts get before it's a problem? |
| Consistency | If someone else labelled the same posts, how often should you agree? |
| Confidence | Should a confident prediction be right more often than an unsure one? |

Two things worth knowing before you pick numbers, because both will affect
whether you hit them:

- **Your smallest label will have the jumpiest score.** If a label has 50
  examples, about 8 land in the test split. An F1 computed on 8 examples moves
  a lot between seeds. A target for that label should be looser than one for
  your biggest label, and saying so is a good reason.
- **Unit 6 tests across three seeds, and the target has to hold across all
  three.** A target of 0.65 against results of 0.71, 0.62, 0.68 is a **miss**.
  Pick with that in mind — it is stricter than it first sounds.

---

## 1.

Overall accuracy is at least **0.60** on the held-out test split, on every seed.

**Why this target:** My three labels are close to balanced (113 / 95 / 95), so always guessing the biggest label (`reaction`) scores about 0.37 and a random guess about 0.33. At 0.60 the model is right noticeably more often than not, roughly what I'd want before trusting it to sort a thread. Higher would be optimistic: about 210 training posts, and the `take` vs `analysis` line depends on a single clause ("the detail has to support the point") that a small model may not pick up.

---

## 2.

Every label has an F1 of at least **0.50** on the held-out test split, on every seed.

**Why this target:** A 15% test split of 303 posts gives about 14–17 posts per label, so one wrong post moves a label's F1 by around 0.03–0.07, and the target has to survive three seeds. That's why it sits 0.10 below the accuracy target. I'd still call 0.50 a working label. Below that, `take` or `analysis` is essentially being guessed, which is the boundary my taxonomy is built around.

---

## 3.

The fine-tuned model's accuracy beats the zero-shot baseline (`facebook/bart-large-mnli`, given my label definitions) by at least **0.15** on the same `test_split.csv`.

**Why this target:** My labels hinge on structure (is there a checkable detail, and does it support the point?) rather than topic. A general entailment model reading my one-line definitions should get some of `reaction` right and little else. If ten minutes of fine-tuning doesn't buy 15 points over that, the 300 labels weren't worth making.

---

## 4.

No label is more than **40%** of `labels.csv`, and none is less than **25%**.

**Why this target:** I oversampled long posts and posts containing numbers, and cut `reaction` down from 169 to about 110, so the counts are something I control. With three labels, 40% / 25% is the widest spread I'd accept before the model can gain by leaning toward one label. It also keeps `analysis`, the thinnest label before the cutting, above about 75 examples.

---

## 5.

On every seed, accuracy on the model's **most confident third** of test posts is at least **0.15** higher than on its **least confident third**.

**Why this target:** Hard cases in my data cluster on one boundary (`take` vs `analysis`), so I expect the model's unsure posts to be exactly those. If confidence carries no information about correctness, nobody could use it to send the unsure posts to a human. With about 15 posts per third, one post is about 0.07, so 0.15 means at least two or three more right answers in the confident third.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     UNIT 6 — read this before you change anything above.

     If a criterion turns out to be BROKEN rather than merely unmet, you can
     revise it, and that earns credit. But never delete or edit the original
     line. Add the revision underneath, like this:

         ## 2. Every label performs acceptably

         The model performs well on all labels.

         **Why this target:** ...

         > **Revised in unit 6:** Every label has an F1 of at least 0.60 on
         > the held-out set.
         >
         > **Why revised:** "performs well" gave me nothing to check. I
         > couldn't produce a verdict from it at all.

     That's a revision because the criterion couldn't be MEASURED.

     Lowering a target because you missed it is not a revision, and it costs
     you the point:

         ✗ "Overall accuracy of at least 0.65" → "at least 0.55", because
            0.65 turned out to be optimistic for 200 examples.

     A number you missed stays where it is, gets diagnosed, and gets a fix
     attempted. That's where the points are.
     ───────────────────────────────────────────────────────────────────────── -->
