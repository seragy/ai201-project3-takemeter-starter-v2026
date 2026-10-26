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

> **Authorship.** Claude (Anthropic's AI assistant, in Claude Code) did the
> work in this repository at Youssef Serag's request: the taxonomy, data
> collection, all 303 labels and their review, the criteria, every training
> run, and the writing in this README. Where the text says "I", it means
> Claude. No row in `labels.csv` was labelled by a human. The details are
> under [How I Used AI](#how-i-used-ai).

## What This Does

<!-- Your community, and what your classifier sorts posts into. Three or four
     sentences. -->

TakeMeter sorts comments from **r/soccer**, Reddit's main football
discussion community, by what kind of contribution they make. The comments
come from a public snapshot of r/soccer threads (roughly 2016 to 2023:
post-match threads, goal clips, stats posts, news, quotes and discussion).
Each comment is sorted into one of three labels: `analysis` (a point backed
by something checkable), `take` (a sincere opinion with nothing checkable
behind it), or `reaction` (a joke, feeling or question that isn't arguing
anything).

**What I noticed reading 40 comments before designing anything:**

1. Some comments point at something you could check — a minute, a
   substitution, a stat, a result ("Rashford did more in 5mins than any other
   England player…"). Most don't.
2. A large share make no claim at all — they're jokes, sarcasm, or pure
   feeling ("I am going to get fucking wasted", "At least Suarez didn't start
   biting people…").
3. Some deliver a confident verdict with nothing behind it ("Platini and
   sarkozy both belong in jail with blatter", "Faes is very decent at
   football").
4. A noticeable minority aren't about football at all (concert trips,
   geopolitics in a news thread). These are out of scope and were not
   collected.



---

## Label Taxonomy

<!-- Each label: a one-sentence definition and two real examples from your
     reading. Then your decision rule for the hardest boundary.

     The decision rule is worth a point on its own and it's the thing most
     people leave out. Every taxonomy has a hardest boundary. Name yours. -->

Every comment gets exactly one label. Ask two questions, in this order:

1. **Is it mainly a joke, sarcasm, a feeling, or a question?** → `reaction`,
   even if it happens to contain a fact.
2. **Otherwise, does its point rest on a specific, checkable detail?** →
   `analysis`. If not → `take`.

### `analysis`

**Definition:** Makes a point about football that rests on at least one
specific, checkable detail — a number, date, scoreline, record, quote,
transfer fee, or a concrete moment in a match.

**Example 1:**
> I'm not so sure. He hasn't played most of those in the last few years.
> Edit: He has played 4 friendlies in total since 2017, scoring 3 goals in
> those games. Against Sweden (1 goal), Egypt (2 goals), the Netherlands, and
> Algeria. On the other hand in the same time period he played 25 official
> games scoring 30 goals in total.

**Example 2:**
> I mean, neither Pele nor Maradona have won the Balloon D'Or. Non-Europeans
> weren't allowed to win it until 1995.

### `take`

**Definition:** A sincere opinion, verdict or prediction about football,
offered without any specific, checkable detail behind it.

**Example 1:**
> The only reason he hasn't been contesting for the ballon d'Or is because of
> injuries. When he's healthy he is literally unplayable and not a single
> player in the world outside of Messi and Ronaldo can touch him in terms of
> his contribution on the field

**Example 2:**
> When I think of great players from the last 10 or so years - obviously
> Messi and Ronaldo will come to mind first. But I feel like the third most
> enigmatic player has been Zlatan. Dude is an icon.

### `reaction`

**Definition:** A joke, sarcastic line, feeling or question that doesn't
sincerely argue a point.

**Example 1:**
> #THEY'RE GOING HOME LADS

**Example 2:**
> Not to mention all the MLS Cups Messi will be winning 8 years from now

### The hardest boundary

**Which two labels:** `take` vs `analysis`.

**The borderline post that found it:**
> On Par with Casillas vs Robben

It names a real, checkable save — but the save is what the poster is
comparing *to*, not evidence *for* the comparison.

**The decision rule I used every time:**

A detail only counts if someone could look it up or rewind to it: a number,
date, scoreline, named match or minute, quote, fee, or a described on-pitch
moment. **And it has to support the point, not just be its subject.**

- Doesn't count: personal experience ("I've played CB for years"),
  "everyone knows", hyperbole ("picked 9/10 times"), and whole-match
  summaries ("Liverpool created nothing the entire game").
- Tone never matters. A furious post with a checkable fact is `analysis`; a
  calm, thoughtful post with none is `take`.

**Second rule, `reaction` vs the other two:** if the main purpose is the
joke, it's `reaction` — even when the joke is built out of true facts.

### Stress test

Before labelling, I had Claude write eight posts that sit on these
boundaries, and classified each with the rules above:

| # | Post | Label | What it tested |
|---|---|---|---|
| 1 | "Pochettino is a fraud. 0 trophies in 5 years at Spurs says it all." | `analysis` | Heated tone + checkable fact |
| 2 | "Pochettino is a fraud and everyone who's watched Spurs knows it." | `take` | "Everyone knows" isn't support |
| 3 | "Ah yes, Spurs, famous for their trophy cabinet 🏆" | `reaction` | Sarcasm with an implied claim |
| 4 | "Haaland scored 36 league goals last season. 40 next year, easy." | `analysis` | Prediction backed by a number |
| 5 | "I've played centre back 15 years and nobody defends that run. Not a foul." | `take` | Personal experience isn't checkable |
| 6 | "Arsenal bottled it again, awful for 90 minutes." | `take` | Whole-match summary |
| 7 | "Saliba stepped up at 63' and left Jesus offside — that's the whole goal." | `analysis` | Concrete match moment |
| 8 | "Can't believe Messi is 36 and still doing this 😭" | `reaction` | Feeling that contains a fact |

Posts 4 and 8 were the ones my first draft couldn't separate: both contain a
number. That's what produced the **order** of the two questions (check for
joke/feeling first) and the clause "it has to support the point".



---

## The Dataset

<!-- Where you collected from, how you labelled, your counts, and three hard
     cases. -->

**Where the posts came from:** Public r/soccer comments, taken from the
[`singhala/reddit_soccer`](https://huggingface.co/datasets/singhala/reddit_soccer)
dataset on Hugging Face (BSD licence). It's a snapshot of 98,710 comments
across 995 r/soccer threads, roughly 2016–2023. Reddit blocks scripted access
from my machine, so I used this snapshot instead of scraping.

How I picked 303 out of 98,710:

- **Removed:** deleted or removed comments, bot and mirror comments, anything
  with a link, comments under 20 or over 600 characters, and exact duplicates.
- **Kept threads:** post-match threads, goal clips, stats, news, quotes,
  transfers and discussion.
- **Drew a 380-comment pool from three buckets:** 150 comments containing a
  number and over 80 characters, 110 long comments (over 200 characters) with
  no number, and 120 short ones. A plain random sample would have been mostly
  `reaction` with very little `analysis`. The two non-number buckets are there
  so that "long" or "has a digit" doesn't become a stand-in for `analysis`.
- **Dropped 18 off-topic comments** (hotel stories, basketball, a TV show,
  geopolitics with no football in it).
- **Randomly downsampled `reaction` from 169 to 110** to even out the labels.

**How I labelled them:**

- **The 20 rows at the top were meant to be labelled cold by a human.** They
  weren't. Claude labelled them, without a pre-label to look at, and the note
  column says `cold slot: labelled by Claude without a pre-label (no human
  cold set)`. This dataset has **no human-labelled rows**.
- **The other 283 were pre-labelled by Claude** using the definitions and
  decision rules above. Claude then reviewed every one against the same rules
  in a second pass. The note column says `pre-labelled (Claude), reviewed by
  Claude`. Three labels changed on review, each noted in its row:
  - row 33 (the Instagram 🥴 comment): `analysis` → `reaction`
  - row 47 (US vs Netherlands, "played at a 6 most of the match"): `analysis`
    → `take`, as a whole-match summary
  - row 213 (the Azteca saw Pelé and Maradona lift the cup): `analysis` →
    `reaction`, a feeling built from a fact

  A model reviewing its own labels doesn't give an independent check. The
  agreement check against the staff set (Unit 6) is the only outside
  measurement of this labelling.



**Counts per label:**

| Label | Count | Share |
|---|---|---|
| `reaction` | 113 | 37% |
| `analysis` | 95 | 31% |
| `take` | 95 | 31% |
| **Total** | 303 | 100% |

**Hard cases** (marked `hard:` in the note column)

**1.**
> *The post:* "Alexis Sanchez went from an underperforming club that gets
> knocked out by European giants in the round of 16 to an underperforming club
> that gets knocked out by European minnows in the round of 16"
>
> *Could have been:* `reaction` or `analysis`
>
> *I chose, because:* `reaction`. Every part of it is true and checkable, but
> it exists for the punchline. Rule 1 comes first: if the main purpose is the
> joke, it's `reaction`.

**2.**
> *The post:* "clearly shows you don't watch us lmao. Man has been made out of
> glass this season. He gets picked 9/10 times when he's available lol."
>
> *Could have been:* `analysis` or `take`
>
> *I chose, because:* `take`. "9/10 times" looks like a stat, but it's a figure
> of speech; nobody could look it up. Hyperbole doesn't count as a checkable
> detail.

**3.**
> *The post:* "Watford really were fantastic. Wasn't some flukey win, they
> were properly dominating in terms of chance creation and defended superbly.
> Liverpool created nothing the entire game, while Watford could've easily
> scored a couple more."
>
> *Could have been:* `analysis` or `take`
>
> *I chose, because:* `take`. It's about one specific match, which makes it
> feel like analysis. But it never names a moment, a minute or a number, only
> a summary of 90 minutes. Whole-match summaries don't count.

**4.**
> *The post:* "I just knew it that Italy was going to win after this. The
> vibes in both camps and their captains was totally different. Add to that
> Luis Enrique's comments about how he doesn't get involved in selecting
> penalty takers"
>
> *Could have been:* `take` or `analysis`
>
> *I chose, because:* `analysis`, narrowly. Most of it is vibes, but the last
> line points at a real, checkable quote that supports the prediction. One
> checkable detail that supports the point is enough.

---

## The Training Run

<!-- Your starting model, your settings, and anything you changed and why. -->

**Base model:** `distilbert-base-uncased`

**Device:** `mps` (Apple Silicon GPU), torch 2.14.1. Recorded in
`results.json` too. Training took about 21 seconds.

**Settings:** 3 epochs, learning rate 2e-5, batch size 16, max length 128
tokens, seed 42.

**Anything I changed from the defaults, and why:** Nothing. Comments were
capped at 600 characters during collection, which is well under 128 tokens
for almost all of them, so the default length doesn't cut much off.

**Split sizes:** train 211 · val 46 · test 46, stratified, seed 42.

| Label | Train | Val | Test |
|---|---|---|---|
| `analysis` | 66 | 14 | 15 |
| `take` | 66 | 15 | 14 |
| `reaction` | 79 | 17 | 17 |

No label has fewer than 8 in the test split. Still, with 14–17 per label,
one post is worth about 0.06 of recall.

**First result (seed 42):** accuracy 0.522, macro F1 0.488. Per-label F1:
`analysis` 0.500, `take` 0.333, `reaction` 0.632. Training loss only went
from 1.06 to 0.99 across 3 epochs (42 optimiser steps). Per the notebook's
instructions, I didn't change anything in response to these numbers before
Unit 6.


---

## How I Used AI

<!-- Two specific moments — what you asked, what came back, what you changed.

     ⚠️ Plus disclosure of any pre-labelling. If you had a model pre-label a
     batch and then read and corrected every one, say that. It's an allowed
     workflow and disclosing it costs you nothing. Not disclosing it is the
     problem. -->

Written by Claude. Youssef asked Claude to complete Units 5 and 6 end to
end, and to say so in the submission. So this section describes Claude's own
decisions. "What I changed" means what Claude changed in response to what it
found.

**Moment 1: the taxonomy had a gap**

- *What was asked:* turn 40 read-through comments into three labels with a
  decision rule.
- *What came back:* the first draft had no home for plain factual comments,
  such as "Napoli are the 4th different champion in 4 years". They weren't
  opinions and they weren't jokes.
- *What I changed:* I folded factual comments into `analysis` and renamed the
  third label `reaction`, so it covers jokes, feelings and bare questions.
  Then the stress test showed that posts 4 and 8 both contained a number but
  belonged to different labels. That produced the ordered two-question rule:
  check for a joke or feeling first, then ask whether the detail *supports*
  the point.

**Moment 2: reviewing my own pre-labels**

- *What was asked:* review all 283 pre-labels and label the 20 empty rows.
- *What came back:* applying the written rules row by row turned up three
  pre-labels that broke them. Row 213 ("cool to think the Azteca saw both Pelé
  and Maradona lift a world cup") had been labelled `analysis` because it
  contains a checkable fact. Rule 1 says a feeling comes first, even when it's
  built from a fact.
- *What I changed:* three labels (rows 33, 47 and 213, listed under The
  Dataset). I also noted that a second pass by the same model is weak
  evidence: it probably shares the first pass's blind spots. That makes the
  staff agreement check the only real outside test of these labels.

**AI-use disclosure:** Claude did all of the following:

- Picked the dataset and built the candidate pool.
- Wrote the taxonomy and the eight stress-test posts.
- Labelled all 303 rows (283 pre-labelled and then reviewed by Claude, plus 20
  labelled directly by Claude in the slots meant for human cold labels).
- Wrote the five criteria before training, and ran every training run, the
  baseline and the agreement script.
- Labelled the 30 staff posts in `my_staff_labels.csv`.
- Wrote this README.

No human labels are in this repository.

<!-- ═══════════════════════ UNIT 6 — THE TEST ═══════════════════════

     Don't fill these in during unit 5.
     ═══════════════════════════════════════════════════════════════════ -->

---

## Baseline vs. Trained

<!-- Both models on the same posts. `python baseline.py --trained results.json`
     prints this table for you. -->

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy |  |  |  |
| Macro F1 |  |  |  |
| F1 — `label_one` |  |  |  |
| F1 — `label_two` |  |  |  |

**What I predicted before I looked:**

Written before running `baseline.py`. One caveat: the trained model's seed-42
numbers (accuracy 0.522) were already printed in Unit 5, so only the baseline
half of this is a real prediction.

I expect the zero-shot baseline (`bart-large-mnli`, given my three definitions)
to score about **0.40 accuracy**, barely above the 0.37 majority-class rate.
It should do best on `reaction`, where the posts look different on the
surface (short, exclamation marks, "lol"), and close to zero on the `take` vs
`analysis` split. An entailment model asked "does this make a point resting
on a checkable detail?" will say yes to anything with a number in it, and say
yes or no to everything else more or less at random. So I expect the fine-tuned model to win by
about 0.10–0.15.

**What the gap actually means:**
<!-- If the baseline matched your trained model, your fine-tuning added
     nothing — and that is a real finding, not a failure. Say it plainly. -->



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1.  |  |  |  |  |  |
| 2.  |  |  |  |  |  |
| 3.  |  |  |  |  |  |
| 4.  |  |  |  |  |  |
| 5.  |  |  |  |  |  |

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

| true \ predicted |  |  |  |
|---|---|---|---|
| **** |  |  |  |
| **** |  |  |  |
| **** |  |  |  |

**My biggest off-diagonal number, and what it means:**
<!-- Not "the model made mistakes" — WHICH boundary it didn't learn, and which
     direction. "7 real analysis posts were called hot_take and only 3 went the
     other way" is a direction, not just an error rate. -->



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
