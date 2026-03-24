# RTPP Code Overview

This document explains what the RTPP codebase does at a high level and how each section fits together.

## What This Code Does

The code estimates plant real-time power potential (RTPP) and an HSL output using:

1. Physics-based calculation from irradiance, temperature, losses, and inverter availability.
2. Online linear regression correction against measured plant active power.
3. Output publishing for control and SCADA diagnostics.

The correction layer is important because PV sites have many practical unknowns and changing conditions (soiling, sensor bias, clipping behavior, seasonal effects, and unmodeled losses). The regression continuously adapts the physics estimate to measured plant behavior.

In short:

- `FB_RTPP_CALC` computes a model-based RTPP.
- `RTPP_REG` learns correction coefficients (`M`, `C`) online.
- `FB_RTPP` orchestrates everything and produces final outputs.
- `PG_RTPP` maps site tags to generic library inputs.

## Main Files and Roles

### `FB_RTPP.xml`

Top-level function block that coordinates the workflow.

Sections:

- **STEP 1: Available inverter handling**
  - Bounds available inverter count to valid range.
- **STEP 2: Sensor averaging**
  - Uses `SensorAverage` for front POA, rear POA, and BOM temperature.
- **STEP 3: Physics RTPP**
  - Calls `rtppCalc` to compute model RTPP (`RTPP_TOTAL`).
- **STEP 4: Regression update**
  - Periodically updates regression from per-inverter normalized values.
- **STEP 5: Corrected RTPP**
  - Applies correction: `RTPP_TOTAL * RTPP_Reg_M + TotalAvailable * RTPP_Reg_C`.
  - This correction compensates for site-specific unknowns that are difficult to model directly in a pure physics equation.
- **STEP 6: HSL logic**
  - Uses `ActivePowerMW` (PV generation meter) and setpoint state.
- **STEP 7: Output mapping**
  - Publishes primary outputs and diagnostics.

Outputs include:

- `RTPP_MW`
- `HSL_MW`
- Regression diagnostics (`Reg_M`, `Reg_C`, `Reg_N`)
- Sensor averages and online counts

### `FB_RTPP_CALC.xml`

Physics model block.

Core calculations:

1. Bifacial gain from rear/front irradiance and bifaciality.
2. Effective irradiance.
3. Temperature factor.
4. Degradation factor from commissioning date to current date.
5. DC output with losses.
6. AC output with inverter efficiency and availability ratio.
7. AC loss application and POI clamp.

Includes robustness clamps for configuration values (efficiency, losses, bifaciality, degradation bounds).

### `RTPP_REG.xml`

Online linear regression learner.

Behavior:

- Keeps recent `(rtpp, active_p)` samples in a **1000-entry ring buffer** (O(1) write, overwrites oldest when full).
- Recomputes a **recency-weighted** least-squares fit on every call: newest sample has weight 1.0, oldest has weight `RTPP_Alpha^(N-1)`. Higher `RTPP_Alpha` = longer memory; lower = faster adaptation.
- Updates `M`, `C` by reference (`VAR_IN_OUT`) so retained globals persist across reboot.
- Requires at least 50 samples before fitting begins.
- Exposes sample count `N` for diagnostics.
- Learns site-specific bias and gain drift over time so the output remains aligned with real plant response.

### `FB_SENSOR_AVG.xml`

Reusable averaging helper.

Purpose:

- Filters sensors by quality and configured min/max bounds.
- Supports per-sensor operator ignore from `UDT_SENSOR.Ignore`.
  - This flag is intended to be mapped from SCADA/HMI so operators can
    temporarily exclude bad or maintenance-affected sensors without code changes.
- Returns average and count of valid sensors.

### `UDT_SENSOR.xml`

Shared sensor data type (`Quality`, `Value`, `Ignore`).

- `Ignore` is a runtime override point that can be driven from SCADA/HMI.
- When `Ignore = TRUE`, that sensor is excluded from averaging logic.

### `GVL_RTPP.xml`

Global variables used by program and SCADA layers.

Contains:

- Output MVs (`RTPP_OUTPUT`, `HSL_OUTPUT`)
- Diagnostic globals (`FRONT_POA_AVG`, `REAR_POA_AVG`, `BOM_AVG`)
- **Retained regression coefficients** (survive controller restart):
  - `RTPP_Reg_M` — regression slope (init 1.0)
  - `RTPP_Reg_C` — regression intercept (init 0.0)
- **Retained regression conditioning thresholds** (adjustable from SCADA at runtime):
  - `RTPP_MinIrradiance` — minimum front POA (W/m²) to allow a regression sample (default 100.0)
  - `RTPP_ClippingThreshold` — skip regression when `ActivePowerMW ≥ POILimit × threshold` (default 0.98)
  - `RTPP_Alpha` — exponential decay factor for recency weighting (default 0.99)

### `PG_RTPP.xml`

Site adapter program.

This is the file you customize per site:

1. Map site tags into generic sensor arrays.
2. Map per-sensor ignore points from SCADA/HMI into each sensor `Ignore` field.
3. Set plant constants (inverter count, capacities, POI, dates).
4. Call `FB_RTPP` with mapped tags and constants.
5. Publish outputs to global MVs.

## Data Flow Summary

1. Site tags -> sensor arrays (`PG_RTPP`).
2. Sensor arrays -> filtered averages (`FB_SENSOR_AVG`).
3. Averages + plant params -> model RTPP (`FB_RTPP_CALC`).
4. Model RTPP + metered active power -> regression update (`RTPP_REG`).
5. Corrected RTPP + HSL logic -> final outputs (`FB_RTPP`).
6. Final outputs -> global points for external use (`PG_RTPP` / `GVL_RTPP`).

## Design Intent

- Keep algorithm generic and reusable.
- Keep site-specific naming isolated to one wrapper (`PG_RTPP`).
- Preserve learned regression coefficients through restart using retained globals.
- Provide diagnostics for commissioning and SCADA trending.
