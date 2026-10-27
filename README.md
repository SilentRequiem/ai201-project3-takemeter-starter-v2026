# TakeMeter

> ### 👋 Start here
>
> **New to this repo? Read [RUNNING.md](RUNNING.md) first** — setup, the
> notebook, the baseline, and what to do when something breaks.
>
> Once `python test.py` passes:
>
> ```bash
> head -5 data/practice_labels.csv     # the shape your labels.csv needs
> ```
>
> Then open `takemeter.ipynb` **in this folder** — in VS Code, or with
> `jupyter notebook` if you prefer. Pick the kernel: the `.venv` inside this
> project. Run section 1, which reports the hardware you'll be training on.
> Everything else waits until you have data.
>
> Nothing to upload, nothing to connect, no accounts and no keys. The notebook
> runs on your machine and writes next to your code.
>
> **The rest of this file is your submission.** Fill it in as you go.

---

<!-- ─────────────────────────────────────────────────────────────────────────
     HOW TO USE THIS FILE

     Unit 5 asks for the first five sections. Unit 6 adds the five below them.

     Everything is pasted as TEXT. No screenshots, no images.

     ⚠️ The confusion matrix especially. The notebook prints one as a markdown
     table, ready to copy. A screenshot of a matrix earns nothing. Paste the
     table.
     ───────────────────────────────────────────────────────────────────────── -->

<!-- ═══════════════════════ UNIT 5 — THE BUILD ═══════════════════════ -->

## What This Does

TakeMeter is a text classifier trained on public posts from r/JRPG. It sorts each post into one of three categories: `analysis`, `request`, or `community_discussion`. I chose these labels because the community has a mix of detailed reviews and arguments, people asking for recommendations or help, and lighter discussion or reaction posts. The goal is to see whether a small fine-tuned model can learn those differences from the way the posts are written.

---

## Label Taxonomy

### `analysis`

**Definition:** A post that develops an opinion, review, or argument about a game or JRPG topic using specific reasons, mechanics, story details, or comparisons.

**Example 1:**
> Romancing SaGa 1 Minstrel Song Review

**Example 2:**
> Nocturne is one of the best games this year

### `request`

**Definition:** A post whose main goal is getting a recommendation, answer, choice, or practical help from other users.

**Example 1:**
> Games like Xenogears, Nier/Drakengard, Stellar Blade

**Example 2:**
> Tactics Ogre vs Final Fantasy Tactics. Which to play first.

### `community_discussion`

**Definition:** A post mainly meant to start conversation, share a reaction or appreciation, report community-relevant news, or invite opinions without building a sustained argument or asking for a personal decision.

**Example 1:**
> Who are the biggest benchwarmers in JRPG history?

**Example 2:**
> The disparities between sprites and their portraits always gives me a good laugh

### The hardest boundary

**Which two labels:** `analysis` and `community_discussion`

**The decision rule I used every time:** If the author makes an evaluative claim and supports it with specific reasons about gameplay, story, mechanics, design, or comparisons, I label it `analysis`. If the post mainly shares a reaction, appreciation, joke, news item, or open-ended prompt without developing that reasoning, I label it `community_discussion`.

For `request`, I use one extra rule: if the author needs the comments to give them an answer, recommendation, or decision, I label it `request`, even if the post includes a lot of context.



---

## The Dataset

**Where the posts came from:** Public posts from r/JRPG.

**How I labelled them:** I labelled the first 40 posts cold without AI help. After that, I used AI to pre-label the remaining posts, then read and reviewed every suggested label myself before keeping it. I also checked the final CSV for duplicate rows and formatting problems before training.

**Counts per label:**

| Label | Count | Share |
|---|---:|---:|
| `analysis` | 50 | 25.0% |
| `request` | 61 | 30.5% |
| `community_discussion` | 89 | 44.5% |
| **Total** | **200** | **100%** |

**Three hard cases**

**1.**
> *The post:* Tactics Ogre vs Final Fantasy Tactics. Which to play first.
>
> *Could have been:* `analysis` or `request`
>
> *I chose, because:* `request`. The post includes a lot of detailed opinions and comparisons, but the main purpose is still to get help deciding which game to play first.

**2.**
> *The post:* Nocturne is one of the best games this year.
>
> *Could have been:* `analysis` or `community_discussion`
>
> *I chose, because:* `analysis`. The title sounds like a reaction post, but the body explains the claim with specific points about the combat, music, visuals, level design, and story.

**3.**
> *The post:* What are the best linear JRPGs in your opinion?
>
> *Could have been:* `request` or `community_discussion`
>
> *I chose, because:* `request`. Even though it invites opinions from the community, the poster is mainly asking people to suggest games that match a specific preference.

---

## The Training Run

**Base model:** `distilbert-base-uncased`

**Settings:** 3 epochs, learning rate `2e-5`, batch size 16, seed 42, and max length 128 tokens.

**Anything I changed from the defaults, and why:** I kept the default training hyperparameters. I only changed the label list to match my taxonomy: `analysis`, `request`, and `community_discussion`.

**Split sizes:** 139 training posts, 31 validation posts, and 30 test posts. The test split contains 8 `analysis`, 9 `request`, and 13 `community_discussion` posts.

**Device:** CPU on Windows using PyTorch 2.14.1+cpu.

---

## How I Used AI

**Moment 1**

- *What I asked for:* I shared the candidate distinctions I noticed while reading r/JRPG and asked for help stress-testing where the boundaries were unclear.
- *What came back:* AI suggested a three-label taxonomy and pointed out that a long post can still be a `request` if its main purpose is getting an answer or recommendation.
- *What I changed:* I kept the three labels `analysis`, `request`, and `community_discussion`, and used a specific decision rule to separate detailed analysis from lighter discussion.

**Moment 2**

- *What I asked for:* After I labelled the first 40 posts myself, I used AI to pre-label later batches so I could review them faster.
- *What came back:* AI returned suggested labels, flagged ambiguous cases, and helped catch duplicate rows and CSV quoting problems.
- *What I changed:* I read and reviewed every pre-labelled row, kept or corrected the label based on my taxonomy, and used a cleaned 200-row CSV for training.

**Pre-labelling disclosure:** The first 40 rows were labelled cold by me. The remaining 160 rows were AI pre-labelled and then reviewed by me before training.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

| Measure | Baseline | Trained | Difference |
|---|---:|---:|---:|
| Overall accuracy | 0.333 | 0.433 | +0.100 |
| Macro F1 | 0.262 | 0.202 | -0.061 |
| F1 — `analysis` | 0.444 | 0.000 | -0.444 |
| F1 — `community_discussion` | 0.143 | 0.605 | +0.462 |
| F1 — `request` | 0.200 | 0.000 | -0.200 |

**What I predicted before I looked:** I expected the trained model to do best on `request` posts because recommendation and help-seeking language seemed like something it could learn from my labeled examples.

**What the gap actually means:** Fine-tuning improved overall accuracy by 10 percentage points, but it did not improve the classifier evenly. The trained model became much better at `community_discussion` while performing worse than the zero-shot baseline on both `analysis` and `request`, which is why its macro F1 dropped even though its overall accuracy increased.

---

## Run Log — Before

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---:|---:|---:|---|
| 1. Overall accuracy | ≥ 0.80 | 0.433 | 0.433 | 0.533 | MISSED |
| 2. Every label F1 | ≥ 0.70 | min 0.000 | min 0.000 | min 0.000 | MISSED |
| 3. Dataset balance | largest label ≤ 65% | 44.5% | 44.5% | 44.5% | MET |
| 4. Labeling consistency | ≥ 80% agreement | pending agreement check | pending | pending | PENDING |
| 5. Confidence usefulness | most-confident third ≥ least-confident third + 15 pts | +70 pts | +40 pts | +30 pts | MET |

Accuracy moved 0.100 across the three seeds. That is right at the assignment's warning threshold, so the exact score is sensitive to the split even though all three runs show the same broader problem.

### Confusion matrix

| true \\ predicted | analysis | request | community_discussion |
|---|---:|---:|---:|
| **analysis** | 0 | 0 | 8 |
| **request** | 0 | 0 | 9 |
| **community_discussion** | 0 | 0 | 13 |

**My biggest off-diagonal number, and what it means:** 9 real `request` posts were called `community_discussion`, while no `community_discussion` posts were called `request`. On seed 42 the model leaned completely toward `community_discussion` instead of learning the request/discussion boundary.

---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 |  |  |  |  |
| 2 |  |  |  |  |
| 3 |  |  |  |  |
| 4 |  |  |  |  |
| 5 |  |  |  |  |

**Diagnoses**

<!-- For each miss: the cause, and how you know. The four common causes are:
     too few examples for a label, a boundary you applied inconsistently, a
     genuinely hard label pair, and a task the model can't reach from this
     much data.

     ⚠️ Use your agreement report as evidence. It is the only instrument you
     have that can tell a LABELLING problem from a MODEL problem, and this
     section is graded on whether you used it that way. -->



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate:** ___ / 30 = ___%

<!-- Nobody grades this number. A 60% who argues every disagreement from the
     stated rules beats a 95% who wrote "staff was right" nine times. Several
     of the 30 were chosen because they're genuinely ambiguous — you should be
     winning some of these. -->

**Disagreements**

<!-- Three lines each: the post, both labels, and who you think is right and
     why — grounded in the staff definitions you were both applying.

     Then sort each into one of three piles:
       (a) the rule covered it and I applied it loosely → a consistency problem
       (b) the rule genuinely doesn't say               → a gap in the definitions
       (c) the rule is ambiguous here and my reading is defensible → argue it.
           This is a legitimate win.

     Pile (a) is the one that matters most for your diagnosis: if you applied a
     written rule two different ways on 30 posts, that is direct evidence about
     what you did across your own 200. -->

**1.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**2.**
> *The post:*
>
> *Staff said / I said:*
>
> *My call, and why:*
>
> *Which pile:*

**What the pattern in my disagreements tells me:**



---

## The Improvement

**What I changed:**

**Which diagnosis pointed at it:**

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

**Did it help, and how do I know:**

<!-- If it didn't, say so. Relabelling that didn't help is a genuinely
     interesting result and earns full credit. -->



---

## What's Still Broken

<!-- For each criterion still missed: what you'd do, and why you stopped. -->



**The gap between what I meant my labels to capture and what the model
learned:**
<!-- Two sentences. Your confusion matrix is the evidence. -->



<!-- ═════════════════════════════════════════════════════════════════════

     SUBMISSION CHECKLIST — unit 5

       [ ] criteria.md has five numbered criteria, each naming a NUMBER
       [ ] Each has a reason underneath tied to your data or taxonomy
       [ ] labels.csv: at least 150 rows, text/label/note, ONE file not split
       [ ] No label above 70%
       [ ] All five unit 5 sections have real content
       [ ] Label Taxonomy includes the decision rule for your hardest boundary
       [ ] The Dataset includes three hard cases
       [ ] results.json and test_split.csv committed (the notebook does this)
       [ ] At least four new commits
       [ ] Repository URL submitted — WRITE IT DOWN

     SUBMISSION CHECKLIST — unit 6

       [ ] Baseline vs. Trained table, with your prediction written beforehand
       [ ] Run Log — Before, five criteria across three seeds
       [ ] Confusion matrix TYPED AS A MARKDOWN TABLE
       [ ] A verdict on every criterion
       [ ] A diagnosis for every miss, using the agreement report as evidence
       [ ] Agreement Report with every disagreement adjudicated
       [ ] One improvement, with Run Log — After
       [ ] What's Still Broken
       [ ] results_three_seeds_before.json, results_three_seeds_after.json,
           baseline_results.json and
           agreement_results.json committed
       [ ] At least four new commits
       [ ] The SAME repository URL as last unit

     Do not delete and recreate this repository.
     ═════════════════════════════════════════════════════════════════════ -->

---

📖 **How to run this project: [RUNNING.md](RUNNING.md)**
