# 🚛 Two-Echelon Vehicle Routing Problem (2E-VRP)
### Urban Waste Collection Optimization — Tirupati City

---

## 📌 Project Overview

This project implements a **Two-Echelon Vehicle Routing Problem (2E-VRP)** to optimize daily municipal solid waste collection across **303 nodes** (1 depot + 302 collection points) in Tirupati city, Andhra Pradesh, India.

The system uses a **two-tier vehicle architecture**:
- **Tier 1 — Big Vehicles:** Operate on main roads. Travel only between the central depot and a cluster hub.
- **Tier 2 — Small Vehicles (100 kg capacity):** Navigate narrow lanes. Collect waste from individual shops/markets and bring it to the hub for the big vehicle to transport back to the depot.

All routes are planned within a strict **6:00 PM – 11:00 PM (300-minute)** operational time window.

---

## 🗺️ Problem Definition

| Parameter | Value |
|---|---|
| Problem Type | CVRPTW — Capacitated VRP with Time Windows |
| Total Nodes | 303 (1 Depot + 302 Collection Points) |
| Depot Location | Satya Bedu, Tirupati (13.6288°N, 79.4192°E) |
| Vehicle Capacity | 1000 kg (big vehicle) / 100 kg (small vehicle) |
| Time Window | 6:00 PM – 11:00 PM (300 minutes) |
| Number of Clusters | 9 (one big vehicle per cluster) |
| Objective | Minimize total distance travelled |
| Distance Formula | Haversine (real GPS-based distances) |

---

## 🏗️ System Architecture

```
DEPOT (Node 0)
    │
    │  Big Vehicle Route
    ▼
  HUB NODE  ◄──────────────────────────┐
    │                                  │
    │  Small Vehicle Routes             │  Small vehicles
    ▼                                  │  return to hub
SHOP / MARKET NODES ──────────────────┘
(Narrow roads, city_general zone)
```

**Flow:**
1. Big vehicle travels: `Depot → Hub → Depot`
2. Small vehicles travel: `Hub → Collection Nodes → Hub`
3. Small vehicles offload collected waste at the hub
4. Big vehicle carries all waste from hub back to depot

---

## 🔧 Tech Stack

| Library | Purpose |
|---|---|
| `pandas` | Data loading and manipulation |
| `numpy` | Numerical operations and random waste assignment |
| `scikit-learn` | K-Means clustering algorithm |
| `folium` | Interactive HTML maps on real city geography |
| `matplotlib` | Static scatter plots and route visualizations |
| `seaborn` | Color palettes for multi-cluster visualizations |
| `math` | Haversine distance formula (GPS-based distances) |
| `pulp` | Linear programming (imported for MILP extension) |

---

## 📁 Notebook Structure

The notebook is organized into the following sections:

### ⚙️ Setup & Data (CELL 0 – CELL 6-A)

| Cell | Description |
|---|---|
| **CELL 0** | Imports all libraries. Defines global constants: depot GPS, vehicle capacity, time window, waste values, Haversine function. |
| **CELL 1** | Loads 303-node CSV. Assigns waste per node (shop: 10/15/25 kg random, market: 200 kg fixed). Calculates minimum vehicles needed. |
| **CELL 2** | Interactive Folium map of all 303 nodes plotted on real Tirupati city geography. Click any marker for node details. |
| **CELL 3** | Static scatter plot of all nodes with node ID numbers labeled. Suitable for reports and printing. |
| **CELL 4** | Builds 303×303 Distance Matrix (km) and Time Matrix (minutes) using the Haversine formula. Used by all routing cells. |
| **CELL 5** | K-Means clustering divides 302 nodes into K geographic groups (one per vehicle). Does not enforce capacity yet. |
| **CELL 6** | **STRICT Zero-Overload Fix** — iteratively re-runs K-Means, increasing K by 1 each time any cluster exceeds capacity, until all clusters are within limits. |
| **CELL 6-A** | Displays final cluster-wise node assignment table for verification. |

### 🛠️ Helper Functions & Combined Visualizations (CELL 7 – Route Report)

| Cell | Description |
|---|---|
| **Helper Functions** | Defines `nearest_neighbor_route()` — Nearest Neighbor TSP heuristic. Initializes `ROUTES` global storage. Runs initial routing for all clusters. |
| **Combined Folium Map** | All 9 cluster routes drawn together on one interactive map. |
| **Combined Scatter Plot** | All 9 clusters plotted together on one static scatter plot. |
| **Routes Folium Map** | Vehicle travel paths shown as polylines on real city map. |
| **Animated Map (AntPath)** | Animated route visualization showing direction of vehicle travel. |
| **Route Report** | Printed summary: total distance, travel time, and stop count per cluster. |

### 🚛 Individual Cluster Processing (Clusters 0 – 8)

Each of the **9 clusters** is processed through **10 identical steps**:

| Step | Description |
|---|---|
| **STEP 1** | Filter cluster nodes from `df_nodes` into cluster-specific DataFrame. |
| **STEP 2** | Assign road types: `city_general` zone → `narrow_road`; all others → `main_road`. |
| **STEP 3** | **Hub Selection** — find node closest to cluster centroid using Haversine distance. |
| **STEP 4** | **Big Vehicle Route** — `Depot (Node 0) → Hub → Depot`. |
| **STEP 5** | **Collection Nodes** — all cluster nodes except the hub (small vehicles visit these). |
| **STEP 6** | **Small Vehicle Sub-Clusters** — greedy nearest-neighbor grouping, each group ≤ 100 kg. |
| **STEP 7** | **Small Vehicle Routes** — Nearest Neighbor TSP: `Hub → Nodes → Hub`. |
| **STEP 8** | Print complete route summary: big vehicle path + all small vehicle paths. |
| **STEP 9** | Static scatter plot — big vehicle (blue solid line) + small vehicles (dashed colored lines). |
| **STEP 10** | Interactive Folium map for this cluster on real city geography. |

### 🔄 Master Loop (STEP 1 – STEP 6)

Automates the complete pipeline for all 9 clusters in a single execution:

| Step | Description |
|---|---|
| **Master STEP 1** | Initialize master storage: `all_big_routes`, `all_small_routes`, `cluster_hubs`. |
| **Master STEP 2** | **Master Loop** — processes all 9 clusters automatically (hub selection → big route → sub-clusters → small routes). |
| **Master STEP 3** | Converts results into `df_big` and `df_small` summary DataFrames. |
| **Master STEP 4** | Prints complete formatted route output for all 9 clusters. |
| **Master STEP 5** | Master scatter plot — all 9 clusters and all routes on one 20×18 inch figure. |
| **Master STEP 6** | **Master Folium Map** — final interactive city map showing all routes across all 9 clusters. |

---

## 🧮 Key Algorithms

### 1. Haversine Distance Formula
Computes real-world GPS distance (km) between two coordinate points accounting for Earth's curvature.
```python
def haversine_km(lat1, lon1, lat2, lon2):
    lat1, lon1, lat2, lon2 = map(radians, [lat1, lon1, lat2, lon2])
    dlat = lat2 - lat1
    dlon = lon2 - lon1
    a = sin(dlat/2)**2 + cos(lat1) * cos(lat2) * sin(dlon/2)**2
    return 6371 * 2 * asin(sqrt(a))
```

### 2. STRICT Capacity-Aware K-Means Clustering
```
While any cluster exceeds vehicle capacity:
    K = K + 1
    Re-run K-Means with updated K
Until all clusters ≤ VEHICLE_CAPACITY kg
```

### 3. Nearest Neighbor TSP (Route Heuristic)
```
Start at hub node
While unvisited nodes remain:
    Move to the closest unvisited node
Return to hub node
```

### 4. Greedy Small Vehicle Sub-Clustering
```
While unassigned nodes remain:
    Start a new sub-cluster
    Add nearest node that fits within 100 kg capacity
    Repeat until no more nodes fit
    Close sub-cluster, start new one
```

---

## 📊 Output Summary

For each cluster the notebook produces:

- ✅ Big vehicle route: `Depot → Hub → Depot`
- ✅ Small vehicle routes: `Hub → [nodes] → Hub` (one per sub-cluster)
- ✅ Total distance (km) and travel time (minutes) per route
- ✅ Static scatter plot (printable)
- ✅ Interactive Folium map (clickable, zoomable)

---

## 🚀 How to Run

1. **Clone the repository**
```bash
git clone https://github.com/your-username/your-repo-name.git
```

2. **Install dependencies**
```bash
pip install pandas numpy scikit-learn folium matplotlib seaborn pulp
```

3. **Upload your node CSV** to `/content/all_303_nodes_tirupati.csv`  
   *(Required columns: `node_id`, `name`, `type`, `zone`, `lat`, `lon`)*

4. **Open in Google Colab or Jupyter Notebook** and run cells in order:
   - Run **CELL 0** first (imports & constants)
   - Run **CELL 1** through **CELL 6-A** (data setup)
   - Run **Helper Functions** cell
   - Run individual **Cluster 0–8** cells **or** run the **Master Loop** for all clusters at once

---

## 📂 File Structure

```
📦 repository
 ┣ 📓 D_V_D_MVRP_COMMENTED.ipynb     ← Main notebook (fully commented)
 ┣ 📄 all_303_nodes_tirupati.csv      ← Node dataset (303 nodes)
 ┗ 📄 README.md                       ← This file
```

---

## 👤 Author

**Nassiruddin**  
M.Sc Project — Vehicle Routing & Optimization  
Tirupati, Andhra Pradesh, India

---
