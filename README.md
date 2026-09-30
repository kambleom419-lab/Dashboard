# Campus Analytics — Power BI Dashboards

Two self-contained Power BI projects built on synthetic engineering datasets.
Each workbook follows the **descriptive → diagnostic** pattern: first establish
*what happened*, then explain *why it happened*.

| # | Project | Domain | Records | Report file |
|---|---------|--------|---------|-------------|
| 02 | College Energy Management | Campus electricity | 24,112 | `Descriptive dashboard.pbix` |
| 03 | Chiller Plant Performance | HVAC / thermal | 21,930 | `Descriptive Dashboard2.pbix` |

**Tools:** Power BI Desktop · DAX · Power Query · Star-schema modelling

---

## Assignment 02 — College Energy Management

**Problem.** The campus is metered per building, but facilities management has no
consolidated view of where and when electricity is consumed, how much is offset by
rooftop solar, or which factors — weather, occupancy, academic calendar, day type —
actually drive the variation in demand and cost.

**Dataset.** 24,112 records — 11 buildings × 2 shifts/day × ~1,096 days (Aug 2022 – Jul 2025).
Columns include `Total_Energy_kWh`, `HVAC_Load_kWh`, `Lighting_Load_kWh`,
`Equipment_Load_kWh`, `Solar_Generation_kWh`, `Power_Factor`, `Electricity_Cost_INR`,
`CO2_Emissions_kg`, `Grid_Outage_Flag`.

**Model.** `Building_Master[Building]` (1) → `Energy_Log_Data[Building]` (many),
plus a `DimDate` calendar table related on `Date`.

### Key measures

| Measure | DAX |
|---|---|
| Total Energy (kWh) | `SUM(Energy_Log_Data[Total_Energy_kWh])` |
| Total Cost (INR) | `SUM(Energy_Log_Data[Electricity_Cost_INR])` |
| Total CO2 (kg) | `SUM(Energy_Log_Data[CO2_Emissions_kg])` |
| Total Solar (kWh) | `SUM(Energy_Log_Data[Solar_Generation_kWh])` |
| Solar Offset % | `DIVIDE([Total Solar (kWh)], [Total Energy (kWh)], 0)` |
| Avg Power Factor | `AVERAGE(Energy_Log_Data[Power_Factor])` |

### Headline numbers

| KPI | Value |
|---|---|
| Total Energy | 1.09 M kWh |
| Total Cost | ₹ 9.43 M |
| Total Solar | 128.16 K kWh |
| Solar Offset | 11.75 % |
| Total CO₂ | 800.2 K kg |

### Dashboards

**Descriptive** — KPI card row; monthly energy trend; donut of `Day_Type`
(Weekday 76.3% · Saturday 11.83% · Sunday 11.86%); energy by `Building_Type` and by
`Academic_Period`; building scorecard with conditional-format background scales on
cost and CO₂. Slicers: Date range, Building_Type, Academic_Period, Day_Type, Shift.

**Diagnostic** — scatter charts of *ambient temperature vs HVAC load*,
*occupancy vs total energy*, and *equipment load vs power factor* (with a 0.90
penalty-threshold reference line); Key Influencers on `Is_High_Consumption`;
Decomposition Tree on Total Energy; outage / diesel-backup table.

### Insights

- **Occupancy is the strongest driver** of consumption (r ≈ 0.83).
- **Ambient temperature drives HVAC load** (r ≈ 0.52) — cooling degree drives AC energy.
- In-Session periods average **≈ 50 kWh/shift** vs **≈ 32 kWh/shift** in vacations.
- **Residential** buildings (hostels) consume the most (~60 kWh/shift);
  **Administrative** the least (~28 kWh/shift).
- **Power factor dips** as equipment/motor load rises — a power-quality penalty risk.
- Solar generation is zero at night and on buildings without rooftop panels,
  capping the campus **solar offset at 11.75%** — a clear case for expanding capacity.

---

## Assignment 03 — Chiller Plant Performance

**Problem.** The campus central plant runs five chillers (2 Centrifugal, 2 Screw,
1 Absorption). Temperature, flow, pressure and vibration are logged, but there is no
single view of plant efficiency (COP) and no way to explain *why* a chiller degrades.

**Dataset.** 21,930 readings — 5 chillers × 6 readings/day (every 4 hours),
Aug 2023 – Jul 2025. Columns include `Cooling_Load_TR`, `Compressor_Power_kW`, `COP`,
`Fouling_Factor`, `Condenser_Approach_C`, `Vibration_mm_s`, `Cumulative_Run_Hours`,
`Risk_Score`, `Chiller_Status`, `Fault_Code`.

**Model.** `Chiller_Master[Chiller_ID]` (1) → `Chiller_Log_Data[Chiller_ID]` (many),
plus a `DimDate` calendar and a derived `Hour_of_Day` column.

### Key measures

| Measure | DAX |
|---|---|
| Total Cooling Load (TR) | `SUM(Chiller_Log_Data[Cooling_Load_TR])` |
| Total Compressor Energy (kWh) | `SUMX(Chiller_Log_Data, Chiller_Log_Data[Compressor_Power_kW] * 4)` |
| Total Energy Cost (INR) | `SUM(Chiller_Log_Data[Energy_Cost_INR])` |
| Avg COP | `AVERAGE(Chiller_Log_Data[COP])` |
| Avg Fouling Factor | `AVERAGE(Chiller_Log_Data[Fouling_Factor])` |
| Avg Vibration | `AVERAGE(Chiller_Log_Data[Vibration_mm_s])` |
| Mode Chiller Status | `CALCULATE(SELECTEDVALUE(...), TOPN(1, VALUES(...), [Record Count], DESC))` |

### Dashboards

**Descriptive** — KPI cards; monthly COP trend by `Chiller_Type`; load profile by
hour of day; status donut (Healthy / Warning / Critical); chiller scorecard with
red-amber-green conditional formatting on COP and fouling.

**Diagnostic** — every visual **segmented by `Chiller_Type`**; scatter of
*fouling vs COP*, *wet-bulb vs COP*, *run-hours vs vibration* (with trend lines);
Key Influencers on `Is_Critical`; Decomposition Tree on `Risk_Score`;
Pareto of `Fault_Code` counts.

### Insights

- **Within** a chiller type, COP falls as fouling rises (**r ≈ −0.40**) and as
  ambient wet-bulb rises (**r ≈ −0.38**).
- Across all types the COP–fouling correlation is **near zero** — Absorption
  (COP ≈ 0.7) and Centrifugal (COP ≈ 6.0) operate on different scales. The analysis
  **must** be segmented by chiller type; this is a deliberate Simpson's-paradox case.
- **Vibration tracks cumulative run-hours almost perfectly (r ≈ 0.95)** → bearing wear
  is the dominant mechanical risk, consistent across all chillers.
- **Condenser approach rises with fouling** (r ≈ 0.21) → usable as a cleaning-schedule trigger.
- Status mix: **Healthy 45.4% · Warning 49.2% · Critical 5.4%** — nearly half the fleet
  is already in Warning.

---

## How to open

1. Install **Power BI Desktop** (free, Windows).
2. Open either `.pbix` file. The data is embedded in the model, so the report
   renders immediately without the original Excel workbook.
3. Use the slicers to filter by date, building/chiller type, period and shift.

---

## Author

*Your Name* — Data Analytics Lab, Mechanical Engineering
