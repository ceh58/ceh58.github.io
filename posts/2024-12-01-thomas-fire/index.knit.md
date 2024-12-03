---
title: "Using False Color Imagery to Visualize the Impact of the Thomas Fire in Python"
#description: "blog post description (appears underneath the title in smaller text) which is included on the listing page"
author:
  - name: Carmen Hoyt
    url: https://ceh58.github.io/
    #orcid: 0000-0002-5300-3075
    affiliation: Master of Environmental Data Science Program @ The Bren School (UCSB)
    affiliation-url: https://ucsb-meds.github.io/ 
date: last-modified
categories: [Python, EDS220, Landsat] # self-defined categories
toc: true
#citation: 
 # url: https://ceh58.github.io/posts/2024-12-01-thomas-fire/ 
image: thomas-fire.jpg
draft: false # setting this to `true` will prevent your post from appearing on your listing page until you're ready!
---

::: {.cell}
::: {.cell-output-display}
![](thomas-fire.jpg)
:::
:::



# About

The Thomas Fire burned over 280,000 acres (about 440 square miles) across Ventura and Santa Barbara counties in December 2017, the largest wildfire in modern California history at the time. The main catalyst for the fire's rapid spread was unseasonably strong Santa Ana wind that brought warm air and low humidity. In the end, 1,063 structures were lost, over 104,607 residents were forced to leave their homes, and damages totaled over $2.2 billion. Lasting environmental effects of the fire included poor air quality and mudflows during the successive rainy season as a result of the vegetation loss[^1].

[^1]: Read more about the Thomas fire [here](https://en.wikipedia.org/wiki/Thomas_Fire).

This analysis uses imagery taken by Landsat 8 on January 16, 2018 to highlight the burn scar left by the Thomas Fire after it was considered fully contained (January 12, 2018). By assigning infrared bands to visible colors (short wave infrared to 'red', near infrared to 'green', and red to 'blue'), we can easily distinguish the burn scar from the surrounding vegetation. Bare earth/dead vegetation reflects swir (short wave infrared), appearing red, and healthy vegetation reflects nir (near infrared), appearing green, in the false color image[^2].

[^2]: Read more about false color imagery [here](https://earthobservatory.nasa.gov/features/FalseColor).

*This analysis was part of EDS 220: Working with Environmental Datasets - Homework Assignment 4 [^3].*

[^3]: See the assignment guidelines [here](https://meds-eds-220.github.io/MEDS-eds-220-course/assignments/assignment4.html).

# Highlights

- This task explores assigning infrared bands to visible colors to obtain false color imagery.

- Necessary steps include cleaning rasters with the `rioxarray` package as well as filtering geo-dataframes with `geopandas` package.

- It is essential to match the Coordinate Reference Systems (CRSs) of shapefiles and rasters to obtain the final figure.

# Repository 

More detailed information can be found on my [Thomas Fire GitHub Repository](https://github.com/ceh58/eds220-hwk4-repeat).

#### Repository structure:

```
├── data
│  ├── thomas_fire.cpg
│  ├── thoams_fire.dbf
│  ├── thomas_fire.prj
│  ├── thomas_fire.shp
│  └── thomas_fire.shx
├── .gitignore
├── README.md
├── hwk4-task2-false-color-HOYT.ipynb
└── hwk4-task2-fire-perimeter-HOYT.ipynb
```

# Dataset Descriptions

Landsat Data:

The landsat dataset used in this analysis is a cleaned, simplified collection of bands (red, green, blue, nir, swir) from [Landsat Collection 2 Level-2](https://planetarycomputer.microsoft.com/dataset/landsat-c2-l2) (collected by Landsat 8 satellite) that was prepared specifically for this project. 

Fire Perimeters Data:

The [fire perimeters dataset](https://catalog.data.gov/dataset/california-fire-perimeters-all-b3436) is an open-source dataset that contains information about the spatial distrubtion of past fires in California published by the State of California (and downloaded as a shapefile). 

# Analysis

#### Part 1

Derived from `hwk4-task2-fire-perimeter-HOYT.ipynb`.

First, import all necessary packages.

```python
import os
import pandas as pd
import geopandas as gpd
import xarray as xr
```

Then, import the fire perimeters dataset (shapefile) and filter for the 2017 Thomas Fire. Save the filtered dataset in a format of your choice. I chose to save it as a shapefile due to its versatility and familiarity.

*Note: I saved the full fire perimeters dataset in my data/ folder in a separate no_push/ folder that was added to my .gitignore due to the size of the data.*

```python
# Create filepath
fp = os.path.join("data", "no_push", "California_Fire_Perimeters_(all).shp")

# Read in data
fire_perimeter = gpd.read_file(fp)

# Lower column names
fire_perimeter.rename(columns=str.lower, inplace=True)

# Select Thomas Fire boundary by filtering for name and year
thomas_fire = fire_perimeter.loc[(fire_perimeter['fire_name'] == "THOMAS") & 
                                 (fire_perimeter['year_']== 2017)]
                                 
# Save Thomas Fire boundary as a shapefile
thomas_fire.to_file(os.path.join("data", "thomas_fire.shp"))
```
#### Part 2

Derived from `hwk4-task2-false-color-HOYT.ipynb`.

First, import all necessary packages.

```python
import os
import pandas as pd
import matplotlib.pyplot as plt
import geopandas as gpd
import xarray as xr
import rioxarray as rioxr
import numpy as np
```

Next, import the landsat data (which has been pre-processed and saved on the server at the given filepath: `/courses/EDS220/data/hwk4_landsat_data", "landsat8-2018-01-26-sb-simplified.nc`).

```python
# Import data
fp = os.path.join("/courses/EDS220/data/hwk4_landsat_data", "landsat8-2018-01-26-sb-simplified.nc")
landsat = rioxr.open_rasterio(fp)
landsat
```



::: {.cell}
::: {.cell-output-display}
![](landsat_xarray.png){width=829}
:::
:::



Notice that the raster has a dimension, `band`, of size one. This dimension is not necessary, so we will use the `squeeze()` and `drop_vars()` functions to remove it.

```python
# Drop the 'band' dimension
landsat = landsat.squeeze().drop_vars('band')

# Confirm 'band' was dropped
print(landsat.dims, landsat.coords)
```

Confirm that `band` no longer appears on the list of dimensions (`landsat.dims`).



::: {.cell}
::: {.cell-output-display}
![](drop_band.png){width=685}
:::
:::



Now we can plot a true color image. To do this, we must select the 'red', 'green', and 'blue' bands, in that order, and assign them to the 'red', 'green', and 'blue' colors using `.imshow()`.

```python
# Select 'red', 'green', and 'blue' variables and plot
landsat[['red', 'green', 'blue']].to_array().plot.imshow()
```

Since there are outliers in these data, the initial plot is black and white and gives us the following warning message:



::: {.cell}
::: {.cell-output-display}
![](true_color_1.png){width=749}
:::
:::



In order to de-weight the outliers and properly scale each band, we will set the `robust` parameter in `.imshow()` to `True`.

```python
# Adjust the scale for a true color plot
landsat[['red', 'green', 'blue']].to_array().plot.imshow(robust = True)
```

This produces our true color image:



::: {.cell}
::: {.cell-output-display}
![](true_color_2.png){width=708}
:::
:::



To create our false color image, we must assign the short wave infrared band ('swir22') to the 'red' color, the near infrared band ('nir08') to the 'green' color, and 'red' band to the 'blue' color using the same function.

```python
# Create a false color image
landsat[['swir22', 'nir08', 'red']].to_array().plot.imshow(robust = True)
```

The result is our false color image:



::: {.cell}
::: {.cell-output-display}
![](false_color.png){width=727}
:::
:::



Finally, we can create our figure.

In order to do this, we must import the Thomas Fire perimeter shapefile we previously saved in Part 1, `thomas_fire.shp`, and check to see that the CRS of the shapefile matches that of the landsat data using `.crs` (from the `geopandas` package) for the shapefile and `.rio.crs` (from the `rioxarray` package) for the raster. 

```python
# Import Thomas Fire shapefile
thomas_fire = gpd.read_file(os.path.join("data", "thomas_fire.shp"))

# Make sure CRSs match
if thomas_fire.crs == landsat.rio.crs:
    print("CRSs match!")
else:
    landsat = landsat.rio.reproject(thomas_fire.crs)
    assert landsat.rio.crs == thomas_fire.crs
    print("We matched the CRSs!")
```



::: {.cell}
::: {.cell-output-display}
![](message.png){width=188}
:::
:::



To plot the image, we must create an aspect ratio to correctly display the size. The aspect ratio is the width/height. 

```python
# Map the false color image with the fire perimeter
landsat_aspect_ratio = landsat.rio.width/landsat.rio.height
```

Then, the figure is set up using the aspect ratio, and each figure element is plotted in sequence using the `matplotlib` package. 

```python
# Setup figure
fig, ax = plt.subplots(figsize = (6, 6*landsat_aspect_ratio))

# Turn the axis off
ax.axis("off")

# Plot the false color image on the figure
landsat[['swir22', 'nir08', 'red']].to_array().plot.imshow(ax = ax,
                                                        robust = True)

# Add Thomas Fire shapefile as a boundary on the figure
thomas_fire.boundary.plot(ax = ax,
                         color = "black")

# Add legend to the figure
ax.legend(labels = ["Fire Boundary"])

# Add annotation to the figure
fig.text(0.5, 0.1,
        'Data Source: CAL FIRE via Data.gov &  Microsof Planetary Computer data catalogue',
         ha='center', va='center', fontsize=8, color='black', fontstyle='italic')

fig.text(0.395, 0.08, 
         'Date Accessed: 11/19/24',
         ha='right', va='center', fontsize=8, color='black', fontstyle='italic')

# Add title
ax.set_title("Thomas Fire Scar (2017)", fontsize=14, fontweight='bold')

plt.show()
```

Our final figure shows the burn scar of the Thomas Fire, displayed in red and outlined by the fire boundary. 



::: {.cell}
::: {.cell-output-display}
![](final_plot.png){width=555}
:::
:::



# References

Landsat data:

Microsoft Open Source, Matt McFarland, Rob Emanuele, Dan Morris, & Tom Augspurger. (2022). microsoft/PlanetaryComputer: October 2022 (2022.10.28). Zenodo. [https://doi.org/10.5281/zenodo.7261897](https://doi.org/10.5281/zenodo.7261897) Accessed: November 19, 2024

Fire perimeter data:

State of California, Kimberly Wallin. (2024). CAL FIRE: May 2024 (2024.05.14). [https://catalog.data.gov/dataset/california-fire-perimeters-all-b3436](https://catalog.data.gov/dataset/california-fire-perimeters-all-b3436) Accessed: November 19, 2024
