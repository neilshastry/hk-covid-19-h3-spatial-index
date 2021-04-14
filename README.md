# HK Covid-19 dynamic visualization of cases using Uber H3 Spatial Index

## Background
Personal project to build a spatial index visualization to quickly identify Covid-19 hotspots in Hong Kong based on HK Gov health data and plan my daily outdoor movements to avoid districts and buildings with high covid cases in the last 14 days.

## Objectives
Visualize neighbourhoods with overlayed h3 polygons on districts and buildings with covid-19 cases based on intensity.

## Tools
[![made-with-python](https://img.shields.io/badge/Made%20with-Python-1f425f.svg)](https://www.python.org/)
[![Made withJupyter](https://img.shields.io/badge/Made%20with-Jupyter-orange?style=for-the-badge&logo=Jupyter)](https://jupyter.org/try)

## Table of Contents
#### [What is H3](#What-is-H3)
#### [Dataset](#Dataset)
#### [Data Wrangling](#Data-Wrangling)
#### [H3 Modules](#Functions-for-H3-Mapping)
#### [Python Visualization](#Python-Visualization)

## What is H3

[![Open Source Love svg1](https://badges.frapsoft.com/os/v1/open-source.svg?v=103)](https://eng.uber.com/category/articles/open-source-articles/)

Uber H3 is a grid system for efficiently optimizing ride pricing and dispatch for visualizing and exploring spatial data. H3 enables Uber to analyze geographic information to set dynamic prices and make other decisions on a city-wide level. They use H3 as the grid system for analysis and optimization throughout marketplaces. It provides functions for converting latitude/longitude coordinates into H3 geospatial hexagonal tiles. 

It is written in C but there are bindings available for other languages.

![Screenshot 2021-04-14 at 12 47 03 PM](https://user-images.githubusercontent.com/36125669/114655955-d4858800-9d1f-11eb-8d6c-247f88c5a782.jpeg)
**Source:** (https://eng.uber.com/h3/)

This powerful tool to visualize spatial data is available as open source [Github](https://uber.github.io/h3/)


## Dataset

The Hong Kong Government has a [website](https://data.gov.hk/en-data/dataset/hk-dh-chpsebcddr-novel-infectious-agent) which publishes excellent and transparent data for probable / confirmed covid-19 patients in both English and Chinese. The main datasets are published and updated with a one day lag and include:
1. Residential and commerical buildings lived / visited in the prior 14 days
2. Details on cases and hospitalization
3. Latest situation of reported cases
4. Flights / Trains / Ships / Taxis taken by confirmed cases

For our purpose, we will limit the use case for our dataset to plot the spatial data based on residential and commercial buildings visited or lived in by Covid-19 patients within the last 14 days using the English language dataset.

The API for the English version of this data is available [here](https://data.gov.hk/en-data/dataset/hk-dh-chpsebcddr-novel-infectious-agent/resource/cf287e7e-cb41-4421-b9d1-068d3a3b61c0).

![Screenshot 2021-04-14 at 1 33 34 PM](https://user-images.githubusercontent.com/36125669/114659416-27623e00-9d26-11eb-9182-ed9b7db4fc99.jpeg)

## Data Wrangling

This section covers brief highlights on importing the data and then the process followed to convert the data to a format we will continue to use throughout the project.

#### 1. Importing Data

**Input:** Raw Data for Residential and Commercial buildings with probable / confirmed cases
![Screenshot 2021-04-14 at 6 16 15 PM](https://user-images.githubusercontent.com/36125669/114698966-4971b580-9d52-11eb-8bb7-7bda49e312b5.jpeg)

We set up a dynamic date counter to pull the most recent data from the API without manual intervention.

```
from datetime import date, timedelta

today = date.today()-timedelta(1)
d = today.strftime("%Y%m%d")
print("d1 =", d)

result=requests.get("https://api.data.gov.hk/v1/historical-archive/get-file?url=http%3A%2F%2Fwww.chp.gov.hk%2Ffiles%2Fmisc%2Fbuilding_list_eng.csv&time=" + d + "-0906")
result = result.content
data = pd.read_csv(io.StringIO(result.decode('utf-8')))

```

**Output:** Pandas dataframe based on imported raw data
![Screenshot 2021-04-14 at 6 53 04 PM](https://user-images.githubusercontent.com/36125669/114699481-f2b8ab80-9d52-11eb-9fdd-99ab10ec4fb1.jpeg)

#### 2. Mapping the Latitude and Longitude

In order to leverage the H3 spatial index we need to map the latitude and longitude of the districts and building names leveraging the geopy library in python. This will provide us the coordinates we can then use to create the data structure required for visualizing hexagons. 

**Output 1:**
Mapped latitude and longitude coordinates. Some missing values present for building names that were unidentified.
<img width="993" alt="Map Lat and Long 1" src="https://user-images.githubusercontent.com/36125669/114703458-e4b95980-9d57-11eb-9055-d4812f4a9660.png">

**Output 2:**
Populating the missing building coordinate data with the district coorindates as a proxy.
<img width="988" alt="Map Lat and Long 2" src="https://user-images.githubusercontent.com/36125669/114703531-01ee2800-9d58-11eb-8dd4-f0c19c528705.png">

This coordinate mapping now allows us to use the H3 spatial index libraries to convert coordinates to hexagonal tiles over Hong Kong.

## H3 Modules

There are three critical mappings we now perform with the H3 modules.

#### 1. geo_to_h3
The geo_to_h3 module helps us convert our latitude and longitude coordinates to a Hex index reference. This Hex reference has a 'resolution' parameter that allows us the flexibly to choose the level of granularity between 1 (widest) to 15 (finest). A resolution of 9 is roughly equivalent to plotting a hexagon over an city block.

#### 2. h3_get_resolution
The h3_get_resolution module allows us to move up or down in granularity based on the resolution parameter previously defined to change the index reference based on the value between 1 to 15.

#### 3. h3_to_geo_boundary
The h3_to_geo_boundary module essentially helps us define our hexagon shape (i.e. Polygon) and map the vertices for the Hex index based on the coordinates that border the Hex index reference that we defined with the geo_to_h3 module.

**Output:** The combined output after running our dataset over both modules looks something like this with a value parameter that counts the number of times that hex id is referenced.
<img width="430" alt="Hex ID" src="https://user-images.githubusercontent.com/36125669/114706631-c6edf380-9d5b-11eb-8432-f23380ad4ff5.png">

## Python Visualization
Our final section covers the mapping of the hex reference shape and coordinate data over a map of Hong Kong.
For our use case, we leverage ['© OpenStreetMap contributors'](https://www.openstreetmap.org/copyright).

We can build a choropleth map by converting our data with the hexagonal references in our previous step into a GeoJson file format by calling the Folium library. Folium is a Python library used for visualizing geospatial data. Folium is a Python wrapper for Leaflet.js which is a leading open-source JavaScript library for plotting interactive maps.

Finally, we plot our outputs at two levels: district with wider hexagons; buildings with finer hexagons.

**Output: District Level**
<img width="987" alt="District Map" src="https://user-images.githubusercontent.com/36125669/114707038-54314800-9d5c-11eb-996b-53101ca7b646.png">

**Output: Building Level**
<img width="988" alt="Building Map" src="https://user-images.githubusercontent.com/36125669/114707060-598e9280-9d5c-11eb-9662-5395ecdbf3fc.png">


## Author
Neil Shastry

## Acknowledgments
I would sincerely like to acknowledge the references and inspirations for this project across a wide range of sources.
1. The [HK Government](https://data.gov.hk/en/) for providing excellent and quality data and making it available for the public to use.
2. Uber for building and open sourcing such a wonderful library and [information](https://eng.uber.com/h3/) for us to reference in our visualization.
3. Abdullah Kurkcu's Better programming [article](https://betterprogramming.pub/playing-with-ubers-hexagonal-hierarchical-spatial-index-h3-ed8d5cd7739d) was indeed the biggest reference point and guide to form the baseline and scope for our work.
4. Abhishek Sharma [piece](https://www.analyticsvidhya.com/blog/2020/06/guide-geospatial-analysis-folium-python/) in Analytics Vidhya.
