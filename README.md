# 🎮 CS2 Pro Match Analytics — IEM Dallas 2025

> Analyzing player performance, weapon effectiveness, map win rates, and utility usage for **Team Vitality** and **MOUZ** at IEM Dallas 2025 — the #1 and #2 ranked teams in the world — using parsed CS2 demo files.

---

## 🧩 Problem Statement

Professional CS2 matches generate rich event-level data through `.dem` demo files — every kill, damage, grenade, and round outcome is recorded tick by tick. Despite this, most post-match analysis relies on surface-level box scores.

This project asks: **what does raw demo data reveal about the performance differences between the world's top two CS2 teams, and how do they compare to the broader Top 100 player baseline?**

Specific questions explored:
- How do individual players perform across weapons and maps?
- Where do pros aim compared to Top 100 players?
- Which side (T vs CT) do teams win more on each map?
- Does utility usage (flashes, smokes, grenades) meaningfully correlate with round wins?

---

## 🗂️ Dataset

| Source | Contents |
|---|---|
| CS2 `.dem` files — [IEM Dallas 2025](https://github.com/tahaz/iemDallasMouzAndVitDems) | Every match played by Vitality or MOUZ at IEM Dallas 2025 |
| `maps_statistics.csv` | Top 100 player T-side / CT-side win rates by map |
| `weapons_statistics.csv` | Top 100 player hit distribution (HS%, Chest%, Leg%) and KPR by weapon |

Demo files were parsed using [`awpy`](https://github.com/pnxenopoulos/awpy) into structured DataFrames: `rounds`, `kills`, `damages`, `grenades`, `smokes`, `infernos`, `bomb`, `shots`.

### 🧑‍🤝‍🧑 Rosters

| Team | Players |
|---|---|
| **Vitality** | apEX, ropz, ZywOo, flameZ, mezii |
| **MOUZ** | Brollan, torzsi, Spinx, Jimpphat, xertioN |

---

## 🔧 Approach

### 📁 Demo Parsing & Data Ingestion

- Demos parsed with `awpy` into 8 event-type DataFrames per match
- Large tick-frequency DataFrames (e.g. grenades) filtered to every Nth tick to reduce size
- All match CSVs downloaded from GitHub programmatically via the GitHub API
- A custom `get_df()` function loads, labels, and combines all match files for a given team and event type, with optional roster filtering

### 🏆 Player Performance Matrix

For each player, the following stats were computed from the kills DataFrame:

| Metric | Description |
|---|---|
| `total_kills` | Total kills across all matches |
| `headshot_rate` | Overall HS% |
| `hs_percentage` | Headshots as % of total kills |
| `avg_distance` | Average kill distance |
| `rifle_hs_rate` / `rifle_kills` | Rifle-specific HS rate and kill count |
| `pistols_hs_rate` / `pistols_kills` | Pistol-specific HS rate and kill count |
| `avg_kills_per_map` | Average kills per map played |
| `normalized_consistency_score` | Normalized inverse of kill std across maps (higher = more consistent) |

### 🗺️ Map Win Rate Analysis

- T-side and CT-side win rates computed per map per team
- Pro team results merged with `maps_statistics.csv` (Top 100 baseline)
- Melted into long format for side-by-side bar chart visualization per map

### 🔫 Weapon Hit Distribution

- Hit location (head, chest, legs) computed per player per weapon from the damages DataFrame
- Arms and neck aggregated into `chest`; left/right legs aggregated into `leg`
- KPR (kills per round) added per player based on total rounds played
- Compared against Top 100 baseline from `weapons_statistics.csv`
- Visualized as bar charts, a difference heatmap (pro vs Top 100), and spider/radar charts

### 💣 Utility Analysis

- Grenade types unified across CT/T-side naming (`CFlashbang`/`CFlashbangProjectile` → `flash`, etc.)
- Utility thrown per map per round computed for: flash, smoke, grenade, incendiary, decoy
- Utility damage per round (HE grenade, Molotov/incendiary, flashbang) computed from damages DataFrame
- Flash-assisted kills extracted from the kills DataFrame
- **Point-biserial correlation** used to measure each utility metric's correlation with round win outcome (binary target)

---

## 📊 Key Findings

- **Pro players aim significantly higher than Top 100** — HS% across all weapons is consistently above the Top 100 baseline, while leg hit % is lower, indicating more precise targeting under professional conditions
- **Smokes and incendiaries show the strongest positive correlation with round wins** — utility usage is not just tactical flavor; it meaningfully predicts round outcomes
- **CT-side win rates dominate on most maps** — consistent with the broader CS2 meta, where defending is generally advantaged; both teams showed stronger CT performance
- **ZywOo and torzsi lead their respective teams** in normalized consistency score across maps — they perform at a high level regardless of opponent or map
- **AWP hit distribution diverges most from Top 100** — pro AWPers hit heads at a disproportionately higher rate, reflecting the precision expected at the elite level
- **MOUZ and Vitality cluster differently on the weapon radar charts** — MOUZ shows higher rifle HS rates while Vitality shows broader KPR distribution across more weapons

---

## 🗃️ Repository Structure

```
projects/
├── Milestone1.ipynb       # Full analysis notebook
└── (demo CSVs downloaded at runtime from GitHub via API)
```

> Demo CSV files are not committed to the repo due to size. They are downloaded at runtime from [`tahaz/iemDallasMouzAndVitDems`](https://github.com/tahaz/iemDallasMouzAndVitDems). The `maps_statistics.csv` and `weapons_statistics.csv` baselines must also be present in the working directory.

---

## 🚀 How to Run

```bash
# 1. Clone the repo
git clone https://github.com/alexjbyoon/projects.git
cd projects

# 2. Install dependencies
pip install pandas numpy matplotlib seaborn scipy awpy polars requests jupyter

# 3. Launch the notebook
jupyter notebook Milestone1.ipynb
```

The notebook will automatically download all demo CSVs from GitHub at runtime. If you want to parse your own `.dem` files, use the `parse_demos()` helper in the notebook with your local demo folder path.

---

## 📦 Requirements

| Package | Purpose |
|---|---|
| `pandas` | Data manipulation and merging |
| `numpy` | Numerical operations |
| `matplotlib` / `seaborn` | Visualizations (heatmaps, bar charts, radar plots) |
| `scipy` | Point-biserial correlation |
| `awpy` | CS2 demo file parsing |
| `polars` | High-performance DataFrame I/O from awpy |
| `requests` | GitHub API calls for CSV download |
| `jupyter` | Notebook environment |

Python 3.9+ recommended.
