# Acceptance criteria — TakeMeter

Five criteria that say what "working" means for this classifier, written in
unit 5 **before** anything was trained.

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

## 1. Overall accuracy

Across the held-out test set, TakeMeter should reach an overall accuracy of at least 0.80.

**Why this target:**
With three labels, random guessing would only be around 33%, so I want the classifier to be correct much more often than chance. I chose 0.80 because I would only consider the classifier useful if it can correctly sort most JRPG posts.

---

## 2. Per-label performance

Each of the three labels — `analysis`, `request`, and `community_discussion` — should have an F1 score of at least 0.70.

**Why this target:**
I do not want a high overall accuracy score to hide one label that performs badly. I chose 0.70 because each category should work reasonably well on its own, even though some labels may have fewer examples than others.

---

## 3. Dataset balance

No single label should make up more than 65% of `labels.csv`.

**Why this target:**
If one label dominates the dataset, the classifier could learn to guess that label instead of learning the differences between my categories. I chose 65% because it stays below the project's 70% maximum while still allowing the natural distribution of posts to be uneven.

---

## 4. Labeling consistency

My agreement rate on the Unit 6 consistency check should be at least 80%.

**Why this target:**
The taxonomy should be clear enough that the same rules can be applied consistently instead of depending on how I feel about a post at the time. I chose 80% because I expect some borderline posts, especially between `analysis` and `community_discussion`, but most posts should still be classified consistently.

---

## 5. Confidence usefulness

The accuracy of TakeMeter's most-confident third of predictions should be at least 15 percentage points higher than the accuracy of its least-confident third.

**Why this target:**
If confidence means anything, the predictions the model is most sure about should be noticeably more reliable than the ones it is unsure about. I chose a 15-point gap because I want confidence to provide useful information instead of just being a number attached to every prediction.

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
