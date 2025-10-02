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

1. Launch Jupyter from the activated environment by running Jupyter Lab with the notebook path (for example, "jupyter lab ENTSO-E_data_retrieve.ipynb"). You can also open the notebook through the classic interface by starting Jupyter Notebook and navigating to the file.
2. Execute the notebook cells in order. Each section is independent, so you can re-run only the parts you need:
   - **Net Transfer capacity - Day Ahead (MW)**: iterates over official neighbour pairs and stores year-specific CSV summaries with mean and maximum capacity values in the saved data folder.
   - **Cross-border Flow (MW)**: queries day-ahead physical flows per interconnection and records the results as annual CSV exports under the saved data directory.
   - **Load**: collects hourly load for each country and generates per-year CSV datasets that capture both time series values and peaks.
   - **Installed capacity**: fetches installed generation capacities (by technology columns) and writes the figures into yearly CSV snapshots for each country.
   - **Cross-border flows before 2015 not included in the API**: downloads the 2011 legacy Excel files directly from the Transparency website. Set `save_as_csv = True` to auto-convert each download to CSV files that follow the same naming pattern as the source spreadsheets.
   - **Process the downloaded data...**: cleans the legacy CSVs, produces an aggregated dataset for the historic flows, and logs any inputs it could not process.
3. Inspect the console output: the notebook prints progress per country and notes any missing endpoints (for example, countries listed in the legacy availability log). Adjust the country list or rerun specific cells as needed.

## Output overview

- A CSV per year capturing mean and max day-ahead net transfer capacity for every neighbour pair (MW).
- A CSV per year summarising cross-border physical flows for the requested period (MW).
- A CSV per year aggregating hourly load per country alongside peak values (MW).
- A CSV per year listing installed generation capacity per technology for each country (MW).
- Year-specific legacy downloads from the ENTSO-E archive containing raw 2011 cross-border schedules.
- A processed legacy dataset that aggregates the cleaned cross-border flows from the historic CSVs.
- A text log enumerating countries whose legacy files were missing or malformed.

## Tips and troubleshooting

- The ENTSO-E API enforces strict rate limits. The notebook already includes `time.sleep` calls; increase the delay if you see HTTP 429 responses.
- For persistent `Error for pair` messages in the NTC or cross-border sections, verify that the neighbour pair exists in the ENTSO-E map or temporarily comment it out.
- Legacy downloads occasionally respond with "File is not a zip file" -- this indicates that the historic dataset is not published for that country/year; the name is recorded in `country_not_available.txt`.
- When re-running only some sections, delete or rename the corresponding CSV if you want to regenerate it from scratch.

## License

This project is distributed under the terms described in `LICENSE`.

