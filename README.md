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

Both models on the same 46 posts in `test_split.csv` (seed 42). The baseline
is `facebook/bart-large-mnli` zero-shot, with my definitions from
`label_definitions.txt` as the prompt.

| Measure | Baseline | Trained | Difference |
|---|---|---|---|
| Overall accuracy | 0.348 | 0.522 | +0.174 |
| Macro F1 | 0.244 | 0.488 | +0.245 |
| F1 — `analysis` | 0.500 | 0.500 | -0.000 |
| F1 — `reaction` | 0.231 | 0.632 | +0.401 |
| F1 — `take` | 0.000 | 0.333 | +0.333 |

I also ran the baseline once with **bare label names** (no definitions) to
see what the definitions are worth. That run also scored **0.348** accuracy,
with macro F1 0.263. It isn't the reported baseline, and its JSON isn't
committed.

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

The prediction was half right. The overall gap (+0.17) landed just above the
0.10–0.15 I guessed. But I had `reaction` backwards: I expected it to be the
baseline's best label, and it was close to its worst. Looking at its
predictions explains why. **With my definitions, the baseline called 37 of 46
posts `analysis`.** That includes 14 of the 17 `reaction` posts. With bare
names, it called 36 of 46 posts `take`. Either way, the zero-shot model
mostly predicted one label. Its `analysis` F1 of 0.500 is mostly an artefact
of predicting `analysis` for nearly everything. Its accuracy (0.348) is
*below* always guessing `reaction` (17/46 = 0.370).

The definitions changed *which* label it collapsed onto, but not how accurate
it was (0.348 both times). For a general entailment model, a sentence like
"rests on a specific, checkable detail" isn't something it can test a post
against.

So fine-tuning added real value, nearly all of it on `reaction` and `take`.
Learning `reaction` from 79 training examples was the biggest win (+0.40 F1).
On `analysis`, the fine-tuned model is no better than the baseline. That's the
boundary my taxonomy is built around, and the model hasn't learned it yet.



---

## Run Log — Before

<!-- Five criteria across three seeds. The notebook's section 6 prints the
     spread table; the Target and Verdict columns are yours. -->

Defaults: 3 epochs, lr 2e-5, batch 16, max length 128, on `mps`.
`results_three_seeds_before.json`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.60 every seed | 0.522 | 0.565 | 0.609 | **MISSED** |
| 2. Lowest per-label F1 | ≥ 0.50 every seed | 0.333 (`take`) | 0.381 (`analysis`) | 0.364 (`take`) | **MISSED** |
| 3. Accuracy gain over zero-shot baseline (same split) | ≥ +0.15 every seed | +0.174 (0.522 vs 0.348) | +0.217 (0.565 vs 0.348) | +0.239 (0.609 vs 0.370) | **MET** |
| 4. Label shares in `labels.csv` | every label 25–40% | 37 / 31 / 31% | same file | same file | **MET** |
| 5. Accuracy, most-confident third minus least-confident third | ≥ +0.15 every seed | +0.467 (0.800 vs 0.333) | +0.467 (0.800 vs 0.333) | +0.533 (0.867 vs 0.333) | **MET** |

The notebook's spread table:

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.522 | 0.565 | 0.609 | 0.087 |
| Macro F1 | 0.488 | 0.535 | 0.578 | 0.090 |
| F1 — `analysis` | 0.500 | 0.381 | 0.688 | 0.307 |
| F1 — `take` | 0.333 | 0.571 | 0.364 | 0.238 |
| F1 — `reaction` | 0.632 | 0.651 | 0.684 | 0.053 |

Accuracy moved 0.087 across seeds, which is under the notebook's 10-point
warning line. The per-label F1s for `analysis` and `take` moved by 0.24–0.31,
which is what about 15 test posts per label does. For criterion 3, I ran the
baseline on each seed's own test split (rebuilt with the notebook's
`split_dataset`). The seed-42 split is identical to the committed
`test_split.csv`.

### Confusion matrix

<!-- ⚠️ TYPED AS A MARKDOWN TABLE. The notebook prints one ready to paste.
     An image of a matrix earns nothing. -->

Seed 42, as the notebook prints it (46 test posts):

| true \ predicted | analysis | take | reaction |
|---|---|---|---|
| **analysis** | 9 | 1 | 5 |
| **take** | 7 | 3 | 4 |
| **reaction** | 5 | 0 | 12 |

All three seeds added together (138 test predictions). One seed's matrix has
cells of 0–7, so the sum gives a steadier picture:

| true \ predicted | analysis | take | reaction |
|---|---|---|---|
| **analysis** | 24 | 7 | 13 |
| **take** | 12 | 15 | 16 |
| **reaction** | 9 | 3 | 39 |

**My biggest off-diagonal number, and what it means:**

On seed 42, **7 real `take` posts were called `analysis` and only 1 went the
other way.** The model treats "sounds like an argument about football" as
`analysis`. It hasn't learned my real rule, that the post needs a checkable
detail that *supports* the point. That's the hardest boundary from Unit 5,
and the model fails it in the direction you'd expect: confident, specific
language gets read as evidence.

Across all three seeds, the biggest off-diagonal cell is a different one:
**16 `take` → `reaction`**, then 13 `analysis` → `reaction`. `take` is the
label the model recognises least (15 of 43, 35% recall). Its errors split
between the other two labels rather than going one way, while `reaction`
pulls in errors from both. That's what a model does when it has learned the
easy feature (short and informal → `reaction`) and not much else.



---

## Verdicts and Diagnoses

<!-- MET or MISSED against LAST UNIT's target. The target has to hold across
     all three seeds, not turn up sometimes. -->

| # | Criterion | Target | Verdict | How I decided |
|---|---|---|---|---|
| 1 | Overall accuracy | ≥ 0.60 on every seed | **MISSED** | 0.522 and 0.565 are below it. Only seed 2024 (0.609) clears it |
| 2 | Every label's F1 | ≥ 0.50 on every seed | **MISSED** | Some label falls below on every seed: `take` 0.333, `analysis` 0.381, `take` 0.364 |
| 3 | Gain over the zero-shot baseline | ≥ +0.15 on every seed | **MET** | +0.174, +0.217, +0.239, each against the baseline on the same split |
| 4 | Label balance | every label 25–40% | **MET** | 113 / 95 / 95 of 303 = 37 / 31 / 31% |
| 5 | Confident third beats unsure third | ≥ +0.15 on every seed | **MET** | +0.47, +0.47, +0.53. This one is less impressive than it looks (see diagnosis 3) |

**Diagnoses**

**1. Criteria 1 and 2 were missed mainly because the model is underfit, not
because it was trained on too few posts.** The evidence:

- Training loss ended at 0.97–1.02 on every seed. A model that always guesses
  uniformly over three labels scores ln 3 ≈ 1.10, so three epochs barely moved
  it.
- Validation loss was still falling at epoch 3 on all three seeds
  (seed 42: 1.072 → 1.039 → 1.023).
- The model's average confidence in its own top guess was 0.38–0.41, barely
  above the 0.33 of a coin with three sides.
- 211 training posts × 3 epochs ÷ batch 16 is only 42 optimiser steps at lr
  2e-5.

This is a model that hasn't finished learning, not one that has hit the limit
of the data. It's the cause I can test most cheaply, so it's the one The
Improvement targets.

**2. Criterion 2 (`take` and `analysis`) also sits on a genuinely hard
boundary. I can't yet rule out that I applied it inconsistently.** The
summed matrix shows `take` with the worst recall (35%). `take` ↔ `analysis`
confusions run 12 / 7 in total, 7 / 1 on seed 42.

This is where the agreement report should separate a labelling problem from
a model problem. A disagreement with staff on their `analysis` / `hot_take`
rule, combined with the same confusion in the matrix, would point at my
labels. But I don't have `data/staff_labels.csv` from the TF yet (see the
Agreement Report), so that test hasn't run.

The only consistency evidence I have is weak. On review, I changed 3 of my
own 283 pre-labels, and 2 of the 3 were the `analysis` boundary. But the same
model labelling and then reviewing isn't independent. So "the boundary is
hard" is supported by the matrix. "I applied it inconsistently" is
**untested**, not ruled out.

**3. Criterion 5 is met, but on a weak model.** The confident third is right
80–87% of the time and the unsure third 33%. But "confident" here means about
0.45 versus about 0.35 softmax, so the ordering carries information while the
values themselves mean little. I wouldn't send posts to a human using a fixed
confidence cut-off from this model.



---

## Agreement Report

<!-- Your rate against the staff set, and every disagreement adjudicated.

     Remember you labelled these 30 under the STAFF taxonomy in
     data/staff_taxonomy.md, not your own — so every argument below is made
     from those definitions and those decision rules. -->

**Agreement rate: not yet measured.** It needs `data/staff_labels.csv`, which
the TF provides only after the staff set has been labelled. That file isn't in
this repo yet, so `python agreement.py` can't run, and there are no
disagreements to adjudicate.

What is done:

- **`my_staff_labels.csv` is committed:** all 30 posts, text copied exactly
  from `data/staff_posts.csv`, labelled by Claude under the **staff**
  taxonomy in `data/staff_taxonomy.md`, not under TakeMeter's own.
- **The calls I expect staff to contest**, with my reasoning recorded *before*
  seeing their labels:
  - **sp_005, sp_007, sp_010 → `analysis`.** Each argues from the writer's own
    measurements: two years of times at 35 vs 50 miles a week, the same loop
    at 4C vs 24C, three runners within 10 seconds. The staff rule asks
    whether the fact is *load-bearing*, not whether a stranger could look it
    up. In all three, the numbers are the whole argument. The case for
    `hot_take` is that one runner's data is anecdote. Rule 1 doesn't say
    anecdotes are excluded.
  - **sp_014 → `hot_take`.** This is almost word for word the staff's own
    example of a number used as decoration: "times are down 3%… so obviously
    the shoes are the whole story".
  - **sp_019 and sp_020 → `hot_take`.** Both are about the writer's world
    ("nobody actually enjoys the long run", "every injury thread"), but they
    make general claims. Rule 2 says heated or personal-sounding general
    claims are still `hot_take`.
  - **sp_025, sp_027, sp_029 → `reaction`.** Each contains a concrete detail
    (blowing up at 20, a 20-degree lean, the wrong shoe size), but none argues
    anything. They report what just happened to the writer.

**Disagreements:** to be filled in once the staff labels arrive. Each one goes
in the three-line format with a pile (a), (b) or (c).

**What the pattern in my disagreements tells me:** not yet known. This is the
test diagnosis 2 is waiting on. If staff and I disagree on `analysis` vs
`hot_take`, and my matrix confuses `take` and `analysis`, that points at my
labels rather than the model.

---

## The Improvement

**What I changed:** `EPOCHS` from 3 to 10 in section 2 of the notebook. That's
140 optimiser steps instead of 42. Nothing else changed: same labels, lr,
batch size, max length and seeds. The "after" runs were written with
`RUN_LABEL = "after"`.

**Which diagnosis pointed at it:** Diagnosis 1. Training loss ended near the
uniform-guess level (0.97–1.02 against ln 3 ≈ 1.10), validation loss was
still falling at epoch 3 on every seed, and mean confidence was about 0.40.
The model hadn't finished learning. I tested underfitting first because it
was the cheapest cause to test, and the only one I could test before the
staff labels arrive. Relabelling the `take`/`analysis` boundary would be the
next fix, but without the agreement report I'd be changing labels on a guess.

### Run Log — After

10 epochs, everything else unchanged. `results_three_seeds_after.json`.

| Criterion | Target | Seed 42 | Seed 7 | Seed 2024 | Verdict |
|---|---|---|---|---|---|
| 1. Overall accuracy | ≥ 0.60 every seed | 0.696 | 0.565 | 0.739 | **MISSED** (seed 7) |
| 2. Lowest per-label F1 | ≥ 0.50 every seed | 0.667 (`analysis`) | 0.385 (`analysis`) | 0.690 (`take`) | **MISSED** (seed 7) |
| 3. Gain over the zero-shot baseline | ≥ +0.15 every seed | +0.348 | +0.217 | +0.369 | **MET** |
| 4. Label shares | every label 25–40% | 37 / 31 / 31% | same file | same file | **MET** (unchanged) |
| 5. Most-confident third minus least-confident third | ≥ +0.15 every seed | +0.333 (0.800 vs 0.467) | +0.400 (0.800 vs 0.400) | +0.133 (0.800 vs 0.667) | **MISSED** (seed 2024) |

| Measure | Seed 42 | Seed 7 | Seed 2024 | Spread |
|---|---|---|---|---|
| Overall accuracy | 0.696 | 0.565 | 0.739 | 0.174 |
| Macro F1 | 0.699 | 0.548 | 0.738 | 0.190 |
| F1 — `analysis` | 0.667 | 0.385 | 0.759 | 0.374 |
| F1 — `take` | 0.741 | 0.593 | 0.690 | 0.148 |
| F1 — `reaction` | 0.690 | 0.667 | 0.765 | 0.098 |

Confusion matrix, all three seeds added together (before → after on the
diagonal: `analysis` 24 → 28, `take` 15 → 28, `reaction` 39 → 36):

| true \ predicted | analysis | take | reaction |
|---|---|---|---|
| **analysis** | 28 | 8 | 8 |
| **take** | 8 | 28 | 7 |
| **reaction** | 11 | 4 | 36 |

**Did it help, and how do I know:**

Yes on two seeds, and no on the third.

- **Seeds 42 and 2024 improved:** accuracy +0.174 and +0.130, macro F1 +0.21
  and +0.16. They now clear criteria 1 and 2 comfortably.
- **`take` was the biggest winner:** summed recall went from 15/43 to 28/43,
  and the seed-42 `take` → `analysis` leak fell from 7 to 3. So part of what
  looked like a hard boundary in diagnosis 2 was underfitting.
- **Seed 7 didn't move:** accuracy was 0.565 both times, and its `analysis`
  F1 is still 0.385.

No verdict changed from MISSED to MET, because each target has to hold on all
three seeds. And the spread across seeds doubled, from 0.087 to 0.174, which
is past the notebook's 10-point warning line.

Criterion 5 went from MET to MISSED. The model is now more confident overall
(mean 0.71–0.75 when right), but on seed 2024 its unsure third is right 67%
of the time, so the gap between the thirds narrowed to 0.133. The criterion
measures a *gap*, so a better model scored worse on it. That says the
criterion is fragile. I've left it as written rather than revising it after
a miss.



---

## What's Still Broken

**Criteria 1 and 2 still miss, because of seed 7's `analysis` posts.** On that
split, 6 of 14 `analysis` posts were called `reaction`. Reading the 14 shows
why: several *open* like reactions before getting to their checkable detail.

- "Congrats to Napoli! Also this makes 4 seasons in a row…"
- "You kidding me? You never noticed? Dude's chomping on his nails…"
- "The usual Michael Owen bullshit…"
- Hard case 4 (Italy and the "vibes"), which I'd already called `analysis`
  only narrowly.

My rule says to ask "is it mainly a joke or feeling?" first, and a model
reading the opening sentence answers yes. **What I'd do next:** once the staff
labels arrive, run the agreement check. If I disagree with staff on their
`reaction`/`analysis` rule (their rule 3, "recency does not outrank
evidence"), relabel or tighten that boundary across `labels.csv` and re-run.
If I agree with staff, the labels are consistent, and the fix is more
`analysis` examples that open informally. **Why I stopped:** the unit allows
one improvement, and relabelling without the agreement evidence would be
guessing.

**Criterion 5 now misses on seed 2024.** I'd check it against
`mean_when_right` vs `mean_when_wrong` (0.71 vs 0.65 on that seed), not just
the thirds. Over three seeds, the thirds are about 15 posts each, and one
seed is too small a sample to tell a calibration problem from noise. I
stopped there because the criterion has to stand as written.

**The measure itself is unstable.** A 0.174 accuracy spread across seeds
means 46 test posts can't separate a 0.60 model from a 0.70 one. More data,
or cross-validation instead of three single splits, would come before any
further tuning.



**The gap between what I meant my labels to capture and what the model
learned:**

I meant `analysis` to mean "a point supported by a checkable detail",
wherever in the post that detail sits. The model mostly learned "sounds
argumentative from the first line". So `analysis` posts that open with a
joke or a cheer go to `reaction` (11 summed `reaction` ↔ `analysis` errors
after the fix, 8 the other way), while the `take`/`analysis` boundary that
worried me most turned out to be largely learnable with enough training.



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
