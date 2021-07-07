




<br /><br /><br /><br /><br /><br />

# 	 <p align=center> Geosnap Tutorial 
#### <p align=center> Stephen Francisco, Kate Oxx
<p align=center> 04/14/2021

<br /><br /><br /><br />

<p align=left>
This tutorial will showcase tools from the 
<a href=https://spatialucr.github.io/geosnap-guide/content/home.html>Geosnap</a> 
package for Python. Geosnap helps spatial analysts identify, explore, and model neighborhoods statically or 
temporally based on parameters chosen by the user. It contains built-in data from the past three censuses 
(1990, 2000, and 2010) as well as annual census-block level data from the Longitudinal Employment-Household 
Dynamics (LEHD) program, but also allows the user to import their own data.  


In this tutorial we will cover boundary harmonization
for standardizing geographic units over time, geodemographics and regionalization, and 
simulating neighborhood change &ndash; just part of the functionality of the Geosnap package.

## Installation

The makers of Geosnap recommend installing with anaconda. Open the Miniconda prompt, navigate to 
the proper environment, and install using the following commands:

		conda activate <your_env>
		conda env create -f environment.yml

To install geosnap in its own conda environment and install the appropriate dependencies,
 add the following two commands:
 
		conda activate geosnap 
		python setup.py develop

## Built-In Data





First, we will import geosnap's central data structure: **Community**
   >"A Community is a dataset that stores information about a collection of 
   neighborhoods over several time periods, including each neighborhood’s 
  physical, socioeconomic, and demographic attributes and its demarcated 
   boundaries. Under the hood, each Community is simply a long-form geopandas 
   geodataframe with some associated metadata."<sup>1</sup>
	
		from geosnap import Community

Processing times can be fairly long when working with large datasets (such as large cities or entire states).
Therefore, we recommend storing the census data locally to improve performance using the following commands:

		from geosnap.io import store_census
		store_census()

To harness the census data built into Geosnap, we will have to assign it to a variable, thus creating our 
own Community. In this case, we will use the 
### use 'from_census' to obtain census data for a desired county using fips codes
phl = Community.from_census(county_fips="42101")

### to access the underlying data, call its gdf attribute to return a geodataframe
### call 'head' module?? to see the first five lines of the gdf 
phl.gdf.head()

### to view all column names, iterate and print each column name
columns = phl.gdf.columns
for col in columns:
    print (col)

### set a reasonable coordinate reference system for the area of focus. in this 
### case, we'll use epsg (European Petroleum Survey Group) 6246 
phl.gdf = phl.gdf.to_crs(2272)



## Boundary Harmonization


### create a time series showing the median rent of each census tract in the county
### cmap determines the colormap for the map
### figsize determines the size (width,height) of the plot in inches (matplotlib)
### HAD TO INSTALL DESCARTES: 'conda install -c conda-forge descartes'
phl.plot_timeseries(
    "p_vacant_housing_units", title="Percent Vacant Housing, Original Boundaries",
    cmap='Reds',
    dpi=200,
    edgecolor = 'black',
    alpha = 0.7,
    figsize=(14,5)
)

### harmonize boundaries using areal interpolation. this means that we use the 
### data from the area of overlap between consecutive time periods to add to 
### existing data for harmonized polygons
### must distinguish between intensive (magnitude is independent of the size
### of the system, aka mean, median, temperature, etc) and extensive (add to or 
### subtract from the existing system) variables 
phl_harm10 = phl.harmonize(
    intensive_variables=["p_vacant_housing_units"],
    extensive_variables=["n_total_pop"],
    weights_method="area",
    target_year=2010,
)

### plot resulting maps

phl_harm10.plot_timeseries(
    "p_vacant_housing_units", title="Percent Vacant Housing (%), 2010 Boundaries",
    dpi=200,
    edgecolor = 'black',
    alpha = 0.7,
    figsize=(14,5)
)

### use 1990 boundaries to see how demographics have changed using those units

phl_harm90 = phl.harmonize(
    intensive_variables=["p_vacant_housing_units"],
    weights_method="area",
    target_year=1990,
)

phl_harm90.plot_timeseries(
    "p_vacant_housing_units", 
    title="Percent Vacant Housing, 1990 Boundaries",
    cmap='Greens',
    dpi=200,
    edgecolor = 'black',
    alpha = 0.7,
    figsize=(14,5)
)

### harmonize boundaries using dasymetric interpolation

import quilt3

quilt3.Package.browse("rasters/nlcd", "s3://spatial-ucr")["nlcd_2001.tif"].fetch(dest="./")

phl_harm10_dasy = phl.harmonize(
    intensive_variables=["p_vacant_housing_units"],
    weights_method="dasymetric",
    raster='nlcd_2001.tif',
    codes=[21, 22, 23, 24],
    target_year=2010,
)

phl_harm10_dasy.plot_timeseries(
    "p_vacant_housing_units", title="Percent Vacant Housing, 2010 Boundaries w/ Dasymetric Interpolation",
    dpi=200,
    edgecolor = 'black',
    alpha = 0.7,
    cmap='Greens',
    figsize=(14,5)
)





## References

1https://spatialucr.github.io/geosnap-guide/content/02_creating_community_datasets.html