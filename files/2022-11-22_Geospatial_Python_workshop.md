![](https://i.imgur.com/iywjz8s.png)


# 2022-11-22 Geospatial Python workshop


Welcome to The Workshop Collaborative Document.

This Document is synchronized as you type, so that everyone viewing this page sees the same text. This allows you to collaborate seamlessly on documents.

----------------------------------------------------------------------------

This is the Document for today: [link](https://tinyurl.com/gsw20221122)

Collaborative Document day 1: [link](https://tinyurl.com/gsw20221121)

Collaborative Document day 2: [link](https://tinyurl.com/gsw20221122)

Collaborative Document day 3: [link](https://tinyurl.com/gsw20221123)


## 👮Code of Conduct

Participants are expected to follow these guidelines:
* Use welcoming and inclusive language.
* Be respectful of different viewpoints and experiences.
* Gracefully accept constructive criticism.
* Focus on what is best for the community.
* Show courtesy and respect towards other community members.
 
## 🎓 Certificate of attendance

If you attend the full workshop you can request a certificate of attendance by emailing to training@esciencecenter.nl .

## ⚖️ License

All content is publicly available under the Creative Commons Attribution License: [creativecommons.org/licenses/by/4.0/](https://creativecommons.org/licenses/by/4.0/).

## 🙋Getting help

To ask a question, raise your hand in zoom. Click on the icon labeled "Reactions" in the toolbar on the bottom center of your screen,
then click the button 'Raise Hand ✋'. For urgent questions, just unmute and speak up!

You can also ask questions or type 'I need help' in the chat window and helpers will try to help you.
Please note it is not necessary to monitor the chat - the helpers will make sure that relevant questions are addressed in a plenary way.
(By the way, off-topic questions will still be answered in the chat)


## 🖥 Website for setup and data

🛠 Setup: [link](carpentries-incubator.github.io/geospatial-python/setup.html)

📚 Download files:
- [brpgewaspercelen_definitief_2020_small.gpkg](https://figshare.com/ndownloader/files/37729413)
- [brogmwvolledigeset.zip](https://figshare.com/ndownloader/files/37729416)
- [status_vaarweg.zip](https://figshare.com/ndownloader/files/37729419)

Satellite imagery search result:
- [search.json](https://raw.githubusercontent.com/esciencecenter-digital-skills/2022-11-21-dc-geospatial/main/files/search.json)

## 👩‍🏫👩‍💻🎓 Instructors

Francesco Nattino, Ou Ku

## 🧑‍🙋 Helpers

Djura Smits, Pranav Chandramouli


## 🗓️ Agenda
| Time | Topic |
|--:|:---|
|09:00 |  Welcome and icebreaker|
|09:15 |  Read and visualize raster data| 
|10:30 |  Coffee break| 
|10:45 |  Vector data in Python| 
|11:30 |  Coffee break| 
|11:45 |  Crop raster data with rioxarray and geopandas| 
|12:30 |  Wrap-up| 
|13:00 |  END| 

## 🔧 Exercises

#### Exercise: find the axes units of the CRS

What units are our data in? See if you can find a method to examine this information using `help(crs)` or `dir(crs)`


#### Exercise: set the plotting aspect ratio

Plotting a satellite image can lead to a "stretched" image. Let’s visualize it with the right aspect ratio. You can use the [documentation](https://xarray.pydata.org/en/stable/generated/xarray.DataArray.plot.imshow.html) of `DataArray.plot.imshow()`.



#### Exercise: Import Line and Point Vector Datasets
Using the steps above, load the waterways and groundwater well vector datasets (`data/status_vaarweg.zip` and `data/brogmwvolledigeset.zip`, respectively) into Python using `geopandas`. Name your variables `waterways_nl` and `wells_nl` respectively.

Answer the following questions:

- What type of spatial features (points, lines, polygons) are present in each dataset?
- What is the CRS, give the EPSG ID.
- What is the total extent (bounds) of each dataset?
- How many spatial features are present in each file?


How to get the answers for the waterways:
```python=
waterways = gpd.read_file("data/status_vaarweg.zip")
```
```python=
waterways.type.unique()
```
```python=
waterways.crs
```
```python=
waterways.total_bounds
```



## 🧠 Collaborative Notes
### Read and visualize raster data
To prepare:
```python=
import rioxarray
import pystac
```

```python=
items = pystac.ItemCollection.from_file("search.json")
items
```

```python=
href = items[0].assets["B09"].get_absolute_href()
href
```

```python=
raster_ams_b9 = rioxarray.open_rasterio(href)
```

Plot it:
```python
raster_ams_b9.plot()
```

Inspect the DataArray:
```python
raster_ams_b9
```

Coordinate reference system:
```python
raster_ams_b9.rio.crs
```

Value to encode missing value
```python
raster_ams_b9.rio.nodata
```

Bounding box:
```python
raster_ams_b9.rio.bounds()
```

Show the values, data will be loaded in memory the first time you run this:
```python
raster_ams_b9.values
```

Plotting will need the actual data, first time you plot the data will be loaded in memory.
```python
raster_ams_b9.plot()
```
![](https://i.imgur.com/G9vEzeF.png)


Fix the contrast:
```python
raster_ams_b9.plot(robust=True)
```
![](https://i.imgur.com/XjHhdtW.png)

### View Raster Coordinate Reference System (CRS) in python
Get the EPSG code number:
```python
epsg = raster_ams_b9.rio.crs.to_epsg()
```

Get more information about this CRS:
```python=
import pyproj
```

```python=
crs = pyproj.CRS(epsg)
```

```python=
crs.area_of_use
```

To check the units of our axes:
```python=
crs.axis_info
```

### Calculating raster statistics
Minimum value:
```python=
raster_ams_b9.min()
```

Other interesting statistics
```python
raster_ams_b9.max()
raster_ams_b9.mean()
raster_ams_b9.std()
```

You can also use (some)numpy functions on dataarrays and get a data array back:
```python
import numpy as np
np.min(raster_ams_b9)
```

Mask the nodata values, they will be represented as `numpy.nan` now. Integers will be converted to floats:
```python
raster_ams_b9_nodata = rioxarray.open_rasterio(href, masked=True)
raster_ams_b9_nodata
```



Plot our masked array
```python=
raster_ams_b9_nodata.plot()
```

Check the statistics again. They have changed because the 0 values are not taken into account anymore:
```python
raster_ams_b9_nodata.min()
raster_ams_b9_nodata.mean()
```

Note that the following is a lot more typing (but avoids conversion to float): `raster_ams_b9.data[raster_ams_b9.data!=raster_ams_b9._FillValue].min()`


Let's inspect the "overview" asset
```python=
overview_href = items[0].assets["overview"].get_absolute_href()
```

```python=
overview = rioxarray.open_rasterio(overview_href)
overview
```

Like numpy, you can inspect the shape of the data array
```python=
overview.shape
```

The plot will look different because it has multiple layers:
```python=
overview.plot()
```
![](https://i.imgur.com/DlyVS8o.png)


To display it as an image:
```python
overview.plot.imshow()
```
![](https://i.imgur.com/ZwQhHkg.png)

### Break until 10:48!

### Vector data in Python
#### Data
Make sure you have the data:
📚 Download files:
- [brpgewaspercelen_definitief_2020_small.gpkg](https://figshare.com/ndownloader/files/37729413)
- [brogmwvolledigeset.zip](https://figshare.com/ndownloader/files/37729416)
- [status_vaarweg.zip](https://figshare.com/ndownloader/files/37729419)

Satellite imagery search result:
- [search.json](https://raw.githubusercontent.com/esciencecenter-digital-skills/2022-11-21-dc-geospatial/main/files/search.json)

Let's start by importing geopandas
```python=
import geopandas as gpd
```

Load our data
```python=
cropfield = gpd.read_file("data/brpgewaspercelen_definitief_2020_small.gpkg")
```

Let's view it:
```python=
cropfield
```

Check documentation of `gpd.read_file`
```python=
gpd.read_file?
```

Let's pretend the file is big and investigate only part of it:
```python=
cropfield = gpd.read_file("data/brpgewaspercelen_definitief_2020_small.gpkg", rows=10)

cropfield
```

Use bbox argument to only load data that is in the same area as our previous sattelite data

Inspect coordinate system:
```python=
cropfield.crs
```

```python=
minx, miny, maxx, maxy = (110_000, 475_000, 140_000, 500_000)
bbox = (minx, miny, maxx, maxy)

bbox
```

Load the data one more time, with limited extent this time:
```python=
cropfield = gpd.read_file("data/brpgewaspercelen_definitief_2020_small.gpkg", bbox=bbox)
cropfield
```

Load directly from url `https://figshare.com/ndownloader/files/3772941`3 :
```python=
test_data = gpd.read_file("https://figshare.com/ndownloader/files/37729413")
```

Check the type of our cropfield data (spoiler alert: it's a `GeoDataFrame`)
```python=
type(cropfield)
```

Check the type of your geometry column:
```python=
cropfield.type
```

Check all the unique types in your geometry column:
```python=
cropfield.type.unique()
```

Check crs again
```python=
cropfield.crs
```

Check bounding box of your GeoDataFrame:
```python=
cropfield.total_bounds
```

```python=
cropfield.plot()
```
![](https://i.imgur.com/TuOs1LH.png)

Select a smaller area using slicing
```python=
xmin, xmax = (120_000, 135_000)
ymin, ymax = (485_000, 500_000)
cropfield = cropfield.cx[xmin:xmax, ymin:ymax]
```

plot our smaller area
```python=
cropfield.plot()
```
![](https://i.imgur.com/YSFZHHx.png)


Save our selected data to disk as a shapefile:
```python=
cropfield.to_file("cropfield.shp")
```

### :coffee: :tea: Break until 11:50! 


```python=
waterways = gpd.read_file("data/status_vaarweg.zip")
waterways.type.unique()
waterways.crs
waterways.total_bounds
```

Visualize waterways:
```python
waterways.plot()
```
![](https://i.imgur.com/8u1lhP0.png)

Inspect coordinates of waterways
```python
print(waterways["geometry"][0])
```

Inspect coordinates of cropfields
```python
print(cropfield["geometry"][0])
```

Flipping x and y coordinates:
```python
import shapely
```
```python
def flip(geometry):
    return shapely.ops.transform( lambda x, y: (y, x), geometry)
```
```python
type(waterways)
```

```python
waterways["geometry"] = waterways["geometry"].apply(flip)
waterways.plot()
```
![](https://i.imgur.com/AsU78YD.png)

Save corrected waterways to shp file:
```python
waterways.to_file("waterways_corrected.shp")
```

### Crop raster
Import the necessary libraries
```python
import pystac
import rioxarray
```

```python
items = pystac.ItemCollection.from_file("search.json")
items
```

Pick the second item:
```python
item = items[1]
item
```

inspect the assets:
```python
item.assets
```

Load the metadata of the "visual" asset
```python
item_href = item.assets["visual"].href
true_color_image = rioxarray.open_rasterio(item_href)
true_color_image
```

## Tips/Tops

### :-1: Tip: what to improve
- Spend a bit less time of Python generics 
- These are very useful tools and more could hvae been shown about their specific features, rather than spending lots of time on simple python executions like reading and slicing
- share all links of your pages beforehand (I meant the pages only)
- For me, I would like a bit more introduction/overview of the benefits of processing geodata with Python compared to say ArcMap or QGIS.
- Maybe spending a bit less time on recaps might help having more time for later explanations
- A lot of time is spend on python aspects of handling data, where Python knowledge was a prerequisite for this workshop?
- Not sure how to handle the different coord systems...
- Can you give some small projects for exercising and future l

### :+1: Top: what went well
- The example of amending incorrect coordinates was really insightful
- Nice examples, and nice trouble-shooting
- Learn a lot, nice exercises and examples
- Great workshop format - I would definetly sign up for another
- Plenty of room for questions, elaboration, and problem-solving. Good job! (x2)
- I liked the reading file
- Very helpful and illustrative workshop


## 📚 Resources

- [GPKG data format](https://docs.fileformat.com/gis/gpkg/)
- [rd viewer](https://openstate.github.io/rdnewviewer/)
