# How the work is done

An account of how analysis work is run on one project, [crypto-signal-agent](https://github.com/vimakushin/crypto-signal-agent): which documents exist, what each one is for, and one case where a measurement left the original claim unproven, and the claim was dropped.

Who does what: the idea, the requirements, the decisions (the documents record them as the owner's) and the choice to stop making unproven claims are the owner's. An AI coding assistant writes the code to those requirements, under the rules in `CLAUDE.md`. Separate review agents with different roles check the result.

The documents are public but written in Russian (links at the end). This page is assembled only from them and from the code. Quotations are translated.

## The chain of documents

1. **Requirements specification** (`TZ_crypto_early_signal_detector.md`, written in September 2026). Section 1 is the original idea: find coins *at an early stage of a move*, before the rise shows on the price chart. The same document lists the data sources, seven candidate signals, functional and non-functional requirements, the split between the first version and later iterations, and a section of risks and limitations. It also states what is out of scope: buy/sell signals, automatic trading, financial advice.
2. **Acceptance criteria** (`MVP.md`). A definition of done: six checklist items, each either met or not, "without 'almost' and 'mostly'". It opens with the condition for calling the first version finished: every morning the user gets a message with a list of candidates and an explanation of why each is on the list. Anything not in the checklist is not part of the first version.
3. **Development plan** (`SURVEILLANCE.md`). Where the project goes next. First written on 13 October 2026, rewritten by the owner on 19 October after the measurement described below.
4. **Decision log and deferred items** (`BACKLOG.md`). What was postponed on purpose, "not forgotten", with the date and the reason. Measurement results and the decisions that followed are recorded there too.

A fifth file, `CLAUDE.md`, holds the working rules for the AI coding assistant that writes the code. It includes a rule for deviations: anything that does not move the nearest unmet checklist item must be named as a deviation, written into the log if it can wait, and done now only if the next item is blocked without it or data would be lost.

There is no separate catalogue of edge cases or failure modes. Analyses of individual failures exist as entries in the log (for example, the gap of 2–3 October 2026, when the daily message went out on stale data and then not at all, and the unexplained anomalies in one source's data on 22–29 September 2026, whose cause the log says it cannot establish).

## A worked example: a measurement that left the original claim unproven

**The idea.** The specification set the goal as finding coins before their growth is visible. Three signals were built for the first version, one of them comparing a protocol's revenue with its token's price: revenue grows, the price has not reacted yet.

**The freeze.** On 15 September 2026 a backtest showed no advantage over the baseline. On 12 October that result was marked, by the owner's decision, as computed with a faulty instrument: the replay script looked one day ahead, and it used first-received revenue values that the source later revised. The note says the result "confirms nothing and refutes nothing". The original entry was kept unedited, with the note above it. Since the formula had been calibrated on the same history, the signals, thresholds, weights and coin lists were frozen from 15 September to 15 October, so that the October measurement would be done on data the formula had not seen.

**A finding about the data.** On 12 October it turned out that the revenue source recalculates its history after the fact. The decision was to re-fetch the full history and keep both versions of the numbers: what the system had received at the time, and what the source reports now. Overwriting the first values "would destroy the evidence of the finding itself".

**The measurement, 15 October 2026.** The script is `scripts/measure_hit_rate.py`; a repeat run at the time reproduced the report byte for byte. Since 19 October it reproduces only on the code of 15 October (commit `455c37f`), because a later change to the outlier filter alters the counts; the log calls that an expected consequence of a decision, not an error. It took every episode of the revenue signal that began on or after 15 September and judged the outcome mechanically, by the rule in `LABELING.md`: the coin's price change over 7, 14 and 30 days minus the median of the other coins on the list; more than +5 percentage points counts as "worked", less than −5 as "did not work", anything in between as "too early to judge". The causes of price moves were not considered.

At the 7-day horizon, compared with a baseline (the same measure applied to the same coins outside their episodes):

| Data version | Episodes with a 7-day outcome | "Worked" | Baseline: same coins outside their episodes |
|---|---|---|---|
| Stored (what the signal saw) | 7 | 1 (14%) | 35% |
| Restated (what the source reports now) | 13 | 4 (31%) | 31% |

For the whole list on all days, the baseline was 29%. The log's conclusion: the data do not show that the signal beats the market, and do not show that it fails either, because the sample is too small. The 30-day horizon could not be measured at all yet. The log's own reading note: this is "not a conclusion that the signal doesn't work, but the absence of a conclusion".

**Caveats recorded with the result.** The formula had been calibrated on the same history. Many episodes touch known source problems (4 of all 8 stored-data episodes and 13 of all 17 restated-data ones have a 7-day window affected), and the gap of 2–3 October is part of the period; the log requires these to be named, not folded into the headline figure. The better result on the restated data (31% against 14%) comes entirely from episodes the live system never raised.

**Two further numbers.**

- *The same window and the same formula gave 8 episodes on the stored data and 17 on the restated data.* The log records the difference as a result in its own right.
- *How many episodes would settle the question.* For a two-sided test at the 5% level with 80% power, with about 30% "worked" for a random coin: telling 30% from 45% takes **163 episodes**; telling 30% from 40% takes 356. The system produces 8 to 17 episodes a month, so 163 episodes is **10–20 months** (about 20 at the lower rate). The rate is known only to within a factor of two.

**A later correction to the same measurement (25 October 2026).** A check of the stored data found that the first value of a day is often wrong in both directions: for 15 of the 25 protocols, 34% of rows were more than 10% too low and 31% more than 10% too high. The log now says that this noise, more than source revisions, accounts for the gap between 8 and 17, that the measurement was made on data "that lie in both directions", that 8 episodes and the pace of 8 a month are therefore understated, and that the 10–20 month estimate shifts toward shorter, though the order of magnitude holds. The 15 October figures were not re-counted for that month. The collectors have since stopped storing an unfinished day.

**What was decided.**

- The project stopped making the claim. The plan says: until this number exists, wording such as "the service finds coins before the growth" would assert something unproven. The project now has two goals. The main one is observation: notice unusual moves in revenue, price and volume and flag them for a person to investigate, with no price prediction. The early-stage search remains as the owner's personal hypothesis, watched but not claimed. `CLAUDE.md` tells the assistant not to describe it as a property of the service in the README, the daily message or the interface.
- The specification was not rewritten. On 22 October 2026 a dated note was put above it: the text below "is deliberately not rewritten", the goal was refined after the 15 October measurement, and the current direction is in `SURVEILLANCE.md`.
- The cost was written down. The decision to keep two coin lists, one for each goal, is recorded with its price: about twice the data-collection work, and the free limits of the data source will be reached sooner.
- The checklist stayed as it was. `MVP.md` says the measurement does not change any of its items and points to the log.

## The same habit elsewhere

Checklist item 6 ("the cycle ran without intervention: three days in a row the message arrives by itself") was ticked on 15 September 2026 after three messages in a row. On 5 October, after an audit, the tick was removed: the streak had broken on 2 and 3 October (a late start with a locked database, then no internet), and the tick "stood on a fact that had ceased to be true". The item is still open, with the condition for restoring it written next to it.

## The documents

All of them are in Russian; no English versions exist.

- [Requirements specification](https://github.com/vimakushin/crypto-signal-agent/blob/master/TZ_crypto_early_signal_detector.md): the dated note at the top, section 1 for the original goal.
- [Acceptance criteria](https://github.com/vimakushin/crypto-signal-agent/blob/master/MVP.md)
- [Development plan](https://github.com/vimakushin/crypto-signal-agent/blob/master/SURVEILLANCE.md): section 1 for the two goals.
- [Decision log](https://github.com/vimakushin/crypto-signal-agent/blob/master/BACKLOG.md): the entry "Итог измерения 15 октября 2026" is the measurement above. The file is long.
- [Working rules](https://github.com/vimakushin/crypto-signal-agent/blob/master/CLAUDE.md)
- [Labelling rule](https://github.com/vimakushin/crypto-signal-agent/blob/master/LABELING.md): how episode outcomes are judged.
- [Definitions of the review agents](https://github.com/vimakushin/crypto-signal-agent/tree/master/.claude/agents): the roles mentioned above (also in Russian).

## Scale

This is a personal project of one person, not a programme of an organisation. The coin list in the example has 25 protocols and three signals, and the system runs on the owner's own computer. The development plan itself says it does not settle "whether this will become a product for anyone other than the author".
