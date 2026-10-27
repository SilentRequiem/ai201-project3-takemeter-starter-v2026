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

**Moment 3**

- *What I asked for:* I shared my three-seed before and after results and asked AI to compare them against the criteria without changing the targets.
- *What came back:* AI calculated the before/after pattern, pointed out that five epochs improved average accuracy and macro F1 on two seeds, but that seed 7 still collapsed and `analysis` remained the weakest label.
- *What I changed:* I did not change the experiment after seeing that analysis. I used it to write the verdicts, diagnosis, and What's Still Broken sections while keeping the one-improvement rule intact.

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
| 4. Labeling consistency | ≥ 80% agreement | 100% | 100% | 100% | MET |
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

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.80 | MISSED | Accuracy was 0.433, 0.433, and 0.533. None of the three seeds reached 0.80. |
| 2 | Per-label performance | Every label F1 ≥ 0.70 | MISSED | The minimum label F1 was 0.000 in all three seeds. `analysis` had F1 0.000 every time, and `request` was 0.000, 0.000, and 0.429. |
| 3 | Dataset balance | No label above 65% | MET | The largest class, `community_discussion`, is 89 of 200 posts, or 44.5%. |
| 4 | Labeling consistency | Agreement ≥ 80% | MET | I matched the staff labels on all 30 staff-taxonomy posts, for 100% agreement. |
| 5 | Confidence usefulness | Most-confident third at least 15 points better than least-confident third | MET | The confidence gaps were +70, +40, and +30 percentage points across seeds 42, 7, and 2024. |

**Diagnoses**

**Criterion 1 — MISSED:** The model did not learn all three classes well enough from this training setup. The confusion matrix for seed 42 shows every test example being predicted as `community_discussion`, and seed 7 shows the same pattern. Seed 2024 improves to 0.533 accuracy but still never predicts `analysis` correctly. My 100% staff agreement does not point to a general rule-application consistency problem, so the stronger evidence is that the trained classifier is under-learning the minority classes.

**Criterion 2 — MISSED:** `analysis` is the clearest failure. Its F1 is 0.000 on all three seeds, even though the dataset contains 50 analysis examples. `request` is also weak, with F1 0.000 on two seeds and 0.429 on seed 2024. The seed-42 matrix shows 8 real `analysis` posts and 9 real `request` posts all being absorbed into `community_discussion`. Because the agreement check was 30/30 under an external written taxonomy, I do not have evidence that inconsistent rule-following is the main cause. The more likely issue is that 200 examples and the current three-epoch training setup were not enough for the model to separate these boundaries reliably.

No diagnosis is needed for Criteria 3, 4, or 5 because they were met.

---

## Agreement Report

**Agreement rate:** 30 / 30 = 100%

I labeled all 30 posts using the staff taxonomy before comparing against the staff answer key. My labels matched staff on every post, so there were no disagreements to adjudicate.

**Disagreements:** None.

**What the pattern in my disagreements tells me:** There was no disagreement pattern because I matched all 30 staff labels. The staff rules were specific enough for me to apply consistently, especially the distinction between a load-bearing fact for `analysis`, an unsupported general claim for `hot_take`, and an immediate personal response for `reaction`. This does not prove my JRPG taxonomy is perfect, but it gives me no evidence that general rule-following inconsistency is what caused the model's poor `analysis` and `request` scores.

---

## The Improvement

**What I changed:** I changed one training setting only: `EPOCHS` from 3 to 5. The learning rate, batch size, max length, labels, taxonomy, dataset, and three seeds stayed unchanged.

**Which diagnosis pointed at it:** The before-run model was collapsing toward `community_discussion`, but my staff agreement check was 30/30, so I did not have evidence that loose rule-following was the main problem. I therefore tested whether the model simply needed more passes through the same 200 examples.

### Run Log — After

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---:|---:|---:|---|
| 1. Overall accuracy | ≥ 0.80 | 0.567 | 0.433 | 0.667 | MISSED |
| 2. Every label F1 | ≥ 0.70 | min 0.000 | min 0.000 | min 0.222 | MISSED |
| 3. Dataset balance | largest label ≤ 65% | 44.5% | 44.5% | 44.5% | MET |
| 4. Labeling consistency | ≥ 80% agreement | 100% | 100% | 100% | MET |
| 5. Confidence usefulness | most-confident third ≥ least-confident third + 15 pts | +60 pts | +60 pts | +50 pts | MET |

**Did it help, and how do I know:** Yes, but only partially. Average accuracy across the three seeds rose from about 0.467 before to 0.556 after, and average macro F1 rose from about 0.258 to 0.398. Seed 42 improved from 0.433 to 0.567 accuracy and seed 2024 improved from 0.533 to 0.667, while seed 7 stayed at 0.433 and still predicted every test post as `community_discussion`. The change therefore improved the model on two splits, but it did not make the classifier reliable across seeds.

The accuracy spread after the change was about 0.233, compared with 0.100 before. That larger spread means the five-epoch model is more sensitive to which examples land in the train and test splits.

---

## What's Still Broken

Criteria 1 and 2 are still missed. Overall accuracy never reached the 0.80 target on any seed, and no seed reached F1 ≥ 0.70 for every label. The biggest remaining problem is `analysis`: its F1 after the change was 0.000, 0.000, and 0.222. `request` improved on seeds 42 and 2024, but seed 7 still collapsed completely.

If I continued, I would focus on the `analysis` class rather than changing several settings at once. I would collect more clearly reasoned `analysis` examples and inspect borderline `analysis` versus `community_discussion` rows, then measure that as a separate experiment. I stopped after the five-epoch rerun because the assignment asks for one measured improvement, and adding another change would make it impossible to attribute the result cleanly.

**The gap between what I meant my labels to capture and what the model learned:** I intended `analysis` to mean a supported opinion or argument, `request` to mean asking for help or recommendations, and `community_discussion` to cover lighter conversation and reactions. The confusion matrices show that the model learned `community_discussion` most strongly and only began learning `request` on some splits, while it still failed to reliably learn `analysis`.

---

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
