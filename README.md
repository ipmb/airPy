# airPy
The airPy package was developed to extract high-resolution satellite data from Google Earth Engine and calculate Machine Learning-ready features for air pollution studies.

The code in this repository performs the following tasks:

**1. Download satellite data from Google Earth Engine**
  * Per specified latitude, longitude, and AOI buffer extent, data is downloaded from Google Earth Engine. The download job is fully specified by the user-generated configuration file, which includes all details for the data to be downloaded and processed, including: collection type, latitude/longitude points, analysis period, and buffer size.
  * Data can be saved as an xarray format covering the user-specified AOI extent, or as individual images per latitude, longitude point given.

**2. Generate Machine Learning-ready features**
* Per latitude, longitude point of interest over the user-specified AOI, relevant statistical features are calculated for the following collections: [MODIS Land cover Yearly Product](https://developers.google.com/earth-engine/datasets/catalog/MODIS_061_MCD12Q1#citations), [MODIS Fire_cci Burned Area Pixel Product](https://developers.google.com/earth-engine/datasets/catalog/ESA_CCI_FireCCI_5_1#description), [Gridded Population of the World Version 4.11)](https://developers.google.com/earth-engine/datasets/catalog/CIESIN_GPWv411_GPW_Population_Density), and [VIIRS Nighttime Day/Night Band Composites Version 1](https://developers.google.com/earth-engine/datasets/catalog/NOAA_VIIRS_DNB_MONTHLY_V1_VCMCFG).
    * Features are calculated based on the temporal cadence specified.
    * Where data is unavailable due to lack of satellite coverage, data is set to NaN or filled with specified fill value to avoid calculation errors.

### `airPy` feature extraction
The diagram below depicts an example of the extraction process over the extent of Australia, matching the 11.1x11.1km spatial resolution of the MOMO-Chem (Multi-mOdel Multi-cOnstituent Chemical data assimilation) model output. Per latitude, longitude point on the MOMO-Chem grid:
* Query the GEE dataset of interest
* Create a ~55.5km radius buffer around the point of interest
* Extract the data over the buffer AOI
* Process features of interest (i.e. maximum population per grid point from World Population dataset, percent of each land cover class from MODIS dataset, etc.)

![`airPy` AOI extraction process.](paper/figures/py-aq_all.png)

## Installation
Clone the repo: 
```
git clone git@github.com:kelsdoerksen/airPy.git
```
Navigate to the airPy folder and install the package using `pip`.
```
pip install airpy
```
Install package dependencies using
```
pip install -r requirements.txt  
```


To use the Google Earth Engine API, you must create and authenticate a Google Earth Engine account. Information on the Earth Engine Python API can be found [here](https://developers.google.com/earth-engine/tutorials/community/intro-to-python-api). 

## Creating ML-ready features from GEE data with ``airPy``
#### Running the airPy pipeline
The airPy pipeline job is fully specified by a configration dictionary generated by the `GenerateConfig` class. To create a new configuration and run the pipeline use the command
```
python run_airpy.py
```
and specify the various parameters of the data of interest. The available configurable parameters are:
* `--gee_data`: The name of the GEE dataset of interest, either modis, pop, fire, or nightlight.
    *    modis: MCD12Q1.061 MODIS Land Cover Type Yearly Global 500m, available 2001-01-01 to 2021-01-01
    *    pop: GPWv411: Population Density (Gridded Population of the World Version 4.11), available 2000-01-01 to 2020-01-01
    *    fire: FireCCI51: MODIS Fire_cci Burned Area Pixel Product, Version 5.1, available 2001-01-01 to 2020-12-01
    *    nightlight: VIIRS Nighttime Day/Night Band Composites Version 1, available 2012-04-01 to 2023-01-01
* `--region`: Boundary region on Earth to extract data in (latitudes), (longitudes). Must be one of:
    *   `globe`: (-90, 90),(-180, 180)
    *   `europe` (35, 65),(-10, 25)
    *   `asia`: (20, 50),(100, 145)
    *   `australia`: (-50, -10),(130, 170)
    *   `north_america`: (20, 55),(-125, -70)
    *   `west_europe`: (25, 80),(-20, 10)
    *   `east_europe`: (25, 80),(10, 40)
    *   `west_north_america`: (10, 80),(-140, -95)
    *   `east_north_america`: (10, 80), (-95, -50)
    *   `toar2`: Locations of TOAR2 stations based on TOAR2 metadata
* `--date`: Date of query. Must be in format 'YYYY-MM-DD'
* `--analysis_type`: Type of analysis for data extraction. Must be one of collection, images.
* `--add_time`: Specify if time component should be added to collection xarray. Useful for integrating into time series ML datasets. One of 'y' or 'n'.
* `--buffer_size`: Specify region of interest (ROI) buffer extent. Units in metres.
* `--config-dir`: Specify the output directory for the config file.
* `--save_dir`: Specify run rave directory.
Example:
```
python run_airpy.py --gee_data fire --region australia --date 2020-01-01 --analysis_type collection --buffer_size 55500 --configs_dir /configs --save_dir /runs
```
Generates a config file named `config_australia_fire_2020-01-01_buffersize_55500_collection.json` and kicks of the airpy job.
To look at the help file for more information on paramaters, run the command ```python run_airpy.py --help```.

## Testing
Tests for each script are stored in the `src/tests` folder. `pytest` is used to test scripts via the following command:
```
pytest -q test_<script_name>.py
```
## Citing
If you found this code useful in your research, please cite via the following:

Doerksen, K (2023), airPy: Generating AI-ready datasets in python for air quality studies using Google Earth Engine, github.com/kelsdoerksen/airPy.
