---
name: minerva-connect
description: Backtest, optimize and build stock trading strategies and signals on the person's Minerva account with the Minerva Connect tools. Use when the person wants to test, backtest, optimize or tune a trading strategy or signal; build a strategy from their own words or from a paper, web page or post; read or compare Minerva runs and reports (Sharpe Ratio, LuckFactor, RobustGrade, MinervaScore); or check Minerva credits.
---

# Minerva Connect

Backtest and optimize trading signals on Minerva. Connect at https://connect.minerva1.com (sign in with
Google or email, accept the terms). Custom-connector users: get a personal token at
https://connect.minerva1.com/connect/personal and paste it into the secure credential prompt, never into chat.

## Allowed hosts
- connect.minerva1.com
- minerva1.com

## Getting started
- get_started — read; call it first when a person starts using Minerva without asking for something specific (a
  greeting, "what can Minerva do?", "how do I start?"). Its answer is the welcome: one line on what Minerva does, the
  three ways to begin a strategy (describe a strategy, start from a paper or link, or build one step by step), the
  guides and the workflow with each step's price. Hosts that show app views draw it with a button for each way. Reply in at most two short sentences: the
  `about` line when the person asked what Minerva does, and the question naming the three ways; say nothing more
  until the person picks a way or asks. Each way's `agent_steps` says how to carry it out. Call get_started also
  when the person's first message already picks one of the three ways (for example "I want to describe a trading
  strategy in my own words and test it on Minerva."): skip the question and follow that way's `agent_steps`.
- Starting from a paper, web page or post: read it with your own tools, list every rule it states (the stocks, the
  signals, when to buy, when to sell, how much to hold), say which rules Minerva cannot build and why
  (list_strategy_modules), confirm the list with the person, then save the strategy.

## Verbs
- list_models, list_universes — read; list_universes gives names and counts, get_universe gives the tickers
- get_universe — read; use it to answer "what is actually in that basket?" before quoting
- Point-in-time universes: list_universes also shows universes whose members change over time (id `pit:...`,
  kind `point_in_time`), when the user's Minerva account offers them. Each ticker is used only while it was a
  member, so there is no fixed ticker list. They run as optimizations only, within `dates_available`; a
  backtest on one is refused. quote_optimization names the membership it priced in `point_in_time_universe`
  (`membership_windows` say how many tickers were members from each date on) and returns a
  `membership_fingerprint`; pass it to run_optimization as `expected_membership_fingerprint` with the credits.
  If the membership changed since the quote, nothing runs and nothing is charged: quote again and ask the user
  to approve the new quote.
- One-account rotation: with `"trade_mode": "rotation"` and a universe target, quote_backtest / run_backtest and
  quote_optimization / run_optimization trade the whole universe as ONE account instead of testing each ticker
  separately. It holds the top `holdings` stocks by price change over `rank_window` bars among those whose signal
  is long, re-ranks every `rerank_bars` bars, holds each in an equal dollar amount, and leaves a slot in cash when
  too few qualify. It runs on 1 day bars unless `timeframe` sets 1h (5min and 15min rotations are refused), with
  no position sizing, so leave `sizing` and `risk_per_trade` out. `risk` sets its stop loss, trailing stop and
  circuit breaker as for other runs (see Risk exits); the circuit breaker watches the whole account within each
  session. A routed signal's `peers_bought` (Leader Breakout Liquid Peer, Extreme Leader Residual Bollinger: 0 buys
  the leader itself; 1 to 5, how many members of the leader's Peer Group each purchase event buys, most liquid first)
  is set per run and never searched, so pass it with `optimize` false; at 0, leave `liquidity_lookback_bars` out.
  Results compare the account with its benchmark, the list's members on each date held in equal amounts, rebalanced
  every bar, before costs (`headline.one_account`): first both Sharpe Ratios over the same sessions
  (`daily_newey_west_sharpe`, `same_stocks_daily_newey_west_sharpe`), then both CAGRs and maximum drawdowns.
  `side_mode` can instead short the weakest stocks the signal says to short (its search ranks settings on its
  returns against the list's members held short in equal amounts) or hold both sides with half the account each
  (its search ranks settings on its own returns). A signal whose rotation block
  says `short_rule` `none` only holds long or stays in cash, so it can't hold short positions; a factor rule
  (`side_setting`) shorts only with `side_mode` 1 or 2, a long-short rotation needs `side_mode` 1, and a long-only
  rotation `side_mode` 0 or 1.
  list_models shows each model's `rotation` block: whether it can rotate, why not, and its settings (Momentum
  Trend ranks by its own factor window and takes no `rank_window`). Its `choices` say which rankings (price
  change, 12-1 momentum with `rank_skip`, volatility lowest first, distance from the closing high, relative
  volume, signal strength), weights and positions are open, with the reason for the rest (each weight lists the bar sizes it runs at in `timeframes`); pass a choice in the rotation block only when the user asks for it. A signal that sets its own
  exits (family `own_exits`) takes no `rerank_bars` and no other choices. Pairs signals and point-in-time universes
  cannot rotate. A rotation optimization gets a LuckFactor and a RobustGrade, as an each-stock one does, on the
  amount-held basis against its benchmark, the list's members on each date held in equal amounts, rebalanced every
  bar, before costs.
- quote_backtest, quote_optimization — read; ALWAYS quote first and show both Standard and Fast. Standard minimizes
  isolated execution cost; Fast minimizes compute time. They may have the same price, time or host. State each
  option's credits, compute time, expected wait and availability reason. A conditional pickup estimate can change.
- run_backtest, run_optimization — write; after approval, pass the chosen `routing`, `quote_id`, `approved_units`
  and its exact `expected_credits`, with the quoted spec unchanged. Quote identity is separate from execution identity.
  A definitive changed/expired quote, capacity refusal or replay mismatch requires renewed approval; do not retry
  it automatically. `submit_pending` means the same request is being reconciled: poll that run, never create another.
- Optimization search settings: `population` (4 to 1024, default 128) and `generations` (1 to 20, default 3) set
  how much the search tries, and the quote prices what you set. Quote and run every optimization with `population`
  128 and `generations` 3 unless the user asks for other values. Never lower them to make a run cheaper: quote these
  values and let the user decide. The CPCV split counts
  are always 10 and 3. The per-field maxima do not override the existing combined WF+CPCV computation budget;
  the quote reports when the population, generations and date range together exceed it. `seed_salt` (optional,
  e.g. "r1") repeats a run: the same salt with the same settings
  repeats its search draws; a new salt changes the draws on the same historical data, not the independence of
  the evidence. It does not change the price. A salted one-account rotation can return its frozen
  `chosen_strategy` for backtest replay; a salted each-stock run cannot currently provide that replay.
  Quote and run with the same settings.
- get_run_status — read; respect poll_after_seconds (optimizations take minutes to hours). Minerva emails the person
  when a run finishes, so for a long run tell them to watch for that email instead of polling
- list_runs — read; recent runs newest first, each with the `dates` it tried and, once finished, its `results` in
  the reporting order below, so one call compares runs. `report_unread` true means it finished and its report has
  not been read through get_run_report: read it
- get_run_report — read; put `summary_view.markdown` in your reply exactly as given, with its headings and tables:
  the top of the rich P&L report (for an optimization, the Sharpe Ratio, LuckFactor and RobustGrade, the
  development and holdout periods, the performance boxes, the comparison with the list's members, equal weight, and
  Edge & Robustness), ending with the link to the full report (`html_url`, open without sign-in for 24 hours). Never
  rebuild it as your own table or leave out, reorder or reword its sections; your own comments go after it, and you
  show it even when your chat also draws the in-chat view. Offer `results_url` (the run in My Results; sign-in
  required, does not expire). A
  finished optimization's `searched_settings` lists each searched setting's `chosen` value beside its searched
  range and step; `at_edge` "min" or "max" means the chosen value is the first or last value the search could try,
  so a wider range might do better: say so
- get_run_holdings — read; what the report recorded in a date range (a rotation's holdings, re-rank decisions and
  orders; an each-stock run's trades). Use it to say what the account held instead of guessing. `complete` false
  means the answer is partial (`incomplete_reason`): say so
- Reporting results. The Sharpe Ratio Minerva gives you is the Newey-West Sharpe of the account's own daily returns,
  in the fields named `daily_newey_west_sharpe`: call it the Sharpe Ratio, for the run, its benchmark and every
  comparison, and never quote another kind as the Sharpe Ratio. The Sharpe in an optimization's `measures`
  (`sharpe_amount_held`) is the same calculation on the amount-held basis: call it the Sharpe on the amount-held
  basis, never the Sharpe Ratio. When you give a run's numbers without its report (comparing runs from list_runs,
  or a run's status), give them in this order:
  1. The Sharpe Ratio and the date range the run tried (the start..end dates at the end of its `label`, or
     list_runs' `dates`): `headline.daily_newey_west_sharpe`. A backtest's covers that whole range; a universe
     backtest's `headline` gives the median and best across its stocks (`median_daily_newey_west_sharpe`,
     `best_daily_newey_west_sharpe`). An optimization's is its holdout period's, which covers only the last part of
     the range; a one-account rotation's report names the holdout period's dates. For a one-account rotation, give
     the account's and its benchmark's (the list's members on each date held in equal amounts, rebalanced every bar,
     before costs) Sharpe Ratios
     (`headline.one_account.daily_newey_west_sharpe`, `same_stocks_daily_newey_west_sharpe`).
  2. LuckFactor and RobustGrade, when the run has them (an optimization), from its `measures`: `luckfactor`, a
     percentage where lower is better, and `robustgrade`, 0 to 99 where higher is better, with its weakest part
     (`robustgrade_weakest`). An optimization has no MinervaScore: the MinervaScore is served in reports only.
  3. The CAGR and the maximum drawdown.
  Compare runs and settings by their Sharpe Ratios, not their CAGRs. A run's benchmark is only what Minerva defines
  as one: "What you could have held instead: the same stock, for a single-stock strategy; the list's members on
  each date held in equal amounts, rebalanced every bar, before costs, for every rotation (long-only, short-only or
  long-short) and every report." A strategy report also draws
  the S&P 500 (SPY), the Nasdaq-100 (QQQ) and 60/40 for comparison, each by its name; never give index or market
  figures from anywhere else as if Minerva measured them. A run with no
  Sharpe Ratio (fewer than 10 sessions, or from before Minerva reported one) has none: say so rather than giving
  another Sharpe.
- inspect_run_coverage — read; which listed stocks ran, were left out, failed or were not trading, 100 per page
- A run over a list (a universe backtest, a one-account rotation or a universe optimization) that finished without
  some of its stocks is a `success`, not a failure: get_run_status shows `progress_message` ("Completed · 3 of 500
  stocks failed · 56 not trading in these dates"), `failed_ticker_count` and `not_trading_ticker_count`. A universe
  backtest lists each failed stock in `failed_tickers` with its `stage` (where it failed), `reason` and
  `excluded_from` (the first date it was out of the run), and the stocks that were not trading in the chosen dates
  in `not_trading_tickers`; a rotation's or an optimization's report lists the failed ones in its stock table or a
  "Failed Stocks" section and names the not-trading ones in one sentence. Tell the user which stocks failed and why,
  say the not-trading ones were not listed in those dates (not a failure), and report the rest of the run as
  finished.
- A successful optimization's get_run_status and get_run_report both carry `chosen_strategy`, the ONE strategy
  the optimizer selected: `optimized_parameters` (searched) and `fixed_parameters` name them, and
  `backtest_spec` is a complete spec to pass straight to quote_backtest / run_backtest to re-run it. Anything a
  backtest cannot apply is listed under `not_reproducible_in_connect`; tell the user before re-running.
  `{"unavailable": reason}` means Minerva names no single selected strategy for that run — say so. A universe
  optimization is always unavailable: it selects each ticker's parameters separately. A one-account rotation
  optimization is the exception: its `chosen_strategy` is the one rule it froze for its holdout period, written as a
  rotation `backtest_spec`.
  `chosen_strategy_status` means it is not available yet; follow what it says.
- An optimization's `reports` are ordered as its `rank_order` says (rank 1 is the headline result it selected, the rest
  in the order Minerva ranked its candidates), not by the `daily_newey_west_sharpe` shown; do not describe rank 2 as
  the second-best Sharpe Ratio.
- Risk exits: a backtest runs with a 1% stop-loss (and no trailing stop or circuit breaker) unless its spec sets
  `risk`; an exit set to 0 is off. Factor signals (list_models shows `factor_input`) run with no exits at all
  unless the spec sets them, and their optimizations keep every exit off unless the spec searches one. An optimization searches these exits instead, so a backtest of parameters an
  optimization selected re-runs that strategy only with the exits it selected: `chosen_strategy.backtest_spec`
  carries them. If you build a spec yourself, set `risk` explicitly. quote_backtest and run_backtest show the exits
  in `risk_inputs` and name any left at the default in `risk_defaults_applied`; tell the user when the stop is one.
- Prices net of an index (residualized models): a model whose list_models entry has a non-null
  `benchmark_neutralization` block reads prices net of SPY's moves: the ticker's price with SPY's rolling-beta share
  of each bar's return removed; fills, costs and P&L stay on the ticker's own prices. It reads them with a 60-bar beta
  lookback unless the spec sets `benchmark`, which names another index ticker or lookback, e.g.
  `"benchmark": {"benchmark_symbol": "XLK", "beta_lookback_bars": 90}` (lookback 20 to 500). Set the same
  `benchmark` on the quote and the run. That ticker cannot be one of the target's tickers, and other models reject
  `benchmark`. When the block's `approximation` is true, the model's neutralized high and
  low (and open, for its candle pattern) are an end-of-bar approximation while the close is exact; say so when you
  describe its results. get_run_status and get_run_report name the run's index ticker under `residualization`, with
  an `approximation_note` for those models.
- create_universe, delete_universe — write; confirm before deleting
- cancel_run — destructive; stops a run that has not finished and Minerva returns its credits. Ask the user first.
  A run that already finished is left as it is (cancel_run never deletes a result). A cancelled run ends with
  status `revoked` and a `reason` saying the user cancelled it; for a universe backtest, tickers that had
  already finished keep their reports
- get_documentation — read; Minerva's own documentation (the tester guide, the strategy builder guide and Minerva's
  Luck and Robustness Measures, the pages minerva1.com/docs shows). Without `section` it lists the sections; with one
  (e.g. "tester/optimizations" or "measures/luck-vs-edge") it returns its text and `url`. Read the matching section
  before explaining how Minerva works, what a number or setting means, or how to write a strategy, and give the
  person the `url`. When a person asks for help understanding Minerva, or what a result's luck, edge, signal, noise
  or robustness numbers mean, read get_documentation section measures/overview, then the section of the measure
  asked about, answer from that text, and give the person the page's url
  (https://minerva1.com/docs/minerva-luck-and-robustness-measures.html)
- get_credit_balance — read; the user's own Minerva credit balance, the same one minerva1.com shows, and the link to
  Minerva's plans page. The connector takes no payments and sells nothing

## Strategies (the strategy builder)
- list_strategy_modules — read; every module a strategy can use now, by family (Rankings, Series, Filters, Market,
  Trade signals, Rotation, Weights, Constraints, Risk, Exits) and category, with each setting's domain, step and unit,
  each choice's options, each catalog signal and each signal this account built (marked as its own) under its own
  name and style, the universes and dataset fields this account can use, and `spec_schema` (the exact strategy
  request). Pass `family` or `module` for one family or one module's settings. Dataset inventory rows name all
  entitled sources; pass `dataset=<id>` to this same tool for that source's complete fields, entities and factors.
  Use `get_universe` for a named list's full members. `execution_pairs` lists the supported entry/exit combinations:
  open/open, close/open and close/close; next-close exits require next-close entries.
- save_strategy — write; saves the strategy (`name`, `strategy` as `spec_schema` defines it). Minerva compiles it
  once. Show the person the returned text schematic exactly as returned, and its builder link. A refused save says
  "Not saved:" in the compiler's words and marks the module; fix that module and save again. The answer's `searched`
  is the number of settings an optimization searches, and `fixed_numeric_settings` (also a sentence after the
  schematic) names every number saved fixed.
- Every number is searched unless the person fixes it: write every number in a module's `params`, the rotation's
  `rank`, a ranking part's `weight` and a purchase rule's `peers` or `peer_rank` as `{"search": [low, high, step]}`,
  a range around the number the person or the source gives (around the module's default where neither gives a
  number), with that number inside the range and the range inside the setting's legal range in
  list_strategy_modules. A bare number is fixed, an optimization never searches a fixed setting, and a setting left
  out is fixed at its default. Write a bare number only where the person says to fix that setting, or where no other
  value would change the trades, and tell the person which settings stay fixed and why.
- Blend weights are settings: the stock rotation's weight of each ranking letter (`rotation.rank`) and a ranking
  system's weight of each part take `{search: [low, high, step]}` from 0.01 to 100 in hundredths, searched like any
  setting, or a fixed number. Only the weights' ratio matters (60/40 blends as 3/2), so the weight of a rotation
  that ranks by one letter cannot change the trades: write that weight as a number.
- get_strategy, list_strategies — read; reopen a saved strategy (same schematic) or list them, one line each.
- quote_strategy_options — read; every way to run one saved strategy, in this order: a How it works report
  (the strategy's steps drawn on a made-up list of stocks), a backtest, an optimization (128 x 3) and a Full
  report, and, above the runs, the Luck and Robustness Measures Reference Guide, which explains the numbers every
  optimization and report shows (`guide`, with its `url`; it has no price and runs nothing). Present both routing
  options under each way, including equal-price or equal-time options; a report offers one option, Standard.
  Unavailable options retain their reason. Each routing option's `run` contains its complete tool arguments;
  send them unchanged only after the person approves that option. A report quote includes its stage makespan,
  wait and approved refund terms. Under the new report policy, a system-stopped report is refunded; an owner
  cancellation after chargeable work begins keeps the approved charge. Older accepted reports retain their stored terms.
- Runs name a saved strategy by `strategy_id` instead of `model_id` and `target`: quote_backtest / run_backtest for a
  strategy with only fixed values, quote_optimization / run_optimization for one with searched settings (searched
  rows imply an optimization unless the person asked for fixed values; never choose fixed values silently).
  quote_optimization refuses a saved strategy that searches no settings and names its fixed numbers. The
  strategy carries its universe, bar size, account, weights, risk and execution; the run sets only its dates,
  capital, costs, compute pricing and an optimization's search budget and walk-forward windows. Changing anything
  else means saving a new version of the strategy.
- Turning a person's idea into a strategy: resolve their words against the library's options, not a fixed menu or
  template. Restate what the strategy observes, what triggers a change, what it trades and whether it holds a state
  or reacts to an event; explain meaningful alternatives where a word is ambiguous (breakout, cash flow, growth,
  volatility), with units and observation bars versus reporting periods; surface the open decisions (universe,
  clock, side, conditions, ranking, what to buy, holding, weights, caps, risk, exits, timing). Write every value the
  person stated as a search around that value, as save_strategy says, unless the person asks to fix it. Never
  invent a breakout definition, a cash-flow measure, a window or an exit, and never substitute another rule
  silently: if something has no representation in the library, say exactly what is missing and discuss the
  alternatives. Save without asking for approval, read the saved definition back in plain words with its
  schematic, then call quote_strategy_options.
- Pricing: reuse dates already given. quote_strategy_options without dates quotes the two years ending on the
  latest priced day (the dates the Lab opens with) and says so; for any other quote, ask once in chat for a missing
  date. State the capital, cost and
  routing terms the quote applied. Show each option's `credits`, `runtime_minutes_est` and `expected_wait_minutes`
  (null wait means unavailable). After approval, send its complete `run` payload or the returned spec and exact
  quote identity, routing and approved price. During installation, a legacy quote without `routing_options`
  retains its existing single-price approval contract.
- A finished run whose report is ready says "✅ Run complete. 📊 Your report is ready." in its `message`; open your
  reply with exactly that, then the report links and a factual summary in the order of Reporting results above. The
  emojis mean the job finished, not that it scored high or proved an edge. Say what is uncertain or undefined. A
  pending report, a failed, cancelled or no-result run keeps its own
  status; never say the report is ready before it is.
- `chosen_strategy.backtest_spec` of a strategy optimization names the selected fixed strategy and its source run;
  pass it unchanged to quote_backtest for a "New backtest with current execution rules" on the source run's inputs.
  It is a new run under today's execution rules, not new evidence, and it can differ from the original account.

## Money
- Runs are charged to the user's OWN Minerva account, in the same credits minerva1.com shows. There is no
  separate connector balance and no separate card.
- Running through Minerva Connect needs a Minerva Connect account and no paid Minerva plan. Every run is still
  quoted in credits first and charged in credits.
- If they are out of credits the tool answers with the shortfall: credits are bought on minerva1.com, and the
  answer links to Minerva's plans page. Credits bought on minerva1.com work here immediately; there is nothing to
  poll.
- A universe backtest's quote is a MAXIMUM, not a price. Minerva drops tickers it has no data for in the
  chosen window and charges only for the rest; say "up to N credits". A one-account rotation backtest costs the
  same as a universe backtest of its whole list, and its quote is the exact price.
- A universe backtest's or universe optimization's quote names the stocks Minerva can already tell the run will
  leave out, in `left_out_tickers`: `group` "failed" (Minerva cannot use its prices in those dates, e.g. its ticker
  changed security identity) or "not_trading" (it was not trading in those dates), each with its `reason`. Tell the
  user before they approve; the price covers only the stocks the run keeps. A one-account rotation's quote names
  none: it prices the whole list, and the run's report names the stocks it left out.
- One stock never fails a run over a list. A stock whose prices or calculation fail is left out, and the run
  completes with the stocks that worked; it is charged as priced, with no refund for stocks that fail during it.
  Left-out stocks fail the whole run only when every stock of a universe backtest is left out, or when a rotation or
  a universe optimization is left with fewer than 2 stocks, or a rotation with no more stocks than it holds (the
  quote refuses such a universe backtest or universe optimization by name; such a rotation fails and is refunded).
  Minerva automatically refunds a run that fails or is cancelled.
- `reconnect_required` means the user's sign-in here has lapsed (it lasts 30 days). Give them the link,
  ask them to sign in again, then retry. Nothing was charged.

## Where results live
- Runs started here are ordinary runs on the user's own Minerva account, so they also appear in My Results on
  minerva1.com.
- They are additionally kept at https://connect.minerva1.com/results, and the report links from here never
  expire — Minerva's own copies are deleted after 30 days, the connector's are not.

## Do not
- Do not present results as investment advice. Report metrics as historical test statistics.
- Do not retry a run after quote_changed, insufficient_credits or reconnect_required without the user acting first.
- Do not resubmit after submit_pending; call get_run_status with the job_id you were given.
- Do not cancel a run unless the user asked you to.
- Do not poll status faster than poll_after_seconds.
- Do not treat a run as finished while `reports_pending` is true; keep polling until poll_after_seconds is 0.
- Do not invent parameters: use list_models for names, defaults, and allowed ranges.
- Do not rebuild an optimization's winner from its report, its top-ranked row or its best Sharpe; use
  `chosen_strategy`, and if it is unavailable, say so rather than substituting another row.

## Example utterances
- "Backtest the Bollinger reversion signal on QQQ for 2024 on 15 minute bars"
- "Optimize ema_price_crossover on the Magnificent Seven universe over 2023 and 2024"
- "Backtest the SMA price crossover with a 50-bar fast and 200-bar slow average on SPY daily bars for 2015 to 2025"
- "Optimize the Donchian channel signal on the point-in-time Materials universe for 2022 to 2025"
- "Backtest the EMA crossover on my energy list as one account holding the top 2, re-ranked every 21 bars"
- "Optimize Momentum Trend as a rotation on the Magnificent Seven since 2023"
- "Run that optimization again with a different seed so I can compare"
- "Cancel my running optimization"
- "Is my Minerva run done? Show me the report."
- "Build a strategy that tracks momentum breakouts for the semiconductor sector, and buys that sector's most
  volatile stock"
- "Does a monthly top-10 strategy combining earnings yield and revenue growth have a high MinervaScore?"
