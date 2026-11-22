# Note sure what the title is

Exploring an adaptive voltage level control instead of the traditional CVR or VVO.

## Contributors
Midrar Adham

## Short Description

The project studies how residential smart inverters can support voltage on a split-phase (120/240 V)
secondary feeder. It uses GridLAB-D steady-state power flow over a full day of measured house demand, and it
checks the simulation against hand calculations (backward/forward sweep). The first research question (RQ1)
measures how much one inverter's reactive power moves the voltages along the feeder, and how that depends on
the inverter's location, its size and count, and the voltage at the transformer primary.

## Tech Stack

* **Simulation:** GridLAB-D 5.3 (Newton-Raphson power flow)
* **Analysis and plots:** Python 3.14 (pandas, matplotlib), managed with Poetry
* **Write-up:** Quarto book (HTML and PDF; the PDF needs a TeX installation)

## Repository Contents

| Path | Contents |
|---|---|
| `gld/` | GridLAB-D models, study scripts and results. See [gld/README.md](gld/README.md). |
| `gld/basecase/residential_8house.glm` | **Baseline model**: 8 houses on a 50 kVA transformer. |
| `gld/rq1/` | RQ1 studies, one numbered folder each. See [gld/rq1/README.md](gld/rq1/README.md). |
| `docs/` | Quarto book: literature, equation derivations, research outline and results chapters. |
| `docs/references.bib` | Bibliography, exported from Zotero. Edit it in Zotero, not by hand. |

## Getting Started

### Prerequisites

* [Poetry](https://python-poetry.org/)
* [GridLAB-D](https://www.gridlabd.org/) 5.3 on the `PATH` (`gridlabd --version`)
* [Quarto](https://quarto.org/) 1.10 or later, plus TeX if you want the PDF

### Installation

1. **Clone the repository:**
   ```bash
   git clone git@github.com:PortlandStatePowerLab/midrar_adaptive_vvo_2026.git
   cd midrar_adaptive_vvo_2026
   ```
2. **Install the Python environment:**
   ```bash
   poetry install
   ```

### Run a study and build the book

All commands run from the repository root.

```bash
# Baseline: 8-house feeder, full day, no inverters
poetry run python gld/rq1/04_eight_houses/run.py
poetry run python gld/rq1/04_eight_houses/plot_base_day.py

# Build the book (HTML and PDF in docs/_book/)
cd docs && poetry run quarto render
```

Simulation outputs (`cases/` and `results/` folders) are not tracked in git; the scripts regenerate them.
Figures used by the book are copied into `docs/figures_qmd/`.

### If I want to work on this project, where should I start from?

1. Read `docs/sections/ch3_research_outline.qmd` for the research questions.
2. Read ch4 in the book (`docs/sections/ch4_initial_testing.qmd`): it walks through the baseline feeder, the
   hand calculations that reproduce GridLAB-D, and the first inverter results.
3. Then read [gld/rq1/README.md](gld/rq1/README.md) and run study 04.

## Resources
- Use the voltage_management folder on Zotero: psu-peg/SLR/Grid Services/voltage_management/
- The research questions are in `docs/sections/ch3_research_outline.qmd`.
