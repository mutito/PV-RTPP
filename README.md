# RTPP Library

Reusable RTAC RTPP library with site adapter pattern.

## Current Module Names

- `RTPP/RTPP_GLOBALS.xml`
- `RTPP/UDT_SENSOR.xml`
- `RTPP/FB_SENSOR_AVG.xml`
- `RTPP/FB_RTPP_CALC.xml`
- `RTPP/RTPP_REG.xml`
- `RTPP/FB_RTPP.xml`
- `RTPP/RTPP_PROGRAM.xml`

## Key Metering Note

`ActivePowerMW` is the **PV generation meter** input (MW) and is used for:

- Regression update target (`RTPP_REG` via `active_p` input)
- HSL calculation path in `FB_RTPP`

`PV_HV_MW` input has been removed from the current interface.

## Docs

- Implementation template: `Docs/RTPP_IMPLEMENTATION_TEMPLATE.md`
- Code overview: `Docs/RTPP_CODE_OVERVIEW.md`
- Commercial license template: `Docs/LICENSE-COMMERCIAL.md`

## Integration Rule

Keep all site-specific tag mappings in `RTPP/RTPP_PROGRAM.xml`. Keep core FB logic generic.
