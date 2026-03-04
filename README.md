# Double TF RSI w/ ML - imaclone.x

A TradingView Pine Script v6 indicator that combines:

- Multi-timeframe RSI
- Optional second RSI from a separate timeframe
- Optional moving average smoothing on RSI
- KNN-style nearest-neighbor blending on the primary RSI
- Optional post-processing filters
- Threshold bands, background highlights, and buy/sell cross labels

This script is built for traders who want a more refined RSI view without changing the core 0-100 RSI structure.

## Version

- Current version: `6.3`
- Pine Script: `//@version=6`

## Authors

Developed by:
- Ryanshatch / 0ximaclone

## What It Does

The indicator calculates a primary RSI using either:

- the current chart timeframe, or
- a custom timeframe

It then optionally applies:

1. RSI smoothing with a selectable moving average
2. KNN-based blending using normalized feature similarity
3. An additional smoothing/filter pass

A second RSI can also be shown from another timeframe for higher-timeframe or confirmation context.

The output remains bounded to the standard RSI scale of `0-100`.

## Core Features

### 1. Multi-timeframe RSI

The primary RSI can run on:

- the chart timeframe, or
- a manually selected timeframe

This allows lower-timeframe charts to reference a slower RSI without changing the chart itself.

### 2. Optional Secondary RSI

A second RSI line can be enabled for confirmation.

Common use cases:

- intraday chart + daily RSI
- daily chart + weekly RSI
- current timeframe RSI + higher timeframe RSI trend check

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

### 4. KNN-style ML Blend

The script includes an optimized nearest-neighbor routine that compares the current RSI state to past RSI states using normalized features.

Features used:

- Normalized RSI
- RSI momentum
- RSI volatility
- RSI slope
- Price momentum

It selects the closest historical matches using top-k selection instead of a full sort, then creates a weighted blend of those neighbor RSI values.

Important: this is a KNN-style similarity blend, not a forward-predictive AI model. It acts more like adaptive smoothing based on similar historical states.

### 5. Optional Filtering

After the KNN stage, the result can be filtered again using:

- None
- Kalman
- DoubleEMA
- ALMA

This can make the final line cleaner, but too much filtering may increase lag.

### 6. Visual Signal Layer

The indicator includes:

- Upper and lower RSI bands
- Optional midline at 50
- Background highlight when RSI is above or below threshold bands
- Background highlight on cross events
- Buy label when RSI crosses up through the lower band
- Sell label when RSI crosses down through the upper band

### 7. Color Modes

The main RSI line can be colored using:

- `None` - static neutral color
- `Trend` - color changes based on being above or below 50
- `Impulse` - color changes when outside the threshold bands

## Inputs

## Timeframe Settings

- **Use chart timeframe**: Uses the current chart resolution for the primary RSI
- **Primary RSI timeframe**: Manual timeframe if chart timeframe is not used
- **Primary RSI length**: Standard RSI length for the main RSI
- **Show 2nd RSI**: Enables the secondary RSI line
- **2nd RSI: use chart timeframe**: Uses current chart resolution for the second RSI
- **2nd RSI timeframe**: Manual timeframe for the second RSI
- **2nd RSI length**: Standard RSI length for the second RSI

## Levels & Visuals

- **Upper line**: Upper threshold, default `70`
- **Lower line**: Lower threshold, default `30`
- **Show mid line (50)**: Shows the 50 RSI midpoint
- **BG highlight above/below bands**: Highlights overbought/oversold zones
- **BG highlight on band crosses**: Highlights threshold cross events
- **Show B/S labels on band crosses**: Displays `B` and `S` markers

## RSI Base

- **Apply MA to RSI**: Smooths RSI before ML processing
- **MA length**: Length used by the selected moving average
- **MA type**: Average type used for smoothing
- **ALMA sigma**: Sigma parameter for ALMA

## KNN Machine Learning

- **Enable KNN on primary RSI**: Turns the KNN blend on or off
- **K neighbors**: Number of nearest historical matches to use
- **Lookback bars**: Historical search window for similarity matching
- **ML weight vs RSI**: Blend balance between raw/smoothed RSI and KNN estimate
- **# features (1-5)**: Number of normalized features included in distance calculation

## Advanced Filtering

- **Apply extra smoothing**: Enables the final post-processing filter
- **Filter**: Selects `None`, `Kalman`, `DoubleEMA`, or `ALMA`
- **Filter strength**: Controls filter intensity

## Coloring

- **Color mode**: Selects `None`, `Trend`, or `Impulse`

## How the KNN Logic Works

The KNN section does the following:

1. Builds normalized feature values from the current RSI state
2. Compares that state to previous bars in the lookback window
3. Measures squared Euclidean distance across selected features
4. Keeps only the nearest `k` matches using top-k selection
5. Computes a weighted average of the matching historical RSI values
6. Blends that estimate with the current RSI using `ML weight vs RSI`

This gives a smoother, context-aware RSI that reacts to historically similar conditions.

## Non-Repainting Behavior

The script uses `request.security()` with:

- `gaps=barmerge.gaps_off`
- `lookahead=barmerge.lookahead_off`

That prevents forward-looking data from being pulled into the current bar during MTF requests.

## Performance Notes

This version is optimized compared to the older commented-out logic kept at the bottom of the script.

Main improvements include:

- top-k selection instead of full array sort for KNN neighbors
- squared distance instead of square root distance for ranking
- built-in `ta.wma()` instead of manual WMA loops
- conditional second RSI calculation only when enabled
- safer clamped filter lengths
- consolidated background coloring logic

Even so, larger `knnLookback` values and higher `K neighbors` values will still increase script cost.

## Suggested Starting Settings

For a cleaner daily setup:

- `Primary RSI length`: `14`
- `Apply MA to RSI`: `true`
- `MA type`: `ALMA`
- `MA length`: `2` to `3`
- `Enable KNN on primary RSI`: `true`
- `K neighbors`: `3` to `5`
- `Lookback bars`: `60` to `120`
- `ML weight vs RSI`: `0.2` to `0.35`
- `Apply extra smoothing`: `false` if you want less lag
- `Upper/Lower`: `70/30` for ranges, `60/40` for strong trends

## Limitations

- This is not a predictive machine learning model in the modern sense.
- The KNN block uses historical similarity, not future classification.
- More smoothing can make signals cleaner but slower.
- On low-history charts, KNN output is limited until enough bars exist.
- Like all oscillators, it should not be used in isolation.

## Installation

1. Open TradingView.
2. Open the Pine Editor.
3. Paste the script into a new indicator.
4. Save the script.
5. Click **Add to chart**.
6. Adjust inputs to match your timeframe and style.

## Best Use Cases

This indicator is useful for:

- spotting RSI regime shifts with less noise
- pairing low-timeframe entries with higher-timeframe RSI context
- confirming overbought/oversold zones
- filtering standard RSI behavior with adaptive historical similarity
- visual trend confirmation using 50-line and band logic

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

## License / Usage

No explicit license block is included in the script shown here.

If you plan to publish or distribute it, add a license section that matches how you want others to use, modify, or credit the work.

## Notes

The script includes a full older version in comments for reference and comparison. If you want a cleaner production version, remove the legacy commented block before publishing.
