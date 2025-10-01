# ENTSO-E Data Fetch

This repository hosts the Jupyter notebook `ENTSO-E_data_retrieve.ipynb`, which automates retrieval of congestion and generation metrics from the ENTSO-E Transparency platform. It wraps the `entsoe-py` client and supporting helpers to pull day-ahead net transfer capacity, cross-border physical flows, national load, installed generation capacity, and legacy pre-2015 cross-border files.

## Features

- Day-ahead net transfer capacity summaries per interconnection using the official ENTSO-E neighbours mapping.
- Cross-border physical flows, national load, and installed generation capacity snapshots saved as reusable CSV files.
- Legacy (pre-2015) cross-border schedule downloader with optional CSV conversion and aggregation helper.
- Configurable country lists and date ranges organised in one repeatable notebook workflow.
- All generated artefacts collected under `saved_data/` for downstream analysis.

## Prerequisites

- Python 3.11 (the notebook metadata targets 3.11.11; 3.10+ should also work).
- An ENTSO-E Transparency API token (free registration at https://transparency.entsoe.eu/).
- Ability to run Jupyter Notebook or JupyterLab.

## Setup

1. Clone the repository and move into it.
   ```
   git clone https://github.com/your-org/ENTSO-E_DataFetch.git
   cd ENTSO-E_DataFetch
   ```
2. Create and activate a virtual environment (example using `venv`):
   ```
   python -m venv .venv
   .venv\\Scripts\\activate
   python -m pip install --upgrade pip
   ```
3. Install runtime dependencies:
   ```
   pip install jupyterlab notebook pandas numpy entsoe-py requests openpyxl
   ```
   You can use Conda instead if you prefer: `conda create -n entsoe python=3.11 pandas numpy entsoe-py requests openpyxl jupyterlab`.
4. Keep the repository clean by making sure `saved_data/` remains writable; it is already committed with placeholders.

## Configuration

- Request an API token from the ENTSO-E Transparency portal and paste it into `API_KEY.txt` (the file should contain only the token, no extra spaces).
- Review the `country_codes` list and the `year` variable at the top of the notebook to scope queries to your needs.
- The notebook assumes Brussels time (`Europe/Brussels`) and will respect ENTSO-E rate limits via short sleeps; adjust the pauses if you hit throttling.
- Legacy downloads write into `saved_data/before_2015/`; the helper keeps a `country_not_available.txt` list with codes that have missing legacy files.

## Usage

1. Launch Jupyter from the activated environment:
   ```
   jupyter lab ENTSO-E_data_retrieve.ipynb
   ```
   or open the notebook via the classic interface with `jupyter notebook`.
2. Execute the notebook cells in order. Each section is independent, so you can re-run only the parts you need:
   - **Net Transfer capacity - Day Ahead (MW)**: iterates over official neighbour pairs and saves `saved_data/NTC_dayahead_df_{year}.csv` with mean and max capacity values.
   - **Cross-border Flow (MW)**: queries day-ahead physical flows per interconnection and saves `saved_data/cross_border_{year}.csv`.
   - **Load**: collects hourly load for each country and writes `saved_data/load_df_{year}.csv`.
   - **Installed capacity**: fetches installed generation capacities (by technology columns) and writes `saved_data/capacity_df_{year}.csv`.
   - **Cross-border flows before 2015 not included in the API**: downloads the 2011 legacy Excel files directly from the Transparency website. Set `save_as_csv = True` to auto-convert each download to CSV (`saved_data/before_2015/cross_border_schedule_2011_{CODE}.csv`).
   - **Process the downloaded data...**: cleans the legacy CSVs and produces `saved_data/before_2015/processed/aggregated_physical_crossborder_flows_2011.csv`. It logs files it could not process and skips empty datasets.
3. Inspect the console output: the notebook prints progress per country and notes any missing endpoints (for example, countries listed in `saved_data/before_2015/country_not_available.txt`). Adjust the country list or rerun specific cells as needed.

## Output overview

- `saved_data/NTC_dayahead_df_{year}.csv`: Mean and max day-ahead net transfer capacity for every neighbour pair (MW).
- `saved_data/cross_border_{year}.csv`: Summary of cross-border physical flows for the requested year (MW).
- `saved_data/load_df_{year}.csv`: Hourly load aggregation per country with peak values across the year (MW).
- `saved_data/capacity_df_{year}.csv`: Installed generation capacity per technology for each country (MW).
- `saved_data/before_2015/cross_border_schedule_2011_{CODE}.csv`: Raw 2011 legacy cross-border schedules downloaded from the ENTSO-E archive.
- `saved_data/before_2015/processed/aggregated_physical_crossborder_flows_2011.csv`: Cleaned and aggregated legacy flows from the processed CSVs.
- `saved_data/before_2015/country_not_available.txt`: Comma-separated list of countries whose legacy files were missing or malformed.

## Tips and troubleshooting

- The ENTSO-E API enforces strict rate limits. The notebook already includes `time.sleep` calls; increase the delay if you see HTTP 429 responses.
- For persistent `Error for pair` messages in the NTC or cross-border sections, verify that the neighbour pair exists in the ENTSO-E map or temporarily comment it out.
- Legacy downloads occasionally respond with "File is not a zip file" -- this indicates that the historic dataset is not published for that country/year; the name is recorded in `country_not_available.txt`.
- When re-running only some sections, delete or rename the corresponding CSV if you want to regenerate it from scratch.

## License

This project is distributed under the terms described in `LICENSE`.

