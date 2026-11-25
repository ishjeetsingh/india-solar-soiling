# Can satellite aerosol data predict solar-panel soiling in India?

**Ishjeet Singh Kadewale** · CHIREC International School, Hyderabad · started November 2026 · *work in progress*

Dust on solar panels ("soiling") cuts output until rain or cleaning removes it. This project tests whether satellite data, including **ISRO's INSAT-3DR aerosol product**, can predict how fast panels get dirty across India. It then turns the best predictor into a **district-level map of the cost-optimal cleaning interval**.

It extends my earlier project **SPADE** (an electrostatic panel-cleaning device) from one device to a national scale.

**[View the interactive map →](https://ishjeetsingh.github.io/india-solar-soiling/figures/cleaning_map.html)**

---

## Preliminary findings (2024 data)

| Predictor of measured soiling rate | Leave-one-out error (%/day) | Beats "guess the average" (0.073)? |
|---|---|---|
| ISRO INSAT-3DR aerosol optical depth (AOD) | 0.094 | No |
| NASA MERRA-2 AOD (same days) | 0.082 | No |
| NASA MERRA-2 **dust deposition** | **0.066** | Yes, modestly |

1. **Total AOD from either agency is a poor proxy for soiling.** Haze from fine smoke and pollution (e.g. Lucknow) stays in the air; coarse dust is what lands on panels.
2. **Modelled dust deposition does better.** This is consistent with field studies showing coarse particles dominate deposited mass.
3. **ISRO and NASA AOD differ substantially at these sites.** INSAT-3DR reads higher (dry-day means 0.57–1.08 vs 0.42–0.77) and agrees only loosely day to day (r = 0.34). At Chennai the difference is not caused by sea pixels. Neither dataset is ground truth here; ground stations (AERONET) would be needed to say which is closer.
4. **Map:** the optimal interval ranges from about 6 days (Jaisalmer, Thar desert) to about 12 weeks (wet north-east), with a median of 15 days.
5. **Sensitivity:** the exact intervals depend on cleaning cost and electricity tariff (median 10–23 days), but the **ranking of districts is stable** (Spearman 0.82–0.91). Rain sets most of the geographic pattern; removing the dust term still gives 0.88 rank agreement.

### Limitations, stated plainly
- Only **7 published measurements at 6 locations** (Gandhinagar has two studies), so the model comparison is exploratory and not statistically conclusive.
- The model form (a + b × predictor) was chosen after seeing that a no-intercept model fit poorly.
- Published soiling rates come from different years, methods and seasons than the 2024 satellite data.
- Cleaning cost (₹2 per kWp per clean) and tariff (₹2.6/kWh) are placeholder assumptions; see the sensitivity analysis.
- Snow, panel tilt and local dust sources are not modelled (e.g. results for Ladakh should be treated with caution).

---

## Repository structure

```
data/
  soiling_sites.csv                       published soiling measurements (ground truth) + sources
  site_daily_2024.csv                     NASA POWER rain, irradiance, AOD at each site
  merra2_site_monthly_dust_2024.csv       MERRA-2 dust deposition at each site
  insat_site_daily_aod_2024.csv           ISRO INSAT-3DR AOD at each site
  district_daily_2024.csv                 NASA POWER rain + irradiance at 820 district centres
  merra2_district_monthly_dust_2024.csv   MERRA-2 dust deposition per district
  district_cleaning_intervals.csv         results: optimal interval per district
  sensitivity.csv                         sensitivity analysis results
figures/
  cleaning_map.html                       interactive map
notebooks/
  01_site_data.ipynb                      NASA POWER data for the soiling sites
  02_merra2_dust_model.ipynb              MERRA-2 dust deposition + soiling model
  03_insat_aod_vs_nasa.ipynb              ISRO INSAT-3DR download + ISRO vs NASA comparison
  04_district_cleaning_map.ipynb          optimal cleaning interval, map, sensitivity
```

All notebooks run in Google Colab. Notebooks 02 and 03 need free accounts (NASA Earthdata, ISRO MOSDAC); credentials are entered at runtime and never stored.

## Method in brief
1. **Ground truth:** published soiling rates (% energy loss per day) at Indian sites.
2. **Predictors:** INSAT-3DR AOD (half-hourly, 0.1°), MERRA-2 AOD (via NASA POWER), MERRA-2 dust dry deposition + settling (monthly).
3. **Validation:** linear model, leave-one-out cross-validation, compared against a leave-one-out "mean" baseline; ISRO and NASA AOD compared on identical days.
4. **Cleaning optimisation:** daily simulation per district (dust builds up, rain ≥ 5 mm resets, manual cleaning every T days); choose T minimising cleaning cost + lost-energy value.
5. **Sensitivity:** cleaning cost, tariff, rain threshold, and removing the dust term.

## Data sources and acknowledgements
- **INSAT-3DR Imager AOD (`3RIMG_L2G_AOD`):** MOSDAC, Space Applications Centre (SAC), ISRO (https://mosdac.gov.in). *We acknowledge MOSDAC for providing the data.*
- **MERRA-2 `M2TMNXADG`:** NASA Global Modeling and Assimilation Office, distributed by GES DISC.
- **Rainfall, irradiance and AOD:** NASA POWER Project (https://power.larc.nasa.gov).
- **District boundaries:** DataMeet community / datta07 INDIAN-SHAPEFILES (CC BY 2.5 India).
- **Soiling measurements:** see the `source` column of `data/soiling_sites.csv`.

Code: MIT licence.
