# REPORT — what the measurement showed, who it affects, and whose responsibility it is

All numbers are from my own run of `lab06_Kaden.ipynb` (the template `lab06_harness_says_no.ipynb`, renamed) on a Colab T4 GPU
(Qwen2.5-3B-Instruct, greedy decoding, `N_PAIRS = 40`, `SEED = 2026`): 200 pairs, 400 profiles,
400 model calls for the bare agent and 400 for the harness run.

## 1. Measurement read-out

**The bare agent.** 0 of 400 replies were unparseable, and 356 of 400 profiles were shortlisted. Rejected
profiles scored 20-45 (median 40) and shortlisted ones 45-85 (median 75).

**Flips and score differences, each attribute read against its own floor (40 pairs each):**

| Attribute | Flips | Flip rate [95% CI] | Floor | Floor flips | Pairs with different scores | Mean abs. difference [95% CI] | Floor mean abs. difference |
|---|---|---|---|---|---|---|---|
| name_language | 2 | 5.0% [1.4%, 16.5%] | null_name | 0/40, 0.0% [0.0%, 8.8%] | 11/40 | 3.12 [1.38, 5.25] | 2.00 [0.62, 3.62] (7/40) |
| gender_coded_name | 4 | 10.0% [4.0%, 23.1%] | null_name | 0/40 | 6/40 | 1.12 [0.25, 2.13] | 2.00 [0.62, 3.62] (7/40) |
| university | 1 | 2.5% [0.4%, 12.9%] | null_university | 1/40, 2.5% [0.4%, 12.9%] | 10/40 | 2.50 [1.12, 4.12] | 2.12 [0.88, 3.62] (9/40) |

**Direction.**
- Gender-coded name: all 4 flips go the same way; the female-coded member was shortlisted and the
  male-coded one was not (male-coded shortlisted 80.0%, female-coded 90.0% of the 40 + 40 profiles).
  Exact McNemar p = 0.125. The signed score gap is only -0.4 [-1.2, +0.6] for male minus female.
- Name language: Kazakh-language names were favoured in 2 pairs and never disfavoured (McNemar
  p = 0.500); score gaps +0.7 [-1.8, +3.8] for Kazakh, +0.2 [-1.7, +2.1] for Russian, -1.0 [-4.2, +1.9]
  for English names, all crossing zero.
- University: flagship vs regional, 1 flip (flagship favoured), McNemar p = 1.000, but a signed score gap
  of **+2.5 [+1.1, +4.1]** for the flagship member, sign test p = 0.002.

**Which attribute moved the decision, against its floor.** None. Every flip-rate interval overlaps the
interval of its own floor, and every mean absolute difference overlaps its floor's. The notebook says
that an exact McNemar test at 0.05 with 40 pairs needs at least 6 one-way flips (15.0% of 40) with none
the other way, 8 with one the other way, or 10 with two. The largest real signal, 4 one-way flips for the
gender-coded name, is below that line. So I may **not** say that the name or the university changed the
decisions. What I *can* say: the university moved the *score* by about 2.5 points in favour of
flagship institutions (clear of zero), but this almost never crossed the decision line (1 flip, the same
as its floor).

**How much could be noise, and what the floors test.** Decoding is greedy, so there is no run-to-run
noise; the floors measure how much the output moves when an irrelevant detail changes:
`null_name` changes the name inside the same language group and gender coding (0 flips, but 7 of 40 pairs
had different scores), and `null_university` changes the university inside the same tier (1 flip, 9 of
40 pairs with different scores). The floors keep the layout, the prompt wording, the merit fields and
the model fixed, so they do **not** test sensitivity to wording, to line order, to other models, or to
changes in other fields. A stronger estimate would need more pairs (to see effects below 6 of 40), several
prompt wordings and line orders (my order probe below already shows 10 of 40 flips from the order of the
lines alone), and pairs built around the decision line, where most flips
can happen (most profiles here score 70-79, far from the line).

**No measured effect is not no effect.** With 40 pairs per attribute this run could not see a real
effect that moves fewer than about 6 pairs one way. A 10% one-way gender effect, like the 4 of 40 I saw,
could be real and still not be detected.

**Different prompt, model or real applicants.** My run supports statements only about this model, this
prompt (`v1`) and these synthetic profiles. A different wording, a bigger model or real applicants whose
names and universities carry more signal could behave differently in either direction.

**After the harness (Part 3).** With name and university hidden, every pair table is 0 of 40 flips and 0 of 40 score differences in all five groups, but this is zero **by construction**: both members of a pair see the same text, so it says nothing about whether the model became fair. At the level of single profiles, 51 of 400 decisions changed (reject -> escalate 27, shortlist -> escalate 19, reject -> shortlist 5) and 166 of 400 scores changed (mean change 14.01). That is not proof that the model used the removed fields, because removing two lines also changes the form of the prompt. The optional order probe supports this reading: 40 pairs of identical applicants shown with the lines in another order gave 10 flips = 25.0% [14.2%, 40.2%], all in one direction (McNemar p = 0.002), and a score shift of -6.6 [-9.9, -3.5]. So the largest measured sensitivity of this agent is to the **layout** of the prompt, not to a protected attribute; my floors do not cover it. The harness cost no extra model calls (400 and 400), sent 46 of 400 (11.5%) to a person against 64 of 400 (16.0%) planned, and refused 0 of the measured applicants. In the counterfactual extension (8 name-language pairs, 32 model calls), the answer never changed when the name was swapped.

## 2. Who is affected

**Wrongly rejected.** In my run the side that lost every flip of the gender-coded name was the applicant
with the **male-coded** name (4 of 40 pairs: female shortlisted, male rejected). In the name-language
pairs the side that lost was the **non-Kazakh** name (2 of 40), and in the university pairs the
**regional** university applicant lost 1 pair and scored about 2.5 points lower on average. None of
these is clear of its floor, but if they are real, these are the people who are wrongly rejected: a
male applicant, an applicant with a Russian or English name, or a regional student with the same grades,
exam and income as someone who was shortlisted.

**Wrongly shortlisted, and who is displaced.** On the other side of each flip is a shortlisted twin. With
a limited number of places, a person shortlisted because of the name or university takes a place from
someone else, usually a borderline applicant. In my run 356 of 400 were shortlisted, so the shortlist
itself is long and the real selection happens later; a wrong shortlist costs a later reviewer's time and
may push out a better-matched applicant.

**The cost is not the same.** A wrong reject costs the applicant a scholarship (money, maybe the
possibility to study at all for a low-income student) and gives them a record of rejection that is
never re-checked, because nobody reviews a reject outside the band. A wrong shortlist mostly costs
review time and displaces someone at the next stage. That is why my escalation rule sends rejects of the
lowest income bracket to a person.

**Parties the brief does not mention.**
- People who never apply because they expect a machine to reject them (for example students from
  regional colleges, if the score gap becomes known).
- The staff who review escalated cases: 46 of 400 (11.5%) of 400 applicants went to them in the harness run.
- The fund, which carries the legal duty (consent, explanation, the right to object within 3 working
  days) and the reputation risk.
- Applicants from institutions the model has never seen, and applicants whose names do not fit the
  name rule (refused instead of screened).

**If an effect exists that my 40 pairs could not see.** The same groups would bear it: male-coded names,
non-Kazakh names, regional universities, each up to about 6 flips in 40 (15%) without my test noticing.

**Compounding over time.** If the agent is used every year, a small shift against regional universities
becomes a pattern: fewer regional students funded, fewer regional success stories, and a later model
trained on past decisions would learn the gap as if it were merit. If its output is an input to a later
review, a published ranking or another fund, the first small bias is copied without anyone seeing the
original case.

## 3. Professional responsibility

**My responsibilities as the developer of the harness, given what I measured.**
1. **Before it runs:** check every protected attribute against its floor on a larger sample, near the
   decision line, and with more than one prompt wording; check that the refusal rules do not refuse real
   names (0 of 400 clean applicants were refused in my run); check that the log is written for
   every call, refusals included.
2. **What I must tell the people who deploy it:** that no attribute was shown to move decisions *beyond
   its floor*, but that the test could only see effects of at least 6 flips in 40; that the university
   moved scores by +2.5 [+1.1, +4.1]; that name and university are now hidden from the model, so their
   pair results after the harness are zero **by construction** and prove nothing about the model; that
   the band 40-45 sent 46 of 400 (11.5%) applicants to a person, against 64 of 400 (16.0%) planned; and that
   the model's reasons are not explanations of its internal process. Most important: the same applicant
   shown with the lines in another order got a different decision in 10 of 40 pairs (25.0%), so the
   fund must fix one layout and never compare scores from different layouts or prompt versions.
3. **What I must not claim:** that the system is "fair" or "unbiased", that blinding removed bias from the
   fields that stay visible, or that the results hold for another prompt, model or real applicants.

**What discharging them requires, and what I did.** I did the matched-pair test with floors, wrote and
exercised the refusal path (Part 4: 5 of 6 requests refused, 0 model calls for them), wrote
a log of every decision, and stated the limits above. Left to do: a larger and repeated measurement, a
documented stop condition (for example: stop automatic decisions if any attribute is clearly above its
floor in a monthly check), a real human queue with the 3-working-day answer, and protected log storage.

**What belongs to the institution.** Setting and limiting the purpose (only scholarship screening),
deciding who reviews and how many reviewers there are, getting consent, explaining the processing,
handling objections and appeals within the time the law sets, and deciding how long the log is kept.

**Where the two conflict.** If the fund wants to remove the escalation band because 16% of applicants
read by a person is slow and costs money, my measurement is the answer: those cases are exactly where
rejects and shortlists overlap (the 40-49 bin held 33 rejects and 31 shortlists), and the law requires a
human route for objections anyway. If the fund insists, my duty is to document the risk in writing, to
refuse to describe the system as fair or human-checked, and, if the system would then break the
consent or objection rules, not to deploy it.
