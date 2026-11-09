# Forest Change Detection with Sentinel-2

> Detecting forest disturbance and burn scars from Sentinel-2 imagery using dNBR, starting with the 2024 St. Mary fire in Montana's Bitterroot Mountains.

**Status:** early work in progress. One exploratory notebook so far.

![NBR 2024, NBR 2025, and dNBR maps of the St. Mary area. The burn scar appears as a dark patch in the 2025 NBR and as a strong positive signal in the dNBR panel.](figures/nbr_dnbr.png)

*Left to right: median summer NBR in 2024, median summer NBR in 2025, and the difference (dNBR). The St. Mary burn scar is the compact patch in the upper middle. The valley on the east side also lights up, mostly from agricultural change.*

---

## What this does

The notebook compares two summer windows (May 15 to July 31, 2024 vs. 2025) to find where vegetation was lost, using the fire as a known test case.

1. Search the [Earth Search](https://element84.com/earth-search/) STAC catalog for Sentinel-2 L2A scenes over a hand-drawn area of interest (scene cloud cover under 40%).
2. Load the NIR, SWIR2, and scene classification (SCL) bands at 10 m with `odc-stac`.
3. Mask no-data, saturated, dark/shadow, cloud, cirrus, and snow pixels using SCL.
4. Compute NBR per scene, take the per-pixel median for each window, and subtract (dNBR = NBR 2024 − NBR 2025).
5. Flag pixels where dNBR is above 0.2 (preliminary threshold), polygonize them, and keep polygons of at least 1 acre.
6. As an alternative to a fixed threshold, I also tried a z-score (Welch t-statistic) built from the per-scene NBR values in each window.

The windows cover the growing season and end before wildfire smoke becomes common.

**Outputs:** a z-score GeoTIFF (COG) and a GeoJSON of flagged polygons.

## Results so far

- The burn scar near St. Mary Peak is clearly recovered in the dNBR map.
- A fixed dNBR threshold also flags a lot of agricultural change in the valley, since crop rotation looks like vegetation loss from year to year.
- The z-score version removes much of that valley signal at moderate cutoffs but responds to crop rotation as well, and it is a relative score, not a formal significance test (scenes within a window are not independent).

Thresholds are preliminary and have not yet been validated against an official fire perimeter.

## Known issues

- Valley agriculture is flagged (both the fixed threshold and the z-score).
- No forest mask yet, so non-forest change is not filtered out.
- No explicit reflectance scale/offset correction is applied in the notebook, so NBR values and thresholds should be read as relative.
- The method compares two seasons and is not near real-time.

## Next steps

- Filter to forest only using the NLCD land cover dataset (a custom forest classifier is a possible later step).
- Compare detected polygons with the official fire perimeter (MTBS / NIFC) and report an overlap number.
- Extend the approach to timber harvest detection (clearcuts and thinning) with other spectral indices.
- Move reusable logic from the notebook into the `harvest` package in `src/`.

## Longer-term goals

I am curious about how the forests of North America change over time. Montana holds a large amount of public forest and has frequent harvest and wildfire activity, which makes it a good place to explore this. Over time I would like this project to grow to cover:

1. **Forest harvest tracking:** where is harvest activity happening?
2. **Wildfire risk modeling:** where is fire most likely to ignite and spread, before it starts?
3. **Fire and burn perimeter detection:** can a fire and its perimeter be detected from satellite imagery soon after it starts?

I would also like to track snow reflectance and snowline over time. The current scope is the Bitterroot Mountains in southwestern Montana, and none of the items above beyond the notebook are built yet.

## How to run

This project uses [uv](https://docs.astral.sh/uv/).

```bash
git clone git@github.com:BilyBrown/forest_change_detection.git
cd forest_change_detection
uv sync
```

Then open `notebooks/initial_test.ipynb` in Jupyter or VS Code and run it top to bottom. The area of interest is read from `data/stMary_aoi.shp`.

## Data

- **Imagery:** Sentinel-2 L2A from the public [Earth Search](https://element84.com/earth-search/) STAC catalog (`sentinel-2-c1-l2a` collection). No account or API key is needed.
- **Area of interest:** a hand-drawn polygon around St. Mary Peak and the adjacent Bitterroot Valley.

## Tools

`pystac-client`, `odc-stac`, `xarray`, `geopandas`, `rasterio`, `shapely`, `matplotlib`

## References

- UN-SPIDER: [Normalized Burn Ratio and burn severity](https://un-spider.org/advisory-support/recommended-practices/recommended-practice-burn-severity/in-detail/normalized-burn-ratio)
