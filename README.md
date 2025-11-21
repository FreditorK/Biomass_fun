# Biomass_fun
Here is the complete project specification, consolidated from all the screenshots into a single, structured, and copyable Markdown file. I have integrated the specific formulas and tables into their respective phases to ensure no information is lost.

-----

# Project Specification: Ecosystem Services Valuation using Sentinel-2 Data

## 1\. Materials & Environment

### Recommended Tools and Software

  * **Python:** Version 3.9 or higher
  * **Interface:** Jupyter Notebook or JupyterLab
  * **Hardware:** 4GB RAM minimum, 2GB disk space

### Installation

To set up the local environment, follow these steps:

```bash
# Create virtual environment
conda create -n biomass python=3.9
conda activate biomass

# Install Libraries
pip install numpy pandas matplotlib rasterio geopandas jupyter

# Verify installation
python -c "import rasterio; print('OK')"
```

-----

## 2\. Data Description

You will download a GeoTIFF file containing **Sentinel-2 Level-2A surface reflectance data** for your assigned study area. The image has been atmospherically corrected and cloud-masked.

> **Important:** Pixel values are scaled between **0-10000** (divide by 10000 to get reflectance). Each pixel represents $100 m^2$ ($10m \times 10m$) at full resolution.

### 10.1. Sentinel-2 Band Information

| Band | Wavelength (nm) | Resolution | Purpose |
| :--- | :--- | :--- | :--- |
| **B2 - Blue** | 490 | 10m | Vegetation discrimination |
| **B3 - Green** | 560 | 10m | Vegetation vigor |
| **B4 - Red** | 665 | 10m | Chlorophyll absorption |
| **B8 - NIR** | 842 | 10m | Vegetation biomass |
| **B11 - SWIR1** | 1610 | 20m | Moisture content |
| **B12 - SWIR2** | 2190 | 20m | Vegetation moisture |

-----

## 3\. Implementation Roadmap

Follow this workflow systematically. Each phase builds on the previous one.

### PHASE 0: Connect CDSE API & Download Sentinel-2 products

  * **Task 0.1:** Connection to CDSE API with the given credentials.
  * **Task 0.2:** Calculate the polygon (buffer) of 25 km around the central point (AOI).
  * **Task 1.3:** Search for Sentinel-2 products and download the one for the date provided.

### PHASE 1: Data Loading & Preprocessing

  * **Task 1.1:** Load the GeoTIFF/jp2 files using appropriate Python libraries (hint: `rasterio`, `gdal`).
  * **Task 1.2:** Extract required spectral bands and store as NumPy arrays.
  * **Task 1.3:** Apply scaling factor to convert to reflectance values (0-1 range).
  * **Task 1.4:** Handle no-data values and cloud-masked pixels.
  * **Task 1.5:** Visualize true-color composite to verify data quality.

> **Key Considerations:** Ensure bands are aligned if mixing 10m and 20m resolutions. Create masks for invalid pixels early to avoid propagation errors.

### PHASE 2: Vegetation Index Calculation

  * **Task 2.1:** Implement **NDVI** calculation with division-by-zero protection.
  * **Task 2.2:** Implement **EVI** calculation (use appropriate coefficients).
  * **Task 2.3:** Optionally implement **SAVI** for areas with sparse vegetation.
  * **Task 2.4:** Create visualization maps for each index.
  * **Task 2.5:** Validate index ranges (NDVI: -1 to 1, EVI: -1 to 1).

> **Hint:** Add small epsilon ($1e^{-8}$) to denominators to avoid division errors. Clip output values to valid ranges.

### PHASE 3: Land Cover Classification

  * **Task 3.1:** Define classification thresholds based on NDVI values.
  * **Task 3.2:** Create categorical raster with classes: `non-forest`, `sparse`, `moderate`, `dense`.
  * **Task 3.3:** Calculate area (hectares) for each class.
  * **Task 3.4:** Generate classification map with appropriate color scheme.
  * **Task 3.5:** Create summary statistics table.

### PHASE 4: Biomass Estimation

  * **Task 4.1:** Select appropriate allometric equation for your region.
  * **Task 4.2:** Apply equation pixel-by-pixel to create biomass raster (tons/ha).
  * **Task 4.3:** Apply vegetation threshold mask (e.g., NDVI \> 0.2).
  * **Task 4.4:** Implement reasonable upper/lower biomass bounds.
  * **Task 4.5:** Calculate total biomass by summing all pixels and converting units.
  * **Task 4.6:** Generate biomass distribution map with proper legend.

> **Critical:** Remember unit conversions: pixel area = $100 m^2$ = 0.01 ha.
> $$Total\_biomass (tons) = \sum(biomass\_per\_pixel \times 0.01)$$

### PHASE 5: Analysis & Validation

  * **Task 5.1:** Compute descriptive statistics (mean, median, std, min, max).
  * **Task 5.2:** Create biomass distribution histogram.
  * **Task 5.3:** Compare results with published values for similar ecosystems.
  * **Task 5.4:** Perform sensitivity analysis on key parameters.
  * **Task 5.5:** Document assumptions and uncertainty sources.

### PHASE 6: Water Detection

  * **Task 6.1:** Calculate **NDWI** and **MNDWI** indices.
  * **Task 6.2:** Apply thresholds to create water masks.
  * **Task 6.3:** Classify water bodies by permanence/type.
  * **Task 6.4:** Calculate total water area and percentages.

### PHASE 7: Water Quality

  * **Task 7.1:** Calculate turbidity index (**NDTI**).
  * **Task 7.2:** Create water quality maps.
  * **Task 7.3:** Identify degraded vs. healthy water bodies.

### PHASE 8: Hydrological Analysis

  * **Task 8.1:** Create riparian buffer zones (30m, 100m, 300m).
  * **Task 8.2:** Calculate buffer vegetation quality (NDVI).
  * **Task 8.3:** Assess wetland connectivity.
      * *Formula:* $$Connectivity = f(distance\_to\_water, corridor\_quality, area\_size)$$
      * *Note:* Higher connectivity = greater contribution to aquifer recharge and flow regulation.
  * **Task 8.4:** Identify priority conservation areas.

### PHASE 9: Ecosystem Service Quantification

Quantify each service using biophysical models validated in scientific literature. All values are calculated per pixel and then aggregated.

  * **Task 9.1:** Calculate biophysical metrics for all 5 services.
  * **Task 9.2:** Create service provision maps.
  * **Task 9.3:** Generate statistics tables.
  * **Task 9.4:** Identify service hotspots.

#### 9.1 Water Flow Regulation

**Metric:** Water storage capacity ($m^3/ha/year$)

| Parameter | Values/Calculation |
| :--- | :--- |
| **Area** | Pixel area in hectares (0.1 ha for 10m pixels) |
| **Depth** | Assumed average: 0.5-2m for wetlands, 0.2-0.5m for riparian |
| **Retention\_Factor** | 0.6-0.9 (depends on soil, vegetation) |
| **Vegetation\_Factor** | $1.0 + (NDVI \times 0.5)$ – accounts for evapotranspiration |

#### 9.2 Water Purification

**Metric:** Pollutant removal capacity ($kg/ha/year$)

$$Purification = Filtration\_Rate \times Vegetation\_Quality \times Contact\_Time$$

  * **Filtration\_Rate:** 500-2000 kg/ha/yr (baseline from literature).
  * **Vegetation\_Quality:** NDVI-based (0.2-1.0 multiplier).
  * **Contact\_Time:** Function of flow velocity and wetland area.

> **Simplified approach:**
> $$Purification \approx 800 \times NDVI \times (Wetland\_Area / Total\_Area)$$

#### 9.3 Sediment Control

**Metric:** Sediment retention ($tons/ha/year$)

$$Sediment\_Retention = USLE \times SDR \times Trap\_Efficiency$$

Where USLE (Universal Soil Loss Equation) can be simplified as:

  * **Erosion proxy:** $(1 - NDVI) \times slope\_factor$ (if DEM available).
  * **SDR (Sediment Delivery Ratio):** 0.1-0.4 (distance-based).
  * **Trap\_Efficiency:** 0.6-0.95 for vegetated riparian zones.

> **Without DEM:**
> $$Sediment\_Retention \approx 15 \times NDVI \times Buffer\_Width\_Factor$$

#### 9.4 Aquifer Recharge Control

**Metric:** Groundwater recharge potential ($mm/year$)

$$Recharge = Precipitation \times Infiltration\_Rate \times (1 - Runoff\_Coefficient)$$

  * **Precipitation:** Use regional annual average (mm/year).
  * **Infiltration\_Rate:** 0.3-0.8 for wetlands, 0.1-0.4 for other areas.
  * **Runoff\_Coefficient:** Function of land cover (0.1 for forests, 0.8 for urban).
  * **Hint:** Higher NDVI = better infiltration. Use NDVI as proxy for infiltration capacity.

#### 9.5 Flood Protection

**Metric:** Flood storage capacity ($m^3$) and protection value.

$$Flood\_Storage = Floodplain\_Area \times Storage\_Depth \times Roughness\_Factor$$

  * **Floodplain\_Area:** Identify using elevation/slope (if DEM) or buffers.
  * **Storage\_Depth:** 0.5-3m (depends on ecosystem type).
  * **Roughness\_Factor:** Manning’s n equivalent, NDVI-based (0.5-1.5).
  * **Protection Value:** Calculate avoided flood damage based on downstream population or infrastructure. Use simplified: **$1,000-10,000 per m3** of storage capacity.

### PHASE 10: Dynamic Ecosystem Service Valuation

Convert biophysical metrics to economic values using benefit transfer methods and temporal adjustment factors.

  * **Task 10.1:** Apply valuation coefficients.
  * **Task 10.2:** Calculate adjustment factors.
  * **Task 10.3:** Compute total ecosystem service value.
  * **Task 10.4:** Create value distribution maps.
  * **Task 10.5:** Prepare final ESV report.

#### 10.1 Valuation Coefficients

Use globally-averaged values adjusted for local context (Source: de Groot et al., 2012; Costanza et al., 2014).

| Ecosystem Service | Value Range ($/ha/year) | Suggested |
| :--- | :--- | :--- |
| Water Flow Regulation | \$200 - \$5,000 | **\$1,500** |
| Water Purification | \$500 - \$10,000 | **\$3,000** |
| Sediment Control | \$100 - \$2,000 | **\$800** |
| Aquifer Recharge | \$50 - \$1,500 | **\$500** |
| Flood Protection | \$1,000 - \$25,000 | **\$8,000\*\* |

> **Important:** These are baseline values. Adjust based on local economic conditions, water scarcity, and downstream beneficiary population.

#### 10.2 Dynamic Adjustment Factors

Make valuations dynamic by incorporating temporal and spatial variability:

$$Dynamic\_Value = Base\_Value \times Quality\_Factor \times Scarcity\_Factor \times Benefit\_Factor$$

  * **Quality\_Factor:** Based on actual ecosystem condition (NDVI, water quality indices). Range: 0.3-1.5.
  * **Scarcity\_Factor:** Water scarcity coefficient for region. Range: 0.8-2.0 (higher in arid regions).
  * **Benefit\_Factor:** Population density / infrastructure value downstream. Range: 0.5-3.0.

> **Example Calculation:**
> Flood Protection Value = $\$8,000 \times 0.9$ (good quality) $\times 1.5$ (moderate scarcity) $\times 2.0$ (high downstream benefit) = **$21,600/ha/year**

#### 10.3 Total Ecosystem Service Value

Calculate total value by summing all services for each pixel/ecosystem unit:

$$Total\_ESV = \sum [Service\_i \times Area\_i \times Quality\_Factor\_i \times Adjustment\_Factors]$$

**Report results as:**

  * **Total annual value:** $ per year for entire study area.
  * **Unit value:** $ per hectare per year.
  * **Spatial distribution:** Maps showing high-value hotspots.

### PHASE 11: Documentation & Presentation

  * **Task 11.1:** Write methodology section explaining approach.
  * **Task 11.2:** Create professional figures with proper titles and labels.
  * **Task 11.3:** Prepare final report (PDF format).
  * **Task 11.4:** Create 5-minute presentation slides.
  * **Task 11.5:** Organize code with comments and documentation.

-----

## 4\. Deliverables

### Required Submission

**1. Jupyter Notebook (.ipynb)**

  * Complete executable code.
  * Explanatory comments.
  * Visible results.

**2. PDF Report (3-5 pages) containing:**

  * Introduction and objectives.
  * Applied methodology.
  * Main results (with maps and charts).
  * Analysis and discussion.
  * Conclusions.

### Optional Submission (Extra Points)

  * Interactive map with Folium.
  * Temporal comparison (if multiple dates available).
  * Uncertainty analysis.
  * Simple web dashboard.
  * Sub-regional analysis.

-----

## 5\. Evaluation Criteria

| Criterion | Weight | Description |
| :--- | :--- | :--- |
| **Scientific Rigor** | 25% | Correct methodology, appropriate references, justified decisions. |
| **Technical Quality** | 25% | Functional code, clean, well-documented, reproducible. |
| **Result Accuracy** | 20% | Values within expected ranges, adequate validation. |
| **Visualization** | 15% | Clear maps, informative charts, professional design. |
| **Analysis & Interpretation** | 10% | Significant insights, discussion of limitations. |
| **Presentation** | 5% | Clarity, structure, effective communication. |

### Detailed Rubric

**Excellent (90-100%):**

  * Complete and correct implementation - Multiple methods compared.
  * Professional visualizations.
  * Deep analysis with valuable insights.
  * Perfectly documented code.

**Good (75-89%):**

  * Correct implementation of main method.
  * Valid and well-presented results.
  * Functional and documented code.
  * Good interpretation of results.

**Acceptable (60-74%):**

  * Basic method implemented.
  * Reasonable results.
  * Functional but improvable code.
  * Basic interpretation.

**Insufficient (\<60%):**

  * Incomplete implementation.
  * Incorrect or unvalidated results.
  * Non-functional code.
  * Lack of analysis.

-----

## 6\. Frequently Asked Questions

### About Data

  * **Q: What do I do if my image has clouds?**
      * A: You can mask clouds using thresholds in specific bands or use the average of multiple dates if available.
  * **Q: Biomass values seem too high/low?**
      * A: Verify you're using the correct equation for your forest type and that indices are in correct ranges (NDVI: 0-1).
  * **Q: Can I use other equations I find in papers?**
      * A: Yes, but you must cite them appropriately and justify why they're suitable for your area.

### About Code

  * **Q: Can I use other libraries not mentioned?**
      * A: Yes, as long as they're available in standard Python environments.
  * **Q: Do we have to use Google Earth Engine?**
      * A: Not mandatory. Data is already downloaded and ready to use.
  * **Q: What if my code gives a memory error?**
      * A: Process the image in blocks or temporarily reduce resolution for testing.

### About Presentation

  * **Q: What should the final presentation include?**
      * A: Brief context, methodology, main results (maps + numbers), conclusions. Maximum 5 minutes.
  * **Q: Should all members present?**
      * A: Recommended that all participate, but you can organize as you prefer.

-----

## 7\. Additional Resources

### Useful Websites

  * **Sentinel-2:** [https://dataspace.copernicus.eu/data-collections/copernicus-sentinel-data/sentinel-2](https://dataspace.copernicus.eu/data-collections/copernicus-sentinel-data/sentinel-2)
  * **Download shapefiles:** [https://mapscaping.com/download-shapefiles-for-any-country/](https://mapscaping.com/download-shapefiles-for-any-country/)
  * **QGIS:** [https://qgis.org/](https://qgis.org/)
  * **Natural Earth Data:** [https://www.naturalearthdata.com/](https://www.naturalearthdata.com/)
  * **DIVA-GIS:** [https://www.diva-gis.org/gdata](https://www.google.com/search?q=https://www.diva-gis.org/gdata)
  * **OpenStreetMap:** [https://www.openstreetmap.org/](https://www.openstreetmap.org/)
  * **Google Earth Engine:** [https://earthengine.google.com](https://earthengine.google.com)
  * **NASA Earthdata:** [https://earthdata.nasa.gov](https://earthdata.nasa.gov)
  * **FAO Forest Resources:** [http://www.fao.org/forestry](http://www.fao.org/forestry)
