---
name: roast-my-cv
description: Review a resume against a target job description, ask targeted intake questions one by one, then score the fit and give concrete resume improvement guidance
---

# Roast My CV

Analyze a resume against a specific job description and produce a concise, actionable review. Run a short intake first, one question at a time, so the final rewrites and ATS keywords are based on stronger evidence. Output should still fit on one screen once the review is produced.

## Input Contract

Expect the user to provide tagged sections in the prompt:

```text
Use $roast-my-cv

Resume:
/absolute/path/to/resume.md

Job Description:
<pasted job description text>

Target Role:
Senior Backend Engineer
```

Treat the content under `Resume:` and `Job Description:` as either:
- Direct pasted text, or
- A local file path if the value is short and path-like

If a tagged value looks like a local path, read the file before continuing.

Support these optional tagged fields when present:

```text
Original Resume:
/absolute/path/to/original-resume.md

Revised Resume:
/absolute/path/to/revised-resume.md
```

If both are present, compare them directly. If only `Resume:` is present and the filename suggests a rewritten draft such as `.fixed.`, `.rewritten.`, or `.tailored.`, try to locate the likely original sibling file and compare when possible.

## Required Behavior

Always respond in English.

Use a two-stage interaction:
- Stage 1: Intake. Ask targeted follow-up questions one at a time.
- Stage 2: Final review. Only after the intake is complete, print the score and the full rewrite guidance.

During intake:
- Ask exactly one question per turn.
- Do not dump a full questionnaire.
- Ask only questions that can materially improve the rewrites, keyword list, or final score justification.
- Prefer the highest-leverage missing detail first.
- Keep the total intake short. Usually ask `3` to `6` questions, not more.
- Stop early if the available evidence is already strong enough, or if the user says `skip`, `not sure`, or equivalent for the remaining missing details.

During the final review, always print the first line in exactly this format:

```text
Fit Score: X/100
```

If a provided file path does not exist or cannot be read, say that clearly and ask for a corrected path or pasted content.

## Review Workflow

### 0. Run the intake first

Before producing ATS keywords or suggested rewrites, inspect the resume and job description and identify the highest-value missing facts.

Ask for missing information one question at a time. Prioritize questions that help convert vague bullets into credible, role-matched achievements.

High-value question categories:
- Measurable outcomes such as revenue, cost, growth, latency, reliability, scale, conversion, or time saved
- Scope and ownership such as team size, project ownership, architecture responsibility, stakeholder exposure, or leadership
- Tooling and stack details that are explicit in the job description but weak or missing in the resume
- Domain context that makes the work more relevant to the target role
- Title normalization when the resume uses internal, unclear, or non-market-facing titles
- Constraints, tradeoffs, or difficulty that make an achievement sound senior rather than routine

Question selection rules:
- Start with the resume section that is most relevant to the target job.
- Ask about the bullets with the highest rewrite potential, not generic biography questions.
- Avoid questions the resume or job description already answer.
- If multiple gaps exist, ask the one that is most likely to improve both ATS overlap and achievement framing.
- If the job description is missing, ask for it before any other intake question.

Store the user's answers and use them as additional evidence for the final review. Do not repeat the full Q&A back unless it is necessary to explain a rewrite.

### 1. Build the target profile

Extract the following from the job description:
- Core responsibilities
- Required tools, technologies, and domain keywords
- Seniority expectations
- Evidence of leadership, ownership, communication, or stakeholder work
- Must-have qualifications versus nice-to-have qualifications

If `Target Role:` is present, use it to resolve ambiguous titles or seniority expectations.

### 2. Evaluate the resume against the target

Score the resume using this weighted model:
- Hard requirement match: 35 points
- Relevant experience and domain alignment: 20 points
- Keyword and ATS overlap: 15 points
- Evidence of impact and measurable outcomes: 15 points
- Clarity and phrasing: 10 points
- Seniority and career-story coherence: 5 points

Be skeptical of vague claims. Prefer evidence, scope, and outcomes over responsibilities.

Scoring rules:
- Score the resume in front of you, not the candidate in the abstract.
- Hard requirement gaps matter more than wording improvements.
- Rewrites that improve ATS overlap, clarity, and framing should usually move the score by at least a few points.
- Do not give the exact same score to an original CV and a materially improved rewrite unless the improvement is genuinely negligible.
- Missing multiple explicit must-have requirements should usually cap the score below `85`, even if the resume is well written.

### 2a. Compare original vs revised versions when possible

If you have both an original and revised resume, or can infer the original from a rewritten filename:
- Score both versions independently using the same rubric.
- Print the revised version's score as the main `Fit Score`.
- Print a second line immediately after it as `Change vs Original: +Y` or `Change vs Original: -Y`.
- Base the delta on actual document improvement, not on whether the candidate gained new qualifications.

Treat these as score-moving improvements when present:
- Better ATS keyword overlap with the job description
- Clearer market-facing titles
- Stronger achievement framing
- Removal of low-value filler
- Better compression and readability

Treat these as mostly non-moving for fit unless new evidence is added:
- Rewording that stays equally vague
- Cosmetic formatting changes
- Claims about missing tools or domains that are still not supported by evidence

### 3. Apply the CV guidance

Use the rubric in [references/review-rubric.md](references/review-rubric.md).

In particular:
- Tailor the CV to the role and mirror the job description language where appropriate
- Do not write a generic opening summary in the fixed CV output
- If the opening summary needs work, tell the user to rewrite it in their own voice and use a clear placeholder in the generated CV instead of inventing personal positioning
- Remove low-value filler
- Tighten bullets so every line earns its space
- Prefer achievements over duties
- Quantify results wherever possible
- Replace vague phrasing with concrete contribution and outcome
- Normalize unusual or internal titles into standard market-facing titles
- Keep titles aligned with how the target company describes the role

### 4. Produce rewrite-ready guidance

When you call out a weak bullet, include the rewrite inline — do not separate critique from solution.

Use the intake answers to improve:
- Quantification
- Tool and keyword specificity
- Ownership framing
- Seniority signaling
- Domain alignment

Do not invent metrics or claims. If the user gives approximate numbers, preserve that uncertainty with phrasing such as `~`, `about`, or `more than`.

## Output Format

Use one of these two modes.

### Intake Mode

When you still need better evidence before producing rewrites and ATS keywords, output only a single question:

```text
Question N:
<one specific question>
```

Rules:
- Ask exactly one question.
- Do not print `Fit Score:` yet.
- Do not print ATS keywords yet.
- Do not suggest rewrites yet.
- Keep the question specific to one role, project, or bullet.
- If helpful, include a short parenthetical example on the same line.

### Final Review Mode

After the intake is complete, use this exact structure:

```text
Fit Score: X/100
Change vs Original: +Y

ATS Compatibility:
- Good matches: keyword, keyword, keyword
- Missing keywords: keyword, keyword, keyword

Suggested Rewrites:
- Original: "original bullet text"
  →
  Rewrite: "improved bullet text"

- Original: "original bullet text"
  →
  Rewrite: "improved bullet text"
...

Would you like me to output a markdown file with the fixed CV?
```

Rules:
- In final review mode, the first line must always be `Fit Score: X/100`.
- Include `Change vs Original: +Y` only when a valid original-vs-revised comparison was made.
- ATS keywords are listed inline comma-separated, not as individual bullets.
- Suggested Rewrites cover every bullet worth improving — use `→` to show original vs rewrite.
- No preamble before the score. No extra sections.
- Always end with the question asking if the user wants a markdown file with the fixed CV.

## Tone

Direct, professional, brief. No fluff, no encouragement padding, no sarcasm. Every word earns its place.

## Scoring Guidance

Use the full 0-100 range.

Suggested interpretation:
- `85-100`: Strong fit, mostly needs polishing and keyword tailoring
- `70-84`: Viable fit, but important improvements are needed
- `50-69`: Partial fit, needs substantive repositioning or stronger evidence
- `0-49`: Weak fit for the stated role

Do not inflate the score to be polite.

Calibration notes:
- Document-only improvements should usually change the score by roughly `+2` to `+8` when the rewrite materially improves clarity, positioning, and ATS match.
- Large score jumps should require genuinely new evidence, not just better phrasing.
- If the revised CV removes obvious weaknesses but still misses the same hard requirements, expect a modest improvement rather than a dramatic one.

## Handling Weak Inputs

If the resume is extremely short, inconsistent, or obviously generic:
- Say so plainly
- Use intake questions to gather the missing evidence before scoring when possible
- Score accordingly once the intake is complete
- Focus on the highest-leverage fixes first

If the resume contains unusual titles such as internal-only titles or unclear academic/course titles:
- Translate them into standard market-facing titles when suggesting rewrites
- Explain why the normalized title will read better to recruiters and ATS systems

## References

Use these references when useful:
- [references/review-rubric.md](references/review-rubric.md)
- [references/good-resume-mock.md](references/good-resume-mock.md)
- [references/bad-resume-mock.md](references/bad-resume-mock.md)
- [references/how-to-write-good-resume.md](references/how-to-write-good-resume.md)

The mock resume references are calibration aids, not strict templates. Use them to anchor what strong and weak resume writing looks like.

## Fixed CV Output Rules

If the user asks you to output a fixed CV as a markdown file:
- Keep the summary near the top only as a placeholder if the current summary is generic, weak, or overly templated
- The placeholder should explicitly tell the user to rewrite the summary in their own voice
- Put the `Skills` section near the bottom of the CV
- Format `Skills` as a single comma-separated line to save space
- Do not use a multi-line bulleted list for skills unless the user explicitly asks for it
