# The prompts

Both Claude arms use `claude-fable-5` at effort `high`. Each arm's answer is
constrained to the JSON schema shown for it.

## The petition arm (Messages API, no tools)

This arm reads what the classifier in `predictions/hn_bert_model.csv`
reads: the petition, the facts fixed at filing, the justices sitting, and the docket
as of this forecast.

**System prompt:**

```text
You estimate whether the Supreme Court of the United States will grant a petition for a writ of certiorari. You see a paid petition exactly as a text classifier sees it: the lead attorney, the question presented, and the body of the petition, with its middle omitted when it is long. Some requests add structured information about the case. Judge only from what is shown.
```

**User message.** Two text blocks. The first is the petition as the classifier
reads it:

```text
LEAD ATTORNEY:
{lead attorney}

QUESTION PRESENTED:
{question presented}

PETITION:
{front of the petition body}

[... middle of the petition omitted ...]

{end of the petition body}
```

The marker and the end of the body appear only when the petition is longer than
the classifier's 8,192-token input. The second block gives the case's record on the
docket as of this forecast: facts fixed at filing, amicus briefs filed so far, whether
the Court has asked for a response, and the respondent's response. Relists, calls for
the Solicitor General's views and the Solicitor General's amicus briefs are left out,
because a petition awaiting its conference cannot have them yet. For example, for
25-1389:

```text
Additional information about the case, its record on the docket as of this forecast:
us petitioner: no | us respondent: no | court below: 5th Cir. | panel below: DENNIS; RICHMAN; HO | dissent below: yes | dissenting below: DENNIS; RICHMAN | decided en banc: no | per curiam below: yes | dissent from rehearing denial: yes | state amici: no | legislator amici: no | amicus briefs: 15 | response requested: no | brief in opposition filed: yes | response waived: no | respondent's brief filed, not in opposition: no
Justices sitting when the petition was filed: Amy Coney Barrett; Brett M. Kavanaugh; Clarence Thomas; Elena Kagan; John G. Roberts Jr.; Ketanji Brown Jackson; Neil M. Gorsuch; Samuel A. Alito Jr.; Sonia Sotomayor

What is the probability that the Court eventually grants this petition, regardless of when it acts?
```

**Answer schema:**

```json
{"type": "object", "properties": {"p_grant": {"type": "number"}}, "required": ["p_grant"], "additionalProperties": false}
```

The full prompt for every case is in `api_prompts.jsonl.gz`, one JSON object per
line, 231 in all. Each has the docket number, the system prompt, the user message's
text blocks in order, the schema, and `request_sha`, the hash of the exact request
that the prediction file records for that case.

## The open-web arm (Claude Code, web search and web fetch)

The only tools are web search and web fetch; there is no project context and no memory.

**System prompt:**

```text
You are a careful analyst. You have web search and web fetch tools. Today's date is 2026-09-26.
```

**User prompt**, with the docket number and caption as they appear in
`predictions/fable5_open.csv`:

```text
Supreme Court of the United States, No. {docket number}: {case name}.

This petition for a writ of certiorari is pending. Will the Court eventually grant or deny this petition, regardless of when it acts?

You may research the case using any source except scotusblog.com and supremecourt.report. Do not use content originating from either excluded site, including copies, quotations, or summaries published elsewhere. Do not use any source that reports a prediction, by anyone else, of whether the Court will grant this petition; if you come across one, disregard it.

Answer grant or deny, and give the probability that the Court eventually grants the petition.
```

**Answer schema:**

```json
{"type": "object", "properties": {"prediction": {"type": "string", "enum": ["grant", "deny"]}, "p_grant": {"type": "number"}}, "required": ["prediction", "p_grant"], "additionalProperties": false}
```
