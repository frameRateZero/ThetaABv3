# θ WAVE v3.0 — Attentional Blink Lab

A mobile-first Progressive Web App for measuring the attentional blink using a rapid serial visual presentation (RSVP) twitch paradigm. v3.0 replaces the adaptive QUEST/EIG engine with a fully structured SOA × contrast grid design and offline curve fitting.

---

## Files

| File | Purpose |
|------|---------|
| `index.html` | Complete app — all logic self-contained |
| `sw.js` | Service worker — network-first caching, offline fallback |
| `manifest.json` | PWA manifest — add-to-home-screen support |

Deploy all three to the same directory on any HTTPS static host (GitHub Pages, Netlify, etc.).

---

## Paradigm

**RSVP Twitch Task**

A stream of letters appears at screen centre at a fixed rate. Occasionally a digit (T1: 1–4) appears. Shortly after, a second digit (T2: 6–9) appears at a variable SOA. The participant taps once for T1, once for T2 if seen.

- T1 tap confirms attentional capture and provides RT
- T2 detection across SOA × contrast combinations maps the blink
- Trials where T1 is not tapped are discarded
- No catch trials, no ITI prompts, no explicit response windows shown

---

## Design — Structured Grid (v3.0)

Rather than adaptive sampling, every trial is pre-specified at session start.

**Default grid:**

| Parameter | Default | Range |
|-----------|---------|-------|
| SOAs | 10 values, 200–800ms | 200ms floor, soaMax ceiling |
| Contrasts | 10, 20, 35, 55, 75, 90% | Fixed 6 levels |
| Reps per cell | 4 | 2–8 (slider) |
| Total trials | 240 | 120–480 |

SOA values are quantised to frame boundaries at calibration time. The complete `SOAs × contrasts × reps` trial list is Fisher-Yates shuffled at session start — fully randomised, no blocking.

**SOA floor at 200ms.** Below this, T2 suppression is dominated by backward masking from T1, not the attentional blink oscillation. Excluded entirely.

**Contrast range 10–90%** ensures the psychometric function is bracketed above and below threshold at every SOA, and confirms whether suppression is absolute (masking) or threshold-shifted (attention).

---

## Settings

| Setting | Description | Default |
|---------|-------------|---------|
| Stream Speed | Item duration in ms (quantised to frames) | 75ms at 120Hz |
| T1 Font Size | Size of T1 digit | 96px |
| SOA Max | Upper bound of SOA range | 800ms |
| Reps per Cell | Repetitions of each SOA × contrast combination | 4× |
| Line Mask | Draw line fragments after each digit | Off |

The reps slider shows the resulting total trial count so you can plan session length before starting.

---

## Offline Curve Fitting

At session end a damped sine is fitted to the per-SOA threshold estimates:

```
threshold(SOA) = base + A × sin(2π × f × SOA / 1000) × exp(−SOA / τ)
```

**Per-SOA threshold:** For each SOA, detection rates across contrast levels are linearly interpolated to find the 75% crossing point.

**Grid search** over:

| Parameter | Range |
|-----------|-------|
| f (Hz) | 2.5–8.0, 12 values |
| A (% contrast) | 5–55, 11 values |
| base (% contrast) | 3–24, 5 values |
| τ (ms) | 150–1000, 6 values |

Minimises sum of squared residuals. **RMSE < 5%** = good fit; **> 10%** = poor fit (check raw heatmap directly before interpreting Hz estimate).

---

## Results Charts

**Detection heatmap** — rows = contrast, columns = SOA, colour = p(detect). Red = 0%, amber = 50%, cyan = 100%. A vertical suppression band marks the blink trough; a horizontal floor band at low contrast confirms the psychometric range is calibrated correctly.

**Threshold curve** — empirical 75%-crossing dots per SOA with fitted damped sine overlay. T1/T2/T3 trough markers shown where predicted troughs fall within the measured SOA range.

---

## CSV Output

Filename: `theta_v3_{timestamp}.csv`

### Trial rows

```
Trial, SOA_ms, T2_Contrast_pct, T1_Char, T2_Char,
T1_Tapped, T1_RT_ms, T2_Detected, T2_RT_ms, Valid
```

- `Valid=0` — T1 not tapped; trial excluded from threshold estimation
- `T2_Detected=0` — T1 tapped, T2 missed (or genuinely suppressed)

### Summary blocks

**Settings** — FPS, item duration, SOA range, grid contrasts, reps per cell, mask on/off

**Threshold_curve** — per-SOA 75% threshold estimates:
```
SOA_ms, Threshold_pct
200, 68.3
300, 41.7
...
```

**Fit** — damped sine parameters:
```
Freq_Hz, Tau_ms, Amplitude_pct, Base_threshold_pct, Fit_RMSE_pct
Trough1_soa_ms, Trough1_threshold_pct
Trough2_soa_ms, Trough2_threshold_pct
Trough3_soa_ms, Trough3_threshold_pct
```

**Session** — valid/discarded counts, mean and median T1 RT

---

## Interpreting Results

**Frequency estimate reliability** scales with reps per cell:

| Reps | Trials | Use |
|------|--------|-----|
| 2 | 120 | Pilot / parameter check |
| 4 | 240 | Standard session |
| 6–8 | 360–480 | Publication quality |

**Trough 2 and 3 visibility** depends on τ. At τ > 500ms, trough 2 amplitude is >50% of trough 1 and clearly measurable. At τ < 300ms, trough 2 is shallow. To capture trough 3 at ~4 Hz, extend SOA max to at least 1000ms (trough 3 ≈ 750ms).

**High-contrast misses** (75–90% contrast, no detection) indicate absolute suppression — sensory masking territory rather than threshold elevation. These anchor the top of the psychometric function and distinguish masking from attentional gating.

**T1 RT** is a quality indicator. Mean RT > 550ms or >30% of trials above 600ms suggests fatigue or stream speed too high. Consider shortening sessions or slowing the stream.

---

## Deployment & Requirements

- Any HTTPS static host; no server-side logic
- No data leaves the device
- iOS: Safari → Share → Add to Home Screen for full-screen PWA
- Android: Chrome → Add to Home Screen

**Display:** 60Hz minimum; 120Hz recommended (SOA step = 8.3ms vs 16.7ms). At 75ms/item on a 120Hz display, SOA quantisation error is ±4ms — negligible for theta-range measurements.

---

## Version History

| Version | Key changes |
|---------|-------------|
| **v3.0** | Structured SOA × contrast grid · offline damped sine fit · detection heatmap · reps-per-cell setting · SOA floor 200ms · no adaptive engine |
| v2.x | QUEST/EIG adaptive engine · contrast-threshold surface model · Latin square sweep warmup · trough-targeted exploration · async EIG |
| v2.0 | Twitch paradigm · T1 RT weighting · adaptive contrast per SOA bin · line mask option |

---

## References

- Shapiro, Raymond & Arnell (1994). Attention to visual pattern information produces the attentional blink in RSVP. *JEP: HPP*, 20(2), 357–371.
- Bonnefond & Jensen (2012). Alpha oscillations serve to protect working memory maintenance against anticipated distractors. *Current Biology*, 22(20), 1969–1974.
- Wyble, Potter, Bowman & Nieuwenstein (2011). Attentional episodes in visual perception. *JEP: General*, 140(3), 488–505.
