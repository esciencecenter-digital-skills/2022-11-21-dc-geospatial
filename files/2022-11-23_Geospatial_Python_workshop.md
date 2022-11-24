![](https://i.imgur.com/iywjz8s.png)


# 2022-11-23 Geospatial Python workshop


Welcome to The Workshop Collaborative Document.

This Document is synchronized as you type, so that everyone viewing this page sees the same text. This allows you to collaborate seamlessly on documents.

----------------------------------------------------------------------------

This is the Document for today: [link](https://tinyurl.com/gsw20221123)

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

Other files produced during previous days:
- Satellite imagery search result: [search.json](https://raw.githubusercontent.com/esciencecenter-digital-skills/2022-11-21-dc-geospatial/main/files/search.json)
- Crop fields: [cropfield.tgz](https://raw.githubusercontent.com/esciencecenter-digital-skills/2022-11-21-dc-geospatial/main/files/cropfield.tgz) 

## 👩‍🏫👩‍💻🎓 Instructors

Francesco Nattino, Ou Ku

## 🧑‍🙋 Helpers

Djura Smits, Pranav Chandramouli


## 🗓️ Agenda
| Time | Topic |
|--:|:---|
|09:00 | Welcome and icebreaker| 
|09:15 | Crop raster data with rioxarray and geopandas|
|10:15 | Coffee break| 
|10:30 | Raster Calculations in Python| 
|11:15 | Coffee break| 
|11:30 | Calculating Zonal Statistics on Rasters / Parallel raster computations using Dask| 
|12:30 | Wrap-up| 
|13:00 | END| 


## 🔧 Exercises
#### Exercise: select the underground water monitoring wells in the cropped raster extent
`brogmwvolledigeset.zip` contains the underground water monitoring wells of the whole Netherlands. They are not all relevant to us. Can you 1) select all wells within the extent of the cropped raster `raster_clip`, and 2) plot the selected wells on top of `raster_clip`?
Hint: think of how we selected the crop fields.

Solution:
```python
# Project to the right crs
wells = wells.to_crs(true_color_image_clip.rio.crs)

#Get the bounding box
xmin, ymin, xmax, ymax = true_color_image_clip.rio.bounds()

# Crop the data
wells = wells.cx[xmin:xmax, ymin:ymax]
wells.plot()
```
![](https://i.imgur.com/oTOcMIT.png)


## 🧠 Collaborative Notes
```python
import pystac
import rioxarray

items = pystac.ItemCollection.from_file("search.json")

item = items[1]

item_href = item.assets["visual"].href

true_color_image = rioxarray.open_rasterio(item_href)

true_color_image

true_color_image.rio.crs

from pyproj import CRS

CRS(true_color_image.rio.crs)

```

```python
import geopandas as gpd
cropfield = gpd.read_file("cropfield.shp")
```

```python
cropfield.crs
```

Transform coordinates of vector data to our raster
```python
cropfield_trans = cropfield.to_crs(true_color_image.rio.crs)
cropfield.trans.crs
```

Transform raster image to our cropfield crs (commented out because it is not a good idea!)
```python
true_color_image_trans = true_color_image.rio.reproject(cropfield.crs)
```


Save our converted cropfield in the cropfield variable:
```python
cropfield = cropfield.to_crs(true_color_image.rio.crs)
```

Inspect clip box function:
```python
true_color_image.rio.clip_box?
```

```python
bounds = cropfield.total_bounds
bounds
```


Clip the raster ima
```python
true_color_image_clip = true_color_image.rio.clip_box(*bounds)
```

```python
true_color_image_clip.plot.imshow()
```
![](https://i.imgur.com/6D0fFmn.png)

```python
from matplotlib import pyplot
```

Plot polygons on top of raster data:
```python
fig, ax = pyplot.subplots()
true_color_image_clip.plot.imshow(ax=ax)
cropfield.plot(ax=ax)
```
![](https://i.imgur.com/OQVMjYf.png)


```python
true_color_image_clip.rio.clip?
```

```python
true_color_image_cropfield = true_color_image_clip.rio.clip(cropfield["geometry"])
```

```python
true_color_image_cropfield.plot.imshow()
```
![](https://i.imgur.com/bttWl1e.png)


```python
true_color_image_clip.rio.to_raster("raster_clip.tif")
```

```python
wells = gpd.read_file("data/brogmwvolledigeset.zip")
```

```python
wells.plot()
```
![](https://i.imgur.com/Ox6ND72.png)

### Break until 10:40! :coffee: 

### Raster calculations
Normalized Difference Vegetation index (NDVI)
NDVI = (NR-red)/(NIR+red)

```python
import pystac
```

```python
items = pystac.ItemCollection.from_file("search.json")
```

```python
item=items[1]
```

```python
href_red = item.assets["B04"].get_absolute_href()
href_nir = item.assets["B08"].get_absolute_href()
```

```python
import rioxarray
```

```python
red = rioxarray.open_rasterio(href_red, masked=True)
nir = rioxarray.open_rasterio(href_nir, masked=True)

# Inspect our red Data
red
```

Our bounding box:
```python
bbox = (629_000, 5_804_000, 639_000, 5_814_000)
```

```python
red_clip = red.rio.clip_box(*bbox)
nir_clip = nir.rio.clip_box(*bbox)
```

Plot the red band (data will be loaded now)
```python
red_clip.plot()
```
The contrast is not very high:
![](https://i.imgur.com/BCd6DrE.png)

Let's add the "robust" argument to increase the contrast:
```python
red_clip.plot(robust=True)
```
![](https://i.imgur.com/pXh5M4C.png)

Plot the near infrared band:
```python
nir_clip.plot(robust=True)
```
![](https://i.imgur.com/D2FC097.png)

Compare crs
```python
red_clip.rio.crs, nir_clip.rio.crs
```

Compare shape:
```python
red_clip.shape, nir_clip.shape
```

```python
ndvi = (nir_clip - red_clip)/(nir_clip + red_clip)
```

Plot the result:
```python
ndvi.plot()
```
![](https://i.imgur.com/zSpzaLb.png)


Histogram:
```python
ndvi.plot.hist()
```
![](https://i.imgur.com/D0CGfcs.png)


Increase number of bins for better resolution
```python
ndvi.plot.hist(bins=50)
```
![](https://i.imgur.com/JfoKpWJ.png)

Check if there are nodata values:
```python
ndvi.isnull().any()
```
Check if red_clip data has any null values:
```python!
red_clip.isnull().any()
```

```python
mask = red_clip.isnull()
red_clip.where(mask, drop=True)
```

Let's view our red band again
```python
red_clip.plot(robust=True)
```
![](https://i.imgur.com/MMkVn2Y.png)

View x coordinates
```python
red_clip.x
```

Mask based on x coordinate:
```python!
mask_x = red_clip.x > 634_000
red_clip_x = red_clip.where(mask_x)
```

Visualize:
```python
red_clip_x.plot(robust=True)
```
![](https://i.imgur.com/lgnuFP2.png)

```python!
red_clip_x_dropped = red_clip.where(mask_x, drop=True)
red_clip_x_dropped.plot(robust=True)
```
![](https://i.imgur.com/tLW0zia.png)

### :tea: Break until 11:35!

### Parallel raster calculations

```python!
import pystac
items = pystac.ItemCollection.from_file("search.json")
```
```python
visual_href = items[-1].assets["visual"].get_absolute_href()
scl_href = items[-1].assets["SCL"].get_absolute_href()
```

```python
import rioxarray
```

```python
scl =rioxarray.open_rasterio(scl_href)
visual = rioxarray.open_rasterio(visual_href)
```

Again, compare crs
```python
scl.rio.crs, visual.rio.crs
```

Compare resolution:
```python
scl.rio.resolution(),  visual.rio.resolution()
```

Run gdalinfo in jupyter
```python!
!gdalinfo $visual_href
```

Reopen our true color image, now with matching resolution
```python!
visual = rioxarray.open_rasterio(visual_href, overview_level=0)
visual.rio.resolution()
```

```python!
%%time
scl=scl.load()
visual = visual.load()
```

```python!
visual.plot.imshow()
```
![](https://i.imgur.com/NziFxAu.png)


```python!
scl.plot()
```
![](https://i.imgur.com/uMjVd8r.png)


```python!
mask = scl.isin([8,9])
visual.where(~mask, other=visual.rio.nodata)
```
This will give an error

Let's drop the single band dimension and try again
```python!
mask = scl.isin([8,9]).squeeze()
visual_masked = visual.where(~mask, other=visual.rio.nodata)
visual_masked.rio.to_raster("band_masked.tif")
```

visualize
```python
visual_masked.plot.imshow()
```
![](https://i.imgur.com/BnAKSS2.png)

View scl dataArray:
```python!
scl
```

Specify chunks to use dask to load data
```python!
scl = rioxarray.open_rasterio(scl_href, chunks=(1, 2048, 2048), lock=False)
scl
```

Load visual in the same way
```python!
visual = rioxarray.open_rasterio(visual_href, overview_level=0, chunks=(3,2048, 2048), lock=False)
```

Loading the data with dask needs to be timed in a different way
```python!
%%time
scl = scl.persist(scheduler="threads", num_workers=4)
visual = visual.persist(scheduler="threads", num_workers=4)
```


```python!
from threading import Lock
```
```python!
mask = scl.isin([8,9]).squeeze()
visual_masked = visual.where(~mask, other=visual.rio.nodata)
visual_stored = visual_masked.rio.to_raster("band_masked.tif", tiled=True, lock=Lock(), compute=False)
```

Visualize computation:
```python!
visual_stored.visualize()
```

## Tips/Tops
### Tip :-1: 
- Ask beforehand what types of figures everyone wants to make, so you can maybe tailer the workshop better to everyone's interest (x2)
- Maybe a more insightfull compute example with a simple dask array would bring across the usefullness of Dask bettter.
- Everything went fine in my opinion
### Top :+1:
- A lot of the Dask-rioxarray pecularities were touched.
- +1 appreciated the lecture a lot
- Liked it a lot, THANKS
- I really liked to the online format of coding together, very efficient way to learn a lot
- Learn a lot and very useful..thanks!
## 📚 Resources
- [Dask documentation](https://docs.dask.org/en/stable/)
- [Sentinel documentation](https://sentinels.copernicus.eu/web/sentinel/home)
## Post-workshop survey

Please fill this in! [link](https://www.surveymonkey.com/r/R3QJDMP)