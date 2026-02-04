# dailyweight
PWA for daily body comp &amp; nutrition tracking with self-calibrating TDEE projections. Logs weight, measurements, steps, cardio &amp; macros — estimates BMR via Katch-McArdle, derives TDEE from step-banded multipliers, then calibrates against actual weight change to project 5 &amp; 10-week outcomes. Fully local, no backend.
PWA for daily body comp & nutrition tracking with self-calibrating TDEE projections. Logs weight, measurements, steps, cardio & macros — estimates BMR via Katch-McArdle, derives TDEE from step-banded multipliers, then calibrates against actual weight change to project 5 & 10-week outcomes. Fully local, no backend.

## How it works

Every morning, log your measurements and yesterday's activity/nutrition. The app builds a picture of your energy balance and projects where you'll be in 5 and 10 weeks if current trends hold.

### Daily inputs

| Field | Unit | Notes |
|---|---|---|
| Weight | kg | Morning, fasted |
| Neck | cm | For Navy BF% |
| Waist | cm | For Navy BF% |
| Hip | cm | Female only, for Navy BF% |
| Steps | count | Yesterday |
| Cardio burned | kcal | Yesterday |
| Calories | kcal | Yesterday |
| Carbs | g | Yesterday |
| Protein | g | Yesterday |
| Fat | g | Yesterday |

### Formula chain

**1. Body Fat % — US Navy Method (Hodgdon & Beckett 1984)**

Male:
```
BF% = 495 / (1.0324 − 0.19077 × log₁₀(waist − neck) + 0.15456 × log₁₀(height)) − 450
```

Female:
```
BF% = 495 / (1.29579 − 0.35004 × log₁₀(waist + hip − neck) + 0.22100 × log₁₀(height)) − 450
```

**2. BMR — Katch-McArdle**

```
LBM = weight × (1 − BF% / 100)
BMR = 370 + (21.6 × LBM)
```

Uses body fat % rather than age/sex proxies — more accurate when you have composition data.

**3. TDEE — Step-banded multiplier + discounted cardio**

```
TDEE = (BMR × step_multiplier) + (cardio_kcal × 0.5)
```

| Daily steps | Multiplier | Band |
|---|---|---|
| < 5,000 | 1.200 | Sedentary |
| 5,000 – 7,499 | 1.375 | Light |
| 7,500 – 9,999 | 1.550 | Moderate |
| 10,000 – 12,499 | 1.725 | Active |
| 12,500+ | 1.900 | Very Active |

Multiplier covers NEAT and TEF. Cardio discounted 50% to avoid double-counting with step-based activity.

**4. Self-calibration**

Compares predicted weight change vs actual weight change over the last 14 days:

```
factor = actual_Δweight / predicted_Δweight      (clamped 0.5 – 1.5)
calibrated_TDEE = BMR + (TDEE − BMR) × factor
```

Applies correction to the activity portion of TDEE only. Shows "Uncalibrated" until 14+ days of data, then self-corrects continuously.

**5. Weight projection**

```
weekly_Δkg = (calibrated_TDEE − avg_daily_kcal) × 7 / 7700
projected_weight = current + weekly_Δkg × weeks
```

**6. Dimension projection**

Derives cm/kg ratios from the last 4 weeks only (toilet roll effect — the relationship between weight loss and measurement change shifts as you get leaner):

```
projected_dim = current + (projected_weight_Δ × cm_per_kg_ratio)
```

Rate-clamped to physiological maximums:
- Waist: ±1.5 cm/week
- Neck: ±0.5 cm/week
- Hip: ±1.5 cm/week

Falls back to linear extrapolation (still clamped) if insufficient data for ratio derivation.

## Setup

Single HTML file. No build step, no dependencies to install.

**Host as PWA (recommended):**

1. Push `tracker.html` to a GitHub repo
2. Enable GitHub Pages (or drop it on Netlify/Vercel)
3. Open the URL on your phone
4. "Add to Home Screen"

Opens full-screen like a native app. Works offline.

**Run locally:**

Just open `tracker.html` in a browser.

## Data

All data stored in browser `localStorage`. No server, no accounts, no telemetry.

- **Export**: Settings → Export JSON (backs up entries + settings)
- **Import**: Settings → Import JSON (merges with existing data, same-date entries overwrite)
- **Clear**: Settings → Clear All Data

Back up regularly via export. Browser storage can be evicted under storage pressure.

## Settings

| Setting | Default | Notes |
|---|---|---|
| Sex | Male | Switches Navy BF% formula, shows/hides hip field |
| Height | 171 cm | Used in Navy BF% calculation |
| Goal weight | 75 kg | For distance-to-goal and weeks-to-goal display |

## Stack

- Vanilla HTML/CSS/JS
- [Chart.js](https://www.chartjs.org/) (CDN) for weight trend chart
- [DM Sans](https://fonts.google.com/specimen/DM+Sans) + [JetBrains Mono](https://fonts.google.com/specimen/JetBrains+Mono) (Google Fonts)
