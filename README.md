# RTPP Library

Reusable RTAC RTPP library with site adapter pattern.

## Current Module Names

- [RTPP_CALC/RTPP_GLOBALS.xml](RTPP_CALC/RTPP_GLOBALS.xml)
- [RTPP_CALC/UDT_SENSOR.xml](RTPP_CALC/UDT_SENSOR.xml)
- [RTPP_CALC/FB_SENSOR_AVG.xml](RTPP_CALC/FB_SENSOR_AVG.xml)
- [RTPP_CALC/FB_RTPP_CALC.xml](RTPP_CALC/FB_RTPP_CALC.xml)
- [RTPP_CALC/RTPP_REG.xml](RTPP_CALC/RTPP_REG.xml)
- [RTPP_CALC/FB_RTPP.xml](RTPP_CALC/FB_RTPP.xml)
- [RTPP_CALC/RTPP_PROGRAM.xml](RTPP_CALC/RTPP_PROGRAM.xml)

## Key Metering Note

`ActivePowerMW` is the **PV generation meter** input (MW) and is used for:

- Regression update target (`RTPP_REG` via `active_p` input)
- HSL calculation path in `FB_RTPP`

`PV_HV_MW` input has been removed from the current interface.

## Docs

- Implementation template: [Docs/RTPP_IMPLEMENTATION_TEMPLATE.md](Docs/RTPP_IMPLEMENTATION_TEMPLATE.md)
- Code overview: [Docs/RTPP_CODE_OVERVIEW.md](Docs/RTPP_CODE_OVERVIEW.md)
- Contribution guide: [CONTRIBUTING.md](CONTRIBUTING.md)
- License: [LICENSE](LICENSE)

## Integration Rule

Keep all site-specific tag mappings in [RTPP_CALC/RTPP_PROGRAM.xml](RTPP_CALC/RTPP_PROGRAM.xml). Keep core FB logic generic.

## Contributing

Contributions are welcome. Please submit all code changes through pull requests.

Before opening a PR:

- Keep changes focused and scoped to one purpose.
- Update docs if behavior, interfaces, or configuration changes.
- Include enough context in the PR description for review and validation.

See [CONTRIBUTING.md](CONTRIBUTING.md) for the full process.
