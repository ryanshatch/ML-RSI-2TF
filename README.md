# Multi-Resolution RSI with Machine Learning

**Developed by imaclone/Ryanshatch**  
**Last Updated:** August 21st, 2025  
**Current Version:** `6.3`  
**Pine Script:** `//@version=6`

A TradingView Pine Script v6 indicator that fuses my **ML-RSI.ai** pipeline with a classic **multi-timeframe RSI** structure.

This script combines:

- Multi-timeframe RSI
- Optional second RSI from a separate timeframe
- Optional moving average smoothing on RSI
- KNN-style nearest-neighbor blending on the primary RSI
- Optional post-processing filters
- Threshold bands, background highlights, and B/S cross labels
- Flexible line and bar coloring modes

One script, dual-resolution oscillators if desired, plus a machine-learning similarity engine and modular signal-processing layers.

---

## Authors

Developed by: **imaclone** / **Ryanshatch**

---

## What It Does

The indicator calculates a **primary RSI** using either:

- the current chart timeframe, or
- a custom user-selected timeframe

It can then optionally apply:

1. RSI smoothing with a selectable moving average
2. KNN-style blending using normalized feature similarity
3. A final post-processing filter pass

A **second RSI** can also be enabled from another timeframe for higher-timeframe confirmation or hierarchical confluence.

The output remains bounded to the standard RSI scale of **0-100**.

---

## Core Features

### 1. Multi-Resolution RSI

The primary RSI can run on:

- the chart timeframe, or
- a manually selected timeframe

This allows lower-timeframe charts to reference a slower RSI without changing the chart itself.

---

### 2. Optional Secondary RSI

A second RSI line can be enabled for confirmation.

Common use cases:

- intraday chart + daily RSI
- daily chart + weekly RSI
- current timeframe RSI + higher timeframe RSI regime check

The secondary RSI is intended as a **confirmation layer**, not a standalone trigger.

---

### 3. RSI Smoothing

The primary RSI can be smoothed before ML logic using one of these averages:

- SMA
- EMA
- DEMA
- TEMA
- WMA
- VWMA
- SMMA
- HMA
- LSMA
- ALMA

This helps reduce raw RSI noise before later processing.

---

### 4. KNN-Style Machine Learning Blend

The script includes a nearest-neighbor similarity engine that compares the current RSI state to historical RSI states using normalized features.

Feature embeddings include:

- RSI magnitude
- RSI momentum
- volatility surface
- regression slope
- price momentum vectors

In practical script terms, the feature set includes:

- normalized RSI
- RSI momentum
- RSI volatility
- RSI slope
- price momentum

The routine selects the closest historical matches using **top-k selection** rather than a full sort, then creates a weighted blend of those neighbor RSI values.

Important: this is **not** a forward-predictive AI model. It is a **KNN-style historical similarity blend**, which behaves more like adaptive smoothing based on similar prior conditions.

---

### 5. Advanced Filtering

After the KNN stage, the output can be filtered again using:

- None
- Kalman
- DoubleEMA
- ALMA

This creates a modular smoothing stack:

- **Kalman** for adaptive, lower-lag filtering
- **DoubleEMA** for stronger dampening
- **ALMA** for smooth convolution-style filtering

Too much filtering can improve visual cleanliness while increasing lag.

---

### 6. Visual Signal Layer

The indicator includes:

- Upper and lower RSI bands
- Optional midline at `50`
- Background highlight when RSI is above or below threshold bands
- Background highlight on band crosses
- Buy label when RSI crosses up through the lower band
- Sell label when RSI crosses down through the upper band

Signal logic:

- **B** flag when ML-RSI crosses upward through the lower threshold
- **S** flag when ML-RSI crosses downward through the upper threshold

These are event markers, not guarantees.

---

### 7. Color Modes

The main RSI line supports multiple color architectures:

- **None** — static neutral color
- **Trend-Following** / **Trend** — color changes based on being above or below the 50-line
- **Impulse** — color changes when outside the threshold bands

Optional bar tinting can be used for full-chart context.

---

## Inputs

## Timeframe Settings

- **Use chart timeframe**: uses the current chart resolution for the primary RSI
- **Primary RSI timeframe**: manual timeframe if chart timeframe is not used
- **Primary RSI length**: standard RSI length for the main RSI
- **Show 2nd RSI**: enables the secondary RSI line
- **2nd RSI: use chart timeframe**: uses current chart resolution for the second RSI
- **2nd RSI timeframe**: manual timeframe for the second RSI
- **2nd RSI length**: standard RSI length for the second RSI

---

## Levels & Visuals

- **Upper line**: upper threshold, default `70`
- **Lower line**: lower threshold, default `30`
- **Show mid line (50)**: shows the 50 RSI midpoint
- **BG highlight above/below bands**: highlights overbought and oversold zones
- **BG highlight on band crosses**: highlights threshold cross events
- **Show B/S labels on band crosses**: displays `B` and `S` markers

---

## RSI Base

- **Apply MA to RSI**: smooths RSI before ML processing
- **MA length**: length used by the selected moving average
- **MA type**: average type used for smoothing
- **ALMA sigma**: sigma parameter for ALMA

---

## KNN Machine Learning

- **Enable KNN on primary RSI**: turns the KNN blend on or off
- **K neighbors**: number of nearest historical matches to use
- **Lookback bars**: historical search window for similarity matching
- **ML weight vs RSI**: blend balance between raw/smoothed RSI and KNN estimate
- **# features (1-5)**: number of normalized features included in distance calculation

---

## Advanced Filtering

- **Apply extra smoothing**: enables the final post-processing filter
- **Filter**: selects `None`, `Kalman`, `DoubleEMA`, or `ALMA`
- **Filter strength**: controls filter intensity

---

## Coloring

- **Color mode**: selects `None`, `Trend`, or `Impulse`

---

## How the KNN Logic Works

The KNN block does the following:

1. Builds normalized feature values from the current RSI state
2. Compares that state to previous bars in the lookback window
3. Measures squared Euclidean distance across selected features
4. Keeps only the nearest `k` matches using top-k selection
5. Computes a weighted average of the matching historical RSI values
6. Blends that estimate with the current RSI using **ML weight vs RSI**

This creates a smoother, context-aware RSI that reacts to historically similar conditions rather than simply applying a fixed average.

---

## Signals and Interpretation

This indicator is designed to help with:

- spotting RSI regime shifts with less noise
- pairing lower-timeframe entries with higher-timeframe RSI context
- confirming overbought and oversold zones
- filtering standard RSI behavior with adaptive historical similarity
- visual trend confirmation using 50-line and threshold-band logic

Typical interpretation:

- use the **primary RSI** as the main decision layer
- use the **secondary RSI** for higher-timeframe confirmation
- use **B/S crosses** as structured events, not isolated trade commands
- adjust **ML weight** and **feature count** depending on whether you want a more classic or more adaptive oscillator response

---

## Usage Notes

- Raise **ML weight** and **feature dimensionality** for deeper similarity recognition
- Lower them for more classic oscillator behavior
- **Kalman** recursion gives adaptive, lower-lag smoothing
- **DoubleEMA** and **ALMA** produce stronger dampening
- A common setup is an **intraday primary RSI** with a **higher-timeframe secondary RSI** for regime anchoring

---

## Non-Repainting Behavior

The script uses `request.security()` with:

- `gaps=barmerge.gaps_off`
- `lookahead=barmerge.lookahead_off`

This prevents forward-looking data from being pulled into the current bar during multi-timeframe requests.

---

## Performance Notes

This version is optimized compared to older logic retained in earlier versions.

Main improvements include:

- top-k selection instead of full array sort for KNN neighbors
- squared distance instead of square root distance for ranking
- built-in `ta.wma()` instead of manual WMA loops
- conditional second RSI calculation only when enabled
- safer clamped filter lengths
- consolidated background coloring logic

Even so, larger **lookback** values and higher **K neighbors** values will still increase script cost.

---

## Suggested Starting Settings

For a cleaner daily setup:

- **Primary RSI length**: `14`
- **Apply MA to RSI**: `true`
- **MA type**: `ALMA`
- **MA length**: `2` to `3`
- **Enable KNN on primary RSI**: `true`
- **K neighbors**: `3` to `5`
- **Lookback bars**: `60` to `120`
- **ML weight vs RSI**: `0.20` to `0.35`
- **Apply extra smoothing**: `false` if you want less lag
- **Upper / Lower**: `70 / 30` for ranges, `60 / 40` for stronger trends

---

## Limitations

- This is not a predictive machine learning model in the modern sense
- The KNN block uses historical similarity, not future classification
- More smoothing can make signals cleaner but slower
- On low-history charts, KNN output is limited until enough bars exist
- Like all oscillators, it should not be used in isolation

---

## Installation

1. Open TradingView
2. Open the Pine Editor
3. Paste the script into a new indicator
4. Save the script
5. Click **Add to chart**
6. Adjust inputs to match your timeframe and trading style

---

## Changelog

### v6.3

- Added optimized top-k KNN selection
- Removed full-sort neighbor selection approach
- Replaced manual WMA with `ta.wma()`
- Added safer filter length handling
- Made second RSI conditional when disabled
- Added explicit non-repainting MTF request settings
- Simplified background highlight logic

### v6.2

- Prior version retained in comments at the bottom of the script for reference
- Used slower full-sort KNN logic and older background handling

### v6 Merge

- Unified the CM-style MTF RSI framework with my KNN-enhanced kernel and filter stack
- Consolidated multiple scripts into one composite indicator

---

## Credits

- MTF band logic inspired by earlier open-source frameworks
- ML kernel and implementation by **imaclone.x**

---

## License / Usage

Copyright (c) 2026 Ryan Hatch, ryanshatch, imaclone, IM A CLONE, and all of the other names associated with Ryan Hatch or imaclone as legal entities and individuals.

All rights reserved.

---

## Disclaimer

For research and algorithmic experimentation only. No signals are guaranteed.

> And please, for the love of God, **DYOFR**.
