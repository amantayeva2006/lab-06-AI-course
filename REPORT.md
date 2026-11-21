# REPORT — what the measurement showed, who it affects, and whose responsibility it is

## 1. Measurement read-out

I screened 400 invented profiles (5 groups of 40 matched pairs: name language, gender-coded name, university, and the
two floors `null_name` and `null_university`) with the bare agent. 0 of 400 replies were unparseable, and the agent
shortlisted 356 of 400 profiles (89%). Because almost everyone is shortlisted, only profiles near the cut can flip, which
limits what flip counts can show.

| attribute | flips (of 40 pairs) | 95% CI | pairs with different scores | mean \|score diff\| |
|---|---|---|---|---|
| name_language | 2 (5.0%) | [1.4, 16.5] | 11 | 3.12 |
| gender_coded_name | 4 (10.0%) | [4.0, 23.1] | 6 | 1.12 |
| university | 1 (2.5%) | [0.4, 12.9] | 10 | 2.50 |
| null_name (floor for the two name attributes) | 0 (0.0%) | [0.0, 8.8] | 7 | 2.00 |
| null_university (floor for university) | 1 (2.5%) | [0.4, 12.9] | 9 | 2.12 |

**No attribute stands clearly above its own floor on decisions.** name_language (2/40) and gender_coded_name (4/40)
are read against null_name (0/40, CI up to 8.8%); their intervals overlap it. university (1/40) is identical to its floor
null_university (1/40). So I cannot say any of the three moves the decision.

Directions, with the caution that none is significant on decisions:

- **Name language.** Shortlist rates were english 96.2%, kazakh 96.4%, russian 92.3%. Kazakh-language name vs the
  others: 2 pairs favoured the Kazakh name, 0 the other (McNemar p = 0.50; score gap +0.7 [-1.8, +3.8]). Russian: 0 vs 1
  (p = 1.0); English: 0 vs 1 (p = 1.0; score gap -1.0 [-4.2, +1.9]). No group is disadvantaged.
- **Gender-coded name.** This has the highest flip rate (4/40), and all 4 flips went against the male-coded name
  (0 favoured it, 4 favoured the female-coded; McNemar p = 0.125). Shortlisted: female 90.0%, male 80.0%. Scores barely
  moved (gap -0.4 [-1.2, +0.6], sign p = 0.219; mean |diff| 1.12, the smallest of the five), which suggests pairs sitting
  at the cut. It is suggestive and worth watching, not an established effect.
- **University.** Decisions did not move beyond the floor, but scores did: flagship minus regional +2.5 [+1.1, +4.1],
  sign p = 0.002 (10 of 40 pairs had different scores, and all 10 went the same way). Shortlisted: flagship 92.5%, regional
  90.0%. Regional applicants get about 2.5 points lower scores, rarely enough to change the outcome. The tables give the
  floor's mean |diff| (2.12) but not its signed gap, so I cannot say how the floor's score differences point.

**What the floors do and do not test.** They change which name (same language and gender coding) or which
university (same tier) is shown, and keep everything else fixed. They show that these surface changes already move scores
(null_name: 7/40 pairs with different scores, mean |diff| 2.00; null_university: 9/40, 2.12). They do not test other
prompt wordings, other fields, combinations of attributes, other models or real applicants. [Detectability, from
`print_detectability`: with n = 40 pairs the run could notice a flip-rate difference of about ___ and could not detect
smaller ones.] So for name_language and gender_coded_name the result is "no measured effect beyond the floor",
which is not "no effect"; the gender result in particular is too uncertain to rule out. A stronger estimate would need
many more pairs and several prompt wordings.

**Predicted but not there:** I predicted Kazakh-coded names would fare worse (they did not), university flips of 20-30%
(observed 2.5%), and flip floors of 5-10% (observed 0% and 2.5%).

## 2. Who is affected

**Wrongly rejected** is the applicant the model scores below the cut for a reason unrelated to merit. In my measurement
the clearest candidate is the applicant from a regional university, who gets about 2.5 points lower scores than an
otherwise identical flagship applicant. Only 44 of 400 profiles were rejected, so a 2.5-point shift mostly matters at
the cut (scores 40-49 held 64 of 400 profiles). The second candidate is the applicant with a male-coded name (shortlisted
80.0% vs 90.0%), but that difference does not clear its floor. **Wrongly shortlisted** is the applicant who benefits
from the same shift (a flagship applicant) and takes a place from someone else, because a scholarship has a limited
number of places. A wrongly rejected applicant loses money, a place and time, and, if rejections are recorded, a record
that follows them; a wrongly shortlisted one costs another applicant the place.

**Parties the brief does not mention:** applicants who never apply because they expect rejection (for example from
regional universities), the staff who review escalated cases (under my harness they read [N] of 400 applicants, [X]%
against 16% planned), the fund, and applicants from universities or name groups the model has seen little of; my sample
uses invented names and universities and cannot say how these are treated.

**If there is divergence my sample could not see:** a 2.5-point shift or a 10% gap in flips at n = 40 can
hide a real effect, and the people who would bear it are those described above. **Compounding:** if the agent is used
year after year or its output feeds a later review, a small score gap becomes a systematic advantage for flagship
universities, and a shortlist built from biased scores trains the next stage on the same bias.

## 3. Professional responsibility

**My responsibilities as developer of the harness.** Before it runs: test the refusals and the log on cases that should
and should not be refused (Part 4: the harness refused [N] of 6 requests and my rules would refuse [N] of 400 clean
applicants); calibrate the escalation band after every change to the prompt, because it does not transfer (I planned 16%
and the run escalated [X]%). To the deployers: say that the harness hides name and university, so it cannot show the model
became fair, only that it can no longer see those fields; that decisions changed relative to the unblinded run for [N]
profiles, which does not prove the model relied on them; and that the measurement had 40 pairs per attribute, one prompt
and one model. I must not claim that the system is non-discriminatory, or that no divergence exists. What I did: measured
against floors, built five controls (validate_input, blind, refuse, log, escalate), logged every call and tied each control to a
principle and a provision (`control_register.json`). What is left: a larger and varied sample, testing other prompt
wordings, and a test with real names.

**What belongs to the institution that deploys it:** setting the purpose, choosing and training the reviewers, handling
appeals, the user agreement and the information the applicant must be given, and deciding what to do with the
applicants the harness refuses.

**Where the two conflict.** If the institution wants a control removed because it slows the process or costs money, my
measurements say what each prevents: escalation costs [X]% of reviewer workload but is the only place a person reads the
borderline cases; blinding costs [N] changed decisions but removes name and university from the model's input;
the refusals stop minors, no-consent applicants and injected instructions that the bare pipeline processed without
objection. I would document the removal, state the risk in writing, and refuse to describe the reduced system as one that
treats matched applicants alike.
