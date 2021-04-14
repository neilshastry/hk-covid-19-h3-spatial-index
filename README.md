# HK Covid-19 dynamic visualization of cases using Uber H3 Spatial Index

## Background
Personal project to build a spatial index visualization to quickly identify Covid-19 hotspots in Hong Kong based on HK Gov health data and plan my daily outdoor movements to avoid districts and buildings with high covid cases in the last 14 days.

## Objectives
Visualize neighbourhoods with overlayed h3 polygons on districts and buildings with covid-19 cases based on intensity.

## Tools
[![made-with-python](https://img.shields.io/badge/Made%20with-Python-1f425f.svg)](https://www.python.org/)
[![Made withJupyter](https://img.shields.io/badge/Made%20with-Jupyter-orange?style=for-the-badge&logo=Jupyter)](https://jupyter.org/try)

## Table of Contents
#### [What is H3](##What-is-H3)
#### [Dataset](##Dataset)
#### [Data Wrangling](#Data-Wrangling)
#### [Functions for H3 Mapping](#Functions-for-H3-Mapping)
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

We set up a dynamic date counter to pull the most recent from the API without manual intervention.

```
from datetime import date, timedelta

today = date.today()-timedelta(1)
d = today.strftime("%Y%m%d")
print("d1 =", d)

result=requests.get("https://api.data.gov.hk/v1/historical-archive/get-file?url=http%3A%2F%2Fwww.chp.gov.hk%2Ffiles%2Fmisc%2Fbuilding_list_eng.csv&time=" + d + "-0906")
result = result.content
data = pd.read_csv(io.StringIO(result.decode('utf-8')))

```

![Screenshot 2021-04-14 at 6 16 15 PM](https://user-images.githubusercontent.com/36125669/114698756-0adbfb00-9d52-11eb-9be0-e52ddd333737.jpeg)



## Functions for H3 Mapping



## Python Visualization


## Author
Neil Shastry

## Acknowledgments

