# 15-Minute City Accessibility Model & Index Scoring (QGIS Capstone)

![QGIS](https://img.shields.io/badge/QGIS-3.34_LTR-588240?logo=qgis&logoColor=white)
![Framework](https://img.shields.io/badge/Urban_Model-15--Minute_City_Index-blue)
![CRS](https://img.shields.io/badge/Projection-EPSG%3A3035%20(ETRS89)-003399)

## Project Overview
This repository serves as the flagship capstone project for a 9-part QGIS Spatial Analysis Portfolio series. 

It implements a comprehensive **15-Minute City Accessibility Model** for Brussels, evaluating pedestrian walking access ($4\text{ km/h}$ base speed) across five essential daily service domains: **Groceries**, **Education**, **Healthcare**, **Public Green Space**, and **Public Transit**.

Using network-based isochrone processing, multi-criteria evaluation (MCE) field algorithms, and spatial demographic overlays, the model assigns a standardized **15-Minute City Score ($0 - 100$)** to every neighborhood unit and quantifies population coverage across accessibility tiers.

---

## Service Domains & Weighting Matrix

Based on European urban liveability standards (e.g., Paris 15-Minute City & C40 Cities Framework):

| Domain | Included OSM Amenity Tags | Distance / Time Thresholds | Score Weight ($W_i$) |
| :--- | :--- | :--- | :---: |
| **1. Fresh Food & Groceries** | `shop=supermarket`, `shop=bakery`, `amenity=marketplace` | $\le 5\text{ min } (333\text{ m}) = 100\%$<br>$\le 10\text{ min } (667\text{ m}) = 60\%$<br>$\le 15\text{ min } (1000\text{ m}) = 20\%$ | **25%** |
| **2. Education & Childcare** | `amenity=kindergarten`, `amenity=school` | Same decay | **20%** |
| **3. Healthcare & Pharmacy** | `amenity=pharmacy`, `amenity=clinic`, `amenity=hospital` | Same decay | **20%** |
| **4. Green Space & Parks** | `leisure=park`, `leisure=playground`, `landuse=recreation_ground` | Same decay | **15%** |
| **5. Public Transit Hubs** | `highway=bus_stop`, `railway=station`, `building=train_station` | Same decay | **20%** |

---

## Composite Index Formula

For each spatial analysis unit $k$ (e.g., $250\text{ m}$ grid cell or H3 hexagon), domain sub-scores $S_{i,k} \in [0, 100]$ are calculated based on shortest network walking time $t_{i,k}$:

$$S_{i,k} = \begin{cases}  100 & \text{if } t_{i,k} \le 5\text{ min} \\ 100 - \left( \frac{t_{i,k} - 5}{10} \right) \times 80 & \text{if } 5 < t_{i,k} \le 15\text{ min} \\ 0 & \text{if } t_{i,k} > 15\text{ min} \end{cases}$$

The final composite **15-Minute City Index ($FMC_k$)** is derived via Weighted Linear Combination:

$$FMC_k = \sum_{i=1}^{5} \left( S_{i,k} \times W_i \right)$$

---

## Workflow Implementation

### Step 1: Pedestrian Graph Setup & Time Field
* **QGIS Manual Reference:** *Section 6.3.4 - Speed-Aware Distance Modeling*
* Set pedestrian design speed to $4.0\text{ km/h} \approx 66.67\text{ m/min}$.
* Calculated line segment cost in minutes (`walk_min`):
  ```sql
  $length / 66.67
  ```
### Step 2: Network Service Areas & Isochrones
* QGIS Manual Reference: Section 6.3.6 - Service Area (from layer)
* Ran Service Area (from layer) for each of the 5 POI layers at distance break-points:
   * $333.3\text{ m}$ ($5\text{ minutes}$)
   * $666.7\text{ m}$ ($10\text{ minutes}$)
  * $1000.0\text{ m}$ ($15\text{ minutes}$)
* Enclosed line networks into polygon vector boundaries using Concave Hull (Alpha Shape).

### Step 3: Grid Scoring & Spatial Join
* QGIS Manual Reference: Section 6.2, 18.14 - Vector Overlays & Field Calculator
* Joined domain polygon isochrones to a standardized $250\text{ m}$ vector analysis grid (grid_250m).
* Executed expression-based scoring to calculate $FMC_k$ and joined Eurostat population grids to determine demographic counts per tier.
