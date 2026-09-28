# Preregistration: Predicting Certiorari at the Supreme Court of the United States (Conference of September 28, 2026)

### Overview

We forecast the likelihood that the Supreme Court will grant each paid petition for a writ of certiorari distributed for its September 28, 2026 conference, and preregister those forecasts before the Court acts on any of them.

We compare three kinds of forecaster:

* a `ModernBERT` classifier, fine-tuned on decided petitions, that reads the petition itself (the lead attorney, the question presented, and the body of the petition) together with facts fixed at filing (whether the United States is a party, details of the court and judges below, and the justices sitting), the amicus briefs filed so far (including whether state or legislator amici filed, and the number of amicus briefs), whether the Court requested a response, and the respondent's response (a brief in opposition, a brief not in opposition, or a waiver);
* a frontier language model (`claude-fable-5`) given the same inputs and nothing more; and
* the same language model with open web search, given only the docket number and caption.

We also compare these forecasts with two external sources: the two lists on SCOTUSblog's "Active Petitions We're Watching" page as of 9/24/26, and the overall ("eventual") grant probabilities published at supremecourt.report, as of 9/21/26.

Our fine-tuned ModernBERT classifier was trained on data drawn from about 45,100 decided paid petitions from the 1996 Term through the 2025 Term (decisions through June 30, 2026), excluding GVRs: about 31,600 for training, and the rest for tuning and held-out testing. **claude-fable-5** was not specifically trained for this task. (Its knowledge history runs through January 2026; all but two of the petitions, 25-882 and 25-901, were docketed after that.)

Our classifier's probability for each petition is the average of four training runs, Platt-scaled on decided petitions held out from training. The scaling uses the outcomes of decided cases, which the language model did not see; it does not change the order of the petitions. `claude-fable-5` was queried through the API with the prompts in `prompts/PROMPTS.md`; `prompts/api_prompts.jsonl.gz` has the full prompt for every petition. The open-web arm used web search with unrestricted access, except that it was forbidden from consulting the other benchmark websites or any source reporting someone else's prediction for the petition. For this arm, the files in `transcripts/` record each search and fetch, but not what they returned (apart from each search's result titles and addresses).

### Outcomes

For our models and that of supremecourt.report, we report the outcome as a predicted probability of grant. For SCOTUSblog, the outcome is binary.

The predictions are stored as follows:

| file                                   | arm                                                          | cases |
|----------------------------------------|--------------------------------------------------------------|------:|
| `predictions/hn_fable5_model.csv`      | `claude-fable-5` reading the petition, plus facts fixed at filing (US party, court and judges below, the justices sitting), amicus briefs filed, whether a response was requested, and the respondent's response | 231   |
| `predictions/hn_bert_model.csv`        | our fine-tuned ModernBERT classifier, trained on decided petitions from 1996 on, reading the same as `hn_fable5_model` | 231   |
| `predictions/fable5_open.csv`          | `claude-fable-5` with web search, given only the docket number and caption | 240   |
| `predictions/scotusblog.csv`           | SCOTUSblog's two lists, as of September 24, 2026: one column for the list of petitions for the next conference (58 of these petitions) and one for its featured list (29); 1 if listed, 0 if not | 240   |
| `predictions/supreme_court_report.csv` | overall grant probabilities from supremecourt.report          | 239   |

Every file lists all 240 petitions; a source with no forecast for a petition leaves it blank.

### Metrics and limitations

Our primary measures are rankings and grant probabilities. (SCOTUSblog's lists are not probabilities, so we will treat each as a binary prediction.)

The test set is every paid petition filed by counsel and distributed for the Court's September 28, 2026 conference: 240 petitions. The open-web arm scores all 240, while the arms that read the petition score 231. The remaining nine were filed in paper form only, so the docket has no petition text to read. Three of them are petitions by the federal government, which the open-web arm and SCOTUSblog mark as likely grants, so the 231 are not a random subset of the 240.

One known limitation of the two arms that read the petition: in 36 cases the Court has asked for a response that is not yet due, and the docket shows no brief, so the respondent's response reads as not filed. The classifier learned from decided cases, where a requested response has almost always been filed by the time of the decision, so it cannot tell "not yet due" from "never filed". We did not correct these forecasts; they are the models' outputs, produced the same way as for every other petition. The cases: 25-1347, 25-1348, 25-1355, 25-1359, 25-1360, 25-1427, 26-2, 26-4, 26-15, 26-18, 26-19, 26-22, 26-23, 26-29, 26-37, 26-46, 26-47, 26-48, 26-60, 26-61, 26-88, 26-91, 26-94, 26-97, 26-98, 26-100, 26-107, 26-110, 26-127, 26-170, 26-194, 26-219, 26-222, 26-231, 26-238, 26-244.

The forecasts, prompts, transcripts, and this registration were made and timestamped before the Court's conference.