# RTPP Code Overview

This document explains what the RTPP codebase does at a high level and how each section fits together.

## What This Code Does

The code estimates plant real-time power potential (RTPP) and an HSL output using:

1. Physics-based calculation from irradiance, temperature, losses, and inverter availability.
2. Online linear regression correction against measured plant active power.
3. Output publishing for control and SCADA diagnostics.

In short:

- `rtppCalc` computes a model-based RTPP.
- `RTPP_REG` learns correction coefficients (`M`, `C`) online.
- `rtppMain` orchestrates everything and produces final outputs.
- `RTPP_PROGRAM` maps site tags to generic library inputs.

## Main Files and Roles

### `rtppMain.xml`

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
- **STEP 6: HSL logic**
  - Sets HSL based on curtailed vs uncurtailed behavior.
- **STEP 7: Output mapping**
  - Publishes primary outputs and diagnostics.

Outputs include:

- `RTPP_MW`
- `HSL_MW`
- Regression diagnostics (`Reg_M`, `Reg_C`, `Reg_N`)
- Sensor averages and online counts

### `rtppCalc.xml`

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

- Keeps FIFO buffers of recent `(rtpp, active_p)` samples.
- Recomputes linear fit when enough samples exist.
- Updates `M`, `C` by reference (`VAR_IN_OUT`) so retained globals persist across reboot.
- Exposes sample count `N` for diagnostics.

### `SensorAverage.xml`

Reusable averaging helper.

Purpose:

- Filters sensors by quality and configured min/max bounds.
- Returns average and count of valid sensors.

### `sensor.xml`

Shared sensor data type (`Quality`, `Value`, optional ignore flag).

### `RTPP_GLOBALS.xml`

Global variables used by program and SCADA layers.

Contains:

- Output MVs (`RTPP_OUTPUT`, `HSL_OUTPUT`)
- Diagnostic globals (`FRONT_POA_AVG`, `REAR_POA_AVG`, `BOM_AVG`)
- **Retained regression coefficients**:
  - `RTPP_Reg_M`
  - `RTPP_Reg_C`

### `RTPP_PROGRAM.xml`

Site adapter program.

This is the file you customize per site:

1. Map site tags into generic sensor arrays.
2. Set plant constants (inverter count, capacities, POI, dates).
3. Call `rtppMain` with mapped tags and constants.
4. Publish outputs to global MVs.

## Data Flow Summary

1. Site tags -> sensor arrays (`RTPP_PROGRAM`).
2. Sensor arrays -> filtered averages (`SensorAverage`).
3. Averages + plant params -> model RTPP (`rtppCalc`).
4. Model RTPP + metered active power -> regression update (`RTPP_REG`).
5. Corrected RTPP + HSL logic -> final outputs (`rtppMain`).
6. Final outputs -> global points for external use (`RTPP_PROGRAM` / `RTPP_GLOBALS`).

## Design Intent

- Keep algorithm generic and reusable.
- Keep site-specific naming isolated to one wrapper (`RTPP_PROGRAM`).
- Preserve learned regression coefficients through restart using retained globals.
- Provide diagnostics for commissioning and SCADA trending.
