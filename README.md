[![make.fun · gta6-trophy-rarity-forecast-01](https://make.fun/badge/gta6-trophy-rarity-forecast-01.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)
[![rank](https://make.fun/badge/gta6-trophy-rarity-forecast-01/intan151/makefun-gta6-trophy-rarity-forecast-01/rank.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)
[![reward](https://make.fun/badge/gta6-trophy-rarity-forecast-01/intan151/makefun-gta6-trophy-rarity-forecast-01/reward.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)

# GTA VI Trophy Rarity Forecaster

Forecasts, for every Grand Theft Auto VI trophy on PlayStation 5, its earn
rate on Exophase at the case's target time, as five quantiles per trophy
(5%, 25%, 50%, 75%, 95%), scored by pinball loss on the logit scale.

## How it works

`main.py` (Python 3.13, standard library only, no network) reads every
`<id>.in` case in the input directory and writes `<id>.out` to the output
directory.

The model works on the logit scale:

- History is collapsed to distinct snapshots (first-seen values), so long
  flat stretches where Exophase has not refreshed do not overweight the fit.
- With 3+ distinct snapshots, a recency-weighted (24 h half-life) least
  squares line of logit(rate) against ln(1 + hours since release) is fitted
  per trophy, slopes shrunk toward their tier mean and anchored at the last
  read (only the fitted *move* is used). That line is blended 25/75 with a
  saturating curve `r(t) = A - B·exp(-t/T)`, whose timescale `T` is chosen
  once for all trophies by pooled residual.
- With fewer than 3 distinct snapshots (flat launch-night history), no trend
  is invented: the median shifts down by 0.5 logit with spread σ = 1.0,
  expecting a refresh without pretending to know when.
- Spread: σ = √(residual variance + (0.5·√days)²), floored at 0.6 and capped
  at 2.0. The trophy at 100% is forecast at a 99% median with σ = 1.0.

## Run it

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

In the grader, `MAKEFUN_INPUT_DIR` / `MAKEFUN_OUTPUT_DIR` are unset and the
program reads `/input` and writes `/output`. Output is deterministic: two
runs on the same case write identical bytes.

## License

MIT — see [LICENSE](LICENSE).
