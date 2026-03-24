# RTPP Implementation Template

Use this checklist and template to deploy the RTPP library to a new site.

## 1. Initial Setup

Complete these steps first:

- Download the RTPP code from GitHub.
- In RTAC User Logic, create a folder named RTTP .

## 2. Required Modules

Import these modules into the RTAC project:

- GVL_RTPP.xml
- UDT_SENSOR.xml
- FB_SENSOR_AVG.xml
- FB_RTPP_CALC.xml
- RTPP_REG.xml
- FB_RTPP.xml
- PG_RTPP.xml

## 3. One-Time Library Rules

- Do not edit FB_RTPP.xml, FB_RTPP_CALC.xml, RTPP_REG.xml for site tag naming.
- Keep all site-specific mappings in PG_RTPP.xml only.
- Retained regression coefficients in GVL_RTPP.xml survive controller restart:
  - RTPP_Reg_M := 1.0
  - RTPP_Reg_C := 0.0
- Regression conditioning thresholds in GVL_RTPP.xml are adjustable from SCADA at runtime:
  - RTPP_MinIrradiance (default 100.0 W/m²)
  - RTPP_ClippingThreshold (default 0.98)
  - RTPP_Alpha (default 0.99)

## 4. Site Configuration Checklist

Important: The implementation body in RTPP_PROGRAM.xml is provided as an import-safe commented template.

- [ ] Uncomment the implementation block in PG_RTPP.xml.
- [ ] Replace all example tags with your site-specific tags.
- [ ] Keep all site-specific changes in PG_RTPP.xml only.

Update the following constants in PG_RTPP.xml:

- [ ] PLANT_TOTAL_INVERTERS
- [ ] PLANT_DC_NOMINAL_MW
- [ ] PLANT_COMM_YEAR
- [ ] PLANT_COMM_MONTH
- [ ] PLANT_POI_LIMIT_MW
- [ ] FRONT_SENSOR_COUNT
- [ ] REAR_SENSOR_COUNT
- [ ] BOM_SENSOR_COUNT

**Electrical parameters (tune to match site equipment and conditions):**

- [ ] BIFACIALITY — module bifaciality coefficient (e.g. 0.70)
- [ ] TEMP_COEF — power temp coefficient (%/degC, typically negative, e.g. -0.0034)
- [ ] DEGRADATION — annual degradation fraction (e.g. 0.005)
- [ ] DC_LOSSES — DC losses fraction (e.g. 0.005)
- [ ] AC_LOSSES — AC losses fraction (e.g. 0.015)
- [ ] INV_EFFICIENCY — inverter efficiency fraction (e.g. 0.98)

Update these tag mappings in PG_RTPP.xml:

- [ ] Front POA Quality and Value tags
- [ ] Rear POA Quality and Value tags
- [ ] BOM temperature Quality and Value tags
- [ ] AvailableInverters tag
- [ ] ActivePowerMW tag (PV generation meter, in MW)
- [ ] PSetpointMW tag

## 5. Deployment Template (Site Wrapper)

Copy and adapt this structure in PG_RTPP.xml.

```st
PROGRAM PG_RTPP
VAR
  RTPP : FB_RTPP;
  FrontPOA : ARRAY[1..30] OF UDT_SENSOR;
  RearPOA  : ARRAY[1..30] OF UDT_SENSOR;
  BOMTemp  : ARRAY[1..30] OF UDT_SENSOR;
END_VAR

VAR CONSTANT
    PLANT_TOTAL_INVERTERS : REAL := 0.0;   // TODO
    PLANT_DC_NOMINAL_MW   : REAL := 0.0;   // TODO
    PLANT_COMM_YEAR       : DINT := 2024;  // TODO
    PLANT_COMM_MONTH      : DINT := 1;     // TODO
    PLANT_POI_LIMIT_MW    : REAL := 0.0;   // TODO

    FRONT_SENSOR_COUNT    : INT := 0;      // TODO
    REAR_SENSOR_COUNT     : INT := 0;      // TODO
    BOM_SENSOR_COUNT      : INT := 0;      // TODO

    ELEC_BIFACIALITY      : REAL := 0.70;   // TODO: tune to module spec
    ELEC_TEMP_COEF        : REAL := -0.0034; // TODO: tune to module spec
    ELEC_DEGRADATION      : REAL := 0.005;  // TODO: tune to site conditions
    ELEC_DC_LOSSES        : REAL := 0.005;  // TODO: tune to site conditions
    ELEC_AC_LOSSES        : REAL := 0.015;  // TODO: tune to site conditions
    ELEC_INV_EFFICIENCY   : REAL := 0.98;   // TODO: tune to inverter spec
END_VAR

// SECTION 1: Front POA mapping
// Replace fPOA[1..4] with your site's front irradiance tags
FrontPOA[1].Quality := fPOA[1].q.validity;
FrontPOA[1].Value   := fPOA[1].instMag;
FrontPOA[2].Quality := fPOA[2].q.validity;
FrontPOA[2].Value   := fPOA[2].instMag;
// ... add fPOA[3..4] similarly

// SECTION 2: Rear POA mapping
// Replace rPOA[1..4] with your site's rear irradiance tags
RearPOA[1].Quality  := rPOA[1].q.validity;
RearPOA[1].Value    := rPOA[1].instMag;
RearPOA[2].Quality  := rPOA[2].q.validity;
RearPOA[2].Value    := rPOA[2].instMag;
// ... add rPOA[3..4] similarly

// SECTION 3: BOM mapping
// Replace BOM[1..12] with your site's temperature tags
BOMTemp[1].Quality  := BOM[1].q.validity;
BOMTemp[1].Value    := BOM[1].instMag;
BOMTemp[2].Quality  := BOM[2].q.validity;
BOMTemp[2].Value    := BOM[2].instMag;
// ... add BOM[3..12] similarly

// SECTION 4: Call RTPP block
RTPP(
    TotalInverters        := PLANT_TOTAL_INVERTERS,
    AvailableInverters    := inv_running.instMag,

    FrontPOASensors       := FrontPOA,
    NumberOfFPOASensors   := FRONT_SENSOR_COUNT,
    RearPOASensors        := RearPOA,
    NumberOfRPOASensors   := REAR_SENSOR_COUNT,
    BOMTempSensors        := BOMTemp,
    NumberOfBOMTempSensors:= BOM_SENSOR_COUNT,

    DCPowerNominalMW      := PLANT_DC_NOMINAL_MW,
    commYear              := PLANT_COMM_YEAR,
    commMonth             := PLANT_COMM_MONTH,

    BifacialityFactor     := ELEC_BIFACIALITY,
    TempCoef              := ELEC_TEMP_COEF,
    DegradationRate       := ELEC_DEGRADATION,
    DCLossesPercent       := ELEC_DC_LOSSES,
    ACLossesPercent       := ELEC_AC_LOSSES,
    InvEfficiency         := ELEC_INV_EFFICIENCY,
    PlantPOILimitMW       := PLANT_POI_LIMIT_MW,

    ActivePowerMW         := pv_gen.instMag * 0.001,          // TODO: PV-only generation meter in MW (not POI if BESS present)
    PSetpointMW           := setpoint.instMag        // TODO
);

// SECTION 5: Publish outputs
RTPP_OUTPUT.instMag := RTPP.RTPP_MW;
RTPP_OUTPUT.t       := System_Time_Control_POU.System_Time;
RTPP_OUTPUT.q       := GOOD_QUALITY;

HSL_OUTPUT.instMag  := RTPP.HSL_MW;
HSL_OUTPUT.t        := System_Time_Control_POU.System_Time;
HSL_OUTPUT.q        := GOOD_QUALITY;

FRONT_POA_AVG       := RTPP.FrontPOA_Avg;
REAR_POA_AVG        := RTPP.RearPOA_Avg;
BOM_AVG             := RTPP.BOMTemp_Avg;
```

## 6. Commissioning Checks

- [ ] Verify ActivePowerMW is PV generation meter in MW (not kW).
- [ ] Verify AvailableInverters is in range 0..TotalInverters.
- [ ] Verify sensor quality bits are valid when expected.
- [ ] Verify Reg_N increments during normal daytime uncurtailed operation.
- [ ] Verify RTPP_Reg_M and RTPP_Reg_C remain after controller restart.
- [ ] Trend RTPP_MW, HSL_MW, Reg_M, Reg_C, Reg_N for at least one day.

## 7. Troubleshooting Quick Notes

- RTPP stuck near zero:
  - Check sensor mapping, quality flags, and irradiance scaling.
- Regression not learning:
  - Check PSetpointMW condition, daytime threshold, and ActivePowerMW scaling.
- Sudden output jumps:
  - Check tag unit mismatches and sensor quality toggling.
