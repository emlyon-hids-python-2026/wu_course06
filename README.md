# MedicStore - Sales Reporting

Case study of the Python Programming course (Prework 6, emlyon business school).

**Live app:** [https://medicstore-wu.streamlit.app/](https://medicstore-wu.streamlit.app/)

You are the manager of **MedicStore**, a company selling medical products in Europe. Until now the team
consolidated and analysed the sales in Excel, which led to manual errors and lost time. This project
automates the whole process: consolidate the sales files, answer the questions of the Data Team and of the
Operational Team, draw the graphs and export the insights to Excel so that they can be reused in the next
presentation to the board.

The same analysis is delivered in **three versions**:

| Version | Format | File | Run |
|---|---|---|---|
| 1 | Jupyter notebook | [notebooks/02_medicstore_notebook.ipynb](notebooks/02_medicstore_notebook.ipynb) | Open in VSCode, *Run All* |
| 2 | Script with functions | [scripts/04_medicstore_functions.py](scripts/04_medicstore_functions.py) | `uv run scripts/04_medicstore_functions.py` |
| 3 | Streamlit web app | [app/medicstore_app.py](app/medicstore_app.py) | `uv run streamlit run app/medicstore_app.py` |

## Table of contents

- [Key results](#key-results)
- [Requirements](#requirements)
- [Installation](#installation)
- [Get the data](#get-the-data)
- [Usage](#usage)
- [The pipeline](#the-pipeline)
- [The Streamlit app](#the-streamlit-app)
- [Project structure](#project-structure)
- [Development](#development)
- [Author](#author)

## Key results

On the 7,886 sales lines (2016 - 2019, 15 European countries):

| Question | Answer |
|---|---|
| Lines in `Sales_MedicStore_part1.csv` / `part2.csv` / `Mapping.xlsx` | 19 / 7,867 / 17 |
| Number of orders (unique `Order ID`) | 3,744 |
| Total sales amount | 2,290,823 |
| Top 3 customers (order count) | Frank Olsen (13), Paul Stevenson (12), Yoseph Carroll (12) |
| Total sales for France | 650,730 |
| Share of France in the total sales | 28.4 % |

`total_sales` is computed for each line as `Quantity × Unit Price`. One order can contain several lines
(one per product), so the number of orders counts the unique `Order ID`.

## Requirements

- [uv](https://docs.astral.sh/uv/) (installs Python and the dependencies)
- Python 3.14 (pinned in `.python-version`, installed automatically by `uv`)
- [VSCode](https://code.visualstudio.com/) is recommended to run the notebook

Main dependencies: `pandas`, `matplotlib`, `openpyxl`, `streamlit` (it also provides `altair`),
`ruff`, `ipykernel`. See [pyproject.toml](pyproject.toml).

## Installation

```bash
git clone https://github.com/emlyon-hids-python-2026/wu_course06.git
cd wu_course06
uv sync          # or: make install
```

## Get the data

The data is **not tracked by git** (`data/*` is ignored). The 3 raw files must be in the
`data/input_medicstore/` folder:

| File | Content |
|---|---|
| `Sales_MedicStore_part1.csv` | First part of the sales (19 lines) |
| `Sales_MedicStore_part2.csv` | Second part of the sales (7,867 lines) |
| `Mapping .xlsx` | Mapping between `Sub-Category` and `Category` (17 lines) |

Two options to get them:

- **Manually:** download them from Brightspace and paste them in `data/input_medicstore/`.
- **With a script:** use the `download_data.py` script given in the Prework 6 guidelines.

Things to know about the files:

- The CSV files use `;` as separator (`pd.read_csv(..., sep=";")`).
- The mapping file name contains a space before `.xlsx` (`Mapping .xlsx`): the code uses this exact name.
- Dates are written `dd/mm/yyyy` and are converted to real dates.

## Usage

```bash
# Version 2: script with functions
uv run scripts/04_medicstore_functions.py

# Version 3: Streamlit app
uv run streamlit run app/medicstore_app.py
```

Run the commands from the project root. Other commands:

| Command | What it does |
|---|---|
| `make install` | Install the dependencies (`uv sync`) |
| `make lint` | Check the code with ruff |
| `make format` | Format the code with ruff |
| `make clean` | Remove the caches |

The notebook writes its results in `data/output_medicstore/`, the script in `data/output_medicstore_v3/`:

- `medicstore_final.xlsx`: the consolidated dataset
- `medicstore_analysis.xlsx`: the final dataset, the analysis tables (one sheet each) and the graphs
- 3 `.png` graphs: orders per country, sales per category, sales per year

## The pipeline

```text
Sales_MedicStore_part1.csv ─┐
                            ├─ concat ──> sales ─┐
Sales_MedicStore_part2.csv ─┘                    ├─ merge (on Sub-Category) ──> + total_sales ──> analysis ──> Excel
                                Mapping .xlsx ───┘
```

1. **Ingestion:** load the CSV files with `pd.read_csv(sep=";")` and the mapping with `pd.read_excel`.
2. **Transformation:** concatenate the parts (`pd.concat`), add the `Category` column with a left merge on
   `Sub-Category`, compute `total_sales = Quantity × Unit Price` and convert the dates. Two checks make
   sure that no line was lost and that every sub-category has a category.
3. **Analysis:**
   - Technical questions (Data Team): shapes, column names, top / last 5 lines, statistical description.
   - Business questions (Operational Team): number of orders, total sales, top 3 customers, total sales
     for France and its share of the total.
4. **Graphs:** the guidelines ask for "visits per gender / department", which do not exist in the sales data.
   They are adapted to orders per country, sales per category and sales per year.
5. **Export:** one Excel file with the tables (one sheet each) and the graphs. The answers are stored in a
   dictionary, then converted to a `Question / Answer` DataFrame.

## The Streamlit app

**Online:** [https://medicstore-wu.streamlit.app/](https://medicstore-wu.streamlit.app/), usable from a
computer or a smartphone browser.

**Locally:**

```bash
uv run streamlit run app/medicstore_app.py
```

Run it from the project root: Streamlit reads the theme from `.streamlit/config.toml` in the launch folder.

The app reproduces the whole pipeline on files loaded by the user:

1. In the sidebar, load **as many sales parts as you have** (CSV, `;` separator) and the mapping (Excel).
2. Optionally **filter** the sales by country, category and year.
3. Browse the four tabs:
   - **Ingestion:** size and preview of each raw file.
   - **Transformation:** consolidation steps, quality checks, final dataset and summary statistics.
   - **Analysis:** key figures (orders, total sales, quantity sold, average sales per order), business
     questions and interactive graphs (orders per country, sales per category, per year, per sub-category).
   - **Export:** download the Excel report, the filtered dataset (CSV) and the graphs (PNG zip).

The files are never read from `data/`: the user always loads them, so the deployed app works without any
data in the repository.

## Project structure

```text
wu_course06/
├── app/
│   ├── medicstore_app.py           # Streamlit app (pipeline, analysis, charts, export, UI)
│   └── assets/
│       ├── medicstore_logo.svg     # Logo displayed in the sidebar
│       └── style.css               # Custom CSS
├── .streamlit/
│   └── config.toml                 # Streamlit theme (light and dark)
├── data/                           # Datasets and outputs (not tracked by git)
│   ├── input_medicstore/           # The 3 raw files
│   ├── output_medicstore/          # Outputs of the notebook
│   └── output_medicstore_v3/       # Outputs of the script
├── notebooks/
│   └── 02_medicstore_notebook.ipynb   # Version 1
├── scripts/
│   ├── demo.py
│   └── 04_medicstore_functions.py     # Version 2
├── src/do_course06/                # Python package (placeholder)
├── Makefile
├── pyproject.toml
└── uv.lock
```

## Development

```bash
make lint      # ruff check
make format    # ruff format
```

- Commit messages follow the [Conventional Commits](https://www.conventionalcommits.org/) style
  (`feat:`, `fix:`, `docs:`, `chore:`, `refactor:`).
- Bare expressions at the end of a cell are allowed in the notebook: they display the value
  (rule `B018` ignored in `pyproject.toml`).

## Author

Qiongyi WU - [qiongyi.wu@edu.em-lyon.com](mailto:qiongyi.wu@edu.em-lyon.com)
