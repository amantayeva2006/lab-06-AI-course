# JUSTIFICATION — the design of the harness, and why

Citations of "the AI Law" below are to the Law of the Republic of Kazakhstan No. 230-VIII "On Artificial Intelligence" of
17 November 2025 (in force 18 January 2026), read in the unofficial English text at
https://adilet.zan.kz/eng/docs/Z2500000230 (mirror: https://zakon.uchet.kz/eng/docs/Z2500000230). The Tulpar Fund
is fictional, so this law does not bind the prototype; I use it as the benchmark for a fund that would operate in Kazakhstan.

## 1. What the system may do

The system screens scholarship applications: it reads one invented applicant profile and answers *shortlist* or *reject* with a
score. It may make that call on its own only when the score is outside the escalation band (below 40 or above 45) and the
request passed the refusals; everything else it only informs a person about. Under Art. 17(2) of the AI Law (classes by
autonomy), I designed for the *medium autonomy* class: the system decides, and a person can correct a decision. The
classification itself is for the owner (Art. 17(1)); I have not performed it.

Controls (all in `screen` and `Harness`, set in Part 2d of the notebook):

- **validate_input**: refuses impossible values (e.g. GPA 9.9). Constraint: safety and reliability, AI Law Art. 11(1).
- **blind** (`name`, `university`): the model never sees these fields. Constraint: non-discrimination, Art. 6.
- **refuse**: purpose other than scholarship screening, applicant under 18, no consent, and my extra rule (`my_extra_refusal`:
  a name, university or programme that is not plain text, e.g. an instruction typed into the name field). Constraint:
  data protection, Art. 10(1).
- **log**: one record per call in `decisions.jsonl`. Constraint: continuous control and documentation, Art. 8(1) and Art. 15(2)(3).
- **escalate**: scores 40-45, unparseable replies, and my extra rule (`my_extra_escalation`: the model's reason cites a
  protected attribute, or decision and score contradict each other). Constraint: human autonomy in decisions, Art. 9.

A decision I made differently because of a constraint: without Art. 6 I would not have hidden `name` and `university`,
because Part 1 found no effect on decisions beyond the floors; I hid them because the measurement could not rule one out
(university moved scores by +2.5, sign p = 0.002).

## 2. What it must refuse

It refuses: a purpose outside scholarship screening (`purpose_not_allowed`), an applicant under 18 (`minor`), an applicant
without consent to automated processing (`no_consent`), impossible values (`invalid_input`), and free-text fields that do
not look like plain text (`name_not_a_plain_name`, `university_not_plain_text`, `program_not_plain_text`). Each is enforced
before the model is called, in `refusal_reason`, `validate_profile` and `my_extra_refusal`. In Part 4 the bare pipeline
refused none of the six requests; the harness refused [N of 6] (codes: [list from the table]), with [0] model calls on each.

**Legitimate cases it also blocks.** My text rule checks only the *shape* of a field, so it can block a real applicant with a
name or institution written in an unusual way (more than three words, digits, a colon). In the measured sample it would
refuse [N] of 400 clean applicants (last line of Part 4), but that sample is invented and plain, so it does not
show how the rule treats real names. A wrongly refused applicant gets no score and no reason they can see; the cost is
delay, and I consider it acceptable only if a person reviews refusals. The rule also does not catch an instruction written
as a plausible name.

**What happens to a refused request.** It is recorded in the log with its reason code. Nothing in my code tells the applicant
or lets them ask for a human decision; that is not built, and belongs under "still fails to protect against".

## 3. What record it keeps

**Audit question:** *why did this application get this outcome, and could someone else reconstruct it without re-running
the model?*

Each record in `decisions.jsonl` holds a record id, the application id, a hash of the exact text the model saw, which fields
were shown and which withheld, the model and prompt version, the harness settings, the raw reply, the decision and score, the
controls that fired, the outcome and reason code, whether it went to a person, and a timestamp. [Show one escalation
record and one refusal record from `print_log_summary` and say which fields answer the question.] It holds field *names*,
never field values, so the log does not become a second copy of the applicants' data; the price is that the profile cannot
be rebuilt from the log alone, only checked against the hash.

I did not build retention, access control or tamper protection: `decisions.jsonl` is an ordinary file that can be edited
after the fact. A deployer would need a fixed retention period, read access for reviewers only, write access for the system
only, and an append-only store or hash chain. The log does not let you reconstruct why the model produced a score.

## 4. The legal constraints

Instrument I read in primary text: the AI Law (see top). What I rely on:

- **Art. 6** (justice and equality): AI systems must exclude discrimination on grounds including gender, nationality,
  language and property status. It *requires* the outcome; it does not say how, so hiding the fields is my choice.
- **Art. 7(2)**: a person about whom an AI decision is made has the right to be informed of the procedure for automated
  processing, its consequences, the possibility of objecting, and how to protect their rights. This is *required*; my harness
  does not inform applicants.
- **Art. 8(1)**: owner, possessor and user must ensure continuous control over the system. **Art. 15(2)(1) and (3)**: owners
  and holders must manage risk and keep documentation proportionate to the system's impact. They do not name decision logs,
  so the content of my log is my own choice.
- **Art. 9**: human autonomy and free will in decision-making must be preserved; **Art. 17(2)** defines the autonomy classes.
  Neither fixes an escalation band; 40-45 is my measurement-based choice.
- **Art. 10(1)**: AI use requires compliance with data protection requirements. **Art. 21(4)** says requirements for decisions
  based *solely* on automated processing of personal data are set by the personal data legislation. I have **not** read that
  legislation, so I do not claim what it requires about consent or automated decisions. [Read the Law "On Personal Data and Their
  Protection" of 21 May 2013 No. 94-V, find the provisions on consent and automated decisions, and cite the article here.]
- **Art. 18** (risk management, with risks updated at least once a year) and **Art. 24** (compensation for damage under the Civil Code)
  are relevant to a deployer; I have not built them.

Where I did more than the law requires: the blinding and the injection refusal, which the AI Law does not mention.

**Which control was hardest to tie to a provision.** I found this after searching, not as a prediction made beforehand:
`blind` (Art. 6 prohibits discrimination but does not require hiding fields) and `log` (Art. 8 and 15 require control and
documentation, not decision records). [If the Digital Code No. 255-VIII or the personal data law gives a more specific
provision, cite it after reading it.]

## 5. The ethical principles

The AI Law itself names the principles in Art. 4: legitimacy, justice and equality, transparency and explainability,
responsibility and accountability, the priority of human well-being and freedom of will in decision-making, data protection
and privacy, safety and security. As a statute it has legal force in Kazakhstan, but not for this fictional fund. Which
decision each bears on: *justice and equality* on the shortlist/reject call (blinding); *transparency* on what the applicant
is told (not built); *accountability* on the log; *human freedom of will* on who reads borderline cases; *data protection* on
what the model and the log may see; *safety* on validation. [Add an ethics source you opened, e.g. the OECD AI Principles or the UNESCO
Recommendation on the Ethics of AI, with its year and section, and say what status it has.]

**Conflicts.** Treating applicants identically (hiding the university) conflicts with treating them fairly given what I
measured: a university can carry a legitimate signal about preparation, and it also stands in for region and means. I put
non-discrimination first because the measured effect was on scores and was one-directional, and hiding costs a weak
signal. **Where ethics asks for more than the law:** the law forbids discrimination, but the fund is also obliged to check
whether the measurement is strong enough to rule it out; with 40 pairs it was not, and that has to be said.

## 6. The trade-offs

| control | what it costs | what it prevents |
|---|---|---|
| validate_input | [N] refusals | meaningless scores on impossible values |
| blind | [N] decisions and [N] scores changed vs the unblinded run (not proof the model relied on the field) | the model reacting to name or university |
| refuse | [N] refusals, possible wrong refusals ([N] of 400 clean) | processing minors, no-consent applicants and injected instructions |
| log | storage and a record to protect | decisions nobody can reconstruct |
| escalate | [N] of 400 ([X]%) sent to a person vs 16% planned (64 of 400); reviewer time | borderline calls nobody re-checks |

The band 40-45 was chosen from the baseline score table: the model's decisions were mixed only in the 40-49 bin, and 45 was
the only score with both decisions. It cannot be assumed to transfer: after blinding the scores moved, and the share actually
escalated was [X]%, against 16% planned. Cases outside it (scores below 40 or above 45) are not re-checked by anyone. I would
choose a wider band if reviewer time allowed, and recalibrate it after any change to the prompt.

**What the design still fails to protect against:** applicants are not told about the automated procedure or how to object
(Art. 7(2)); no retention or tamper protection for the log; no appeal route; the injection rule checks shape, not meaning;
the measurement used 40 pairs, one prompt and one model, so an effect smaller than I could detect (see `REPORT.md`) may be
there; and nothing checks the proxies for gender or language that survive blinding.

## AI-use declaration

I used Claude (Anthropic) during this assignment to read the notebook and explain it, to draft the code for the refusal rule,
the escalation rule and the control register, to find and read the text of the AI Law and check article numbers, and to draft
predictions, observations and the first versions of `REPORT.md` and this file. I ran the notebook myself, copied the numbers
from its output, and checked every legal citation against the primary text. [Edit this paragraph so it says exactly what
you did and what you wrote yourself.]
