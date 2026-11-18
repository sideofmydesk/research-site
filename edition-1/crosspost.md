*Originally published at [Side of My Desk Research](https://research.sideofmydesk.com/edition-1/).*

Teams add a plugin to their AI coding tool, see it work, and keep it for years. A plugin is mostly words: steps that tell the AI which helpers to start, which model each should use, and what to look for. The steps are written for the models of the day. Then a newer model comes out, and few people check whether the steps still help.

## What I asked

Does the official code-review plugin for Claude Code, made by Anthropic, still help? And does it help less on a model that came out after its steps were written? The plugin's current steps arrived on 26 October 2026. Claude Opus 5.5 came out before that. Claude Fable 5.2 came out after, on 9 November 2026.

## What I did

I took 15 real pull requests from the Martian Code Review Bench, three each from Sentry, Grafana, Cal.com, Discourse and Keycloak. People had already listed 53 known issues in them.

Each pull request was reviewed two ways: **with the plugin** (version 9780cf4, as it ships), and **just asking** in one paragraph to report every bug with its file and line. I ran both on Opus 5.5 and Fable 5.2, twice each: 120 reviews. All used Claude Code 2.1.325 with effort set to high.

Claude Code can quietly send a refused request to a different model. So I saved every request and reply, to know which model answered each one. A judge model, Claude Sonnet 5.5, graded with the test set's own method. The ranges come from re-drawing the 15 pull requests at random 10,000 times.

## What I found

**Just asking found about twice as many of the known issues.** On none of the 15 pull requests, on either model, did the plugin find more. The plugin cost about one and a half times as much.

![Bar chart of the share of known issues found. Claude Opus 5.5: 31 percent with the plugin, 66 percent just asking. Claude Fable 5.2: 32 percent with the plugin, 73 percent just asking.](https://research.sideofmydesk.com/edition-1/figures/fig-01-recall.png)

| Model | Setup | Share found (recall) | Share that matched (precision) | Findings per review | Cost per review |
|---|---|---|---|---|---|
| Opus 5.5 | Plugin | 31.1% [19.4, 45.5] | 53.2% | 2.1 | $0.60 |
| Opus 5.5 | Just asking | 66.0% [54.7, 78.6] | 21.3% | 11.3 | $0.41 |
| Fable 5.2 | Plugin | 31.6% [19.6, 44.8] | 43.1% | 2.5 | $1.75 |
| Fable 5.2 | Just asking | 72.6% [63.0, 83.3] | 26.5% | 10.1 | $1.14 |

One plugin review on Fable 5.2 was stopped by a safety filter, a false alarm. The other 119 finished.

**The idea that plugins age badly got some support. It is not proven.** The plugin's lead in share that matched fell from 31.9 points on Opus 5.5 to 16.6 on Fable 5.2, and that range does not include zero. On share found, the extra 6.1-point drop could be chance.

![Chart of the plugin's effect on Fable 5.2 minus its effect on Opus 5.5: 15 points lower on match rate, with a range that does not cross zero, and 6 points lower on known issues found, with a range that crosses zero.](https://research.sideofmydesk.com/edition-1/figures/fig-04-hypothesis.png)

**The plugin picks its own models.** When I chose Fable 5.2, 43% of the plugin's bill was for Opus 5.5 helpers, which did the bug hunting and double-checking. Moving up a model did not give a newer review.

![Bar chart of cost per review, split by the model that did the work. With the plugin on a Fable 5.2 session, 91 cents went to Fable 5.2 and 75 cents to Opus 5.5.](https://research.sideofmydesk.com/edition-1/figures/fig-06-cost.png)

**The older model often skipped the plugin's process.** Opus 5.5 started no helpers in 13 of its 30 plugin reviews. Those cost $0.28 against $0.85 and did about as well. The model chose when to skip, so this is a hint, not a fair test.

**The test set misses real issues.** A checker model judged 285 of the 543 unmatched findings real. That is 123 separate mistakes the test set does not list, against the 53 it does. A second AI review agreed with 24 of 30 sampled judgements, so these numbers are likely somewhat too high.

![Bar chart. The test set lists 53 mistakes for these 15 pull requests. A further 123 separate mistakes were judged real: 99 found only by just asking, 19 found by both, 5 found only with the plugin.](https://research.sideofmydesk.com/edition-1/figures/fig-12-new-issues.png)

## What the plugin does well

The plugin works like a filter. It reported a fifth to a quarter as many findings, and more of them were right. That is by design: its checkers are told to report only code that will not run, code that will definitely give wrong results whatever the input, and broken rules they can quote.

Count a finding as good if it matched a known issue or was judged real. Then the plugin was good 81% of the time on Opus 5.5 and 72% on Fable 5.2, against 63% and 67% for just asking. A team that sees every review note as an interruption may want that.

![Bar chart of average findings per review. With the plugin: 2.1 on Opus 5.5 and 2.5 on Fable 5.2. Just asking: 11.3 and 10.1. The larger part of each bar did not match a known issue.](https://research.sideofmydesk.com/edition-1/figures/fig-05-volume.png)

But just asking still gave about 7 good findings a review. The plugin gave fewer than 2.

## What a team should take from it

- **Read a plugin before you depend on it.** This one picks its own models, covers less than its name suggests, and has a help page written for an older design.
- **Record which model answers each request.** In a set-up test, a stand-in service answered as a different model. My saved files caught it. Claude Code's own summary still named the model I asked for.
- **Set the effort level yourself** when you compare models.
- **Test again when the model changes.** This study cost about $136.

## The limits of this study

- **It is small.** Fifteen pull requests give wide ranges. Only one difference between the models is clear.
- **It is a first look.** I set no pass marks ahead of time.
- **The test set is public.** The models may remember it, which could push scores up and tilt things toward the idea.
- **One wording.** A request for only the surest findings would likely give a shorter list.
- **An AI did the grading.** Claude Sonnet 5.5 matched and checked. The second review was Fable 5.2, one of the models tested. No person checked any of it. That is the biggest thing missing.
- **No clean code,** so false alarms were not measured.
- **Not a clean model comparison.** With the plugin, Opus 5.5 did the checking either way.
- **One plugin, one version, two models.**

## What to do next

A stronger test needs code changes made public after the models were trained, some code with no mistakes, every helper on one model, the plugin's older version, more models, grading checked by people, and pass marks set ahead of time. The method is cheap enough to run each time a new model comes out.

## Where to read more

- [The edition page](https://research.sideofmydesk.com/edition-1/)
- [The full paper (PDF)](https://research.sideofmydesk.com/edition-1/michel-2026-code-review-plugin.pdf)
- [Code and results](https://github.com/sideofmydesk/frontier-eval)
- [How the test works](https://github.com/sideofmydesk/frontier-eval/wiki)

*This study was done with a great deal of help from Claude Code, Anthropic's AI coding tool. Under the author's direction, it wrote the testing programs, ran the reviews, did the analysis, searched for earlier work, and drafted this paper. The author made the design choices and is responsible for what the paper says. The tool we tested, the models that graded it, and the assistant that helped are all Claude models from one company. Readers should keep that in mind.*

*The reviews cost about $136. This was paid from promotional credits that came with the author's Anthropic subscription. The author has no other financial tie to Anthropic. Anthropic had no part in this work.*

Independent work by Side of My Desk LLC; views are my own, not my employer's.
