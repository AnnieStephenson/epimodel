# Epidemics data processing and modelling toolkit

A data library and a toolkit for modelling COVID-19 epidemics.

## Main concepts

* Region database (continents, countries, provinces, GLEAM basins) - codes, names, basic stats, tree structure (TODO).
* All data, including the region database, is stored in [epimodel-covid-data](https://github.com/epidemics/epimodel-covid-data) repo for asynchronous updates.
* Each region has an ISO-based code, all datasets are organized by those codes (as a row index).
* Built on Pandas with some helpers, using mostly CSVs and HDF5.
* Algorithms and imports assuming common dataframe structure (with `Code` and optionally `Date` row index).
* All dates are UTC timestamps, stored in ISO format with TZ.

## Install

* [Get Poetry](https://python-poetry.org/docs/#installation)
* Clone this repository.
* Install the dependencies and this lib `poetry install` (creates a virtual env by default).
* Clone the [epimodel-covid-data](https://github.com/epidemics/epimodel-covid-data/) repository. For convenience, I recommend cloninig it inside the `epimodel` repo directory as `data`.

```sh
## Clone the repositories (or use their https://... withou github login)
git clone git@github.com:epidemics/epimodel.git
cd epimodel
git clone git@github.com:epidemics/epimodel-covid-data.git data

## Install packages
poetry install  # Best run it outside virtualenv - poetry will create its own
# Alternatively, you can also install PyMC3 or Pyro, and jupyter (in both cases):
poetry install -E pymc3
poetry install -E pyro

## Or, if using conda, install (a likely list): pandas pymc3 unidecode jupyter ...

poetry shell # One way to enter the virtualenv (if not active already)
poetry run jupyter notebook  # For example
```

## Basic usage

```python
from epimodel import RegionDataset, read_csv

# Read regions
rds = RegionDataset.load('data/regions.csv')
# Find by name or by code
cz = rds['CZ']
cz = rds.find_one_by_name('Czech Republic')
# Use attribute access on Region
print(cz.Name)
# TODO: attributes for tree-structure access

# Load John Hopkins CSSE dataset with our helper (creates indexes etc.)
csse = read_csv('data/johns-hopkins.csv')
print(csse.loc[('CZ', "2020-03-28")])
```

## Development

* Use Poetry for dependency management.
* We enforce [black](https://github.com/psf/black) formatting (with the default style).
* Use `pytest` for testing, add tests for your code!
* Use pull requests for both this and the data repository.


## Running pipeline to get web export
Assuming you've installed deps via `poetry install` and you are in the root epimodel repo.
Also, you did `cp config.yaml config-local.yaml` and set `export_regions: [CZ, ES]`

0. clone data repo: `git clone https://github.com/epidemics/epimodel-covid-data data`
1. `./do -C config-local.yaml update_john_hopkins`
2. add a Foretold token into `config-local.yaml` in `foretold_channel` and run `./do -C config-local.yaml update_foretold`
3. **TODO?: Run gleamviz and get batch file? What's being fetched inside the file?** 
4. having the Gleam Batch simulation dir results: `./do -C config-local.yaml import_gleam_batch batch_file`
5.  `./do -C config-local.yaml web_export data/batch-2020-04-03T23-35-24.482054+02-00.hdf5`

### Gleam Batch file
Has two dataframes:
* `simulations`: indexed by SimulationID, contains information about what simulation ID had what parameters
* `new_fraction`: actually contains the modelled data for Infected and Recovered (values/columns). Indexed by `['Code', 'Date', 'SimulationID']`:
    * `Code`: country code (e.g. `AE`)
    * `Date`: a date for which we model Infected and Recovered
    * `SimulationID`: corresponding simulation ID to be able to be able to map it to parameters in `simulations` 
