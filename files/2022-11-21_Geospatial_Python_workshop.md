![](https://i.imgur.com/iywjz8s.png)


# 2022-11-21 Geospatial Python workshop


Welcome to The Workshop Collaborative Document.

This Document is synchronized as you type, so that everyone viewing this page sees the same text. This allows you to collaborate seamlessly on documents.

----------------------------------------------------------------------------


This is the Document for today: [link](https://tinyurl.com/gsw20221121)

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


## 👩‍🏫👩‍💻🎓 Instructors

Francesco Nattino, Ou Ku

## 🧑‍🙋 Helpers

Djura Smits, Pranav Chandramouli 


## 🗓️ Agenda
| Time | Topic |
|--:|:---|
| 09:00 | Welcome and icebreaker | 
| 09:15 | Introduction to raster, vector, and CRS | 
| 09:45 | Access satellite imagery using Python | 
| 10:30 | Coffee break | 
| 10:45 | Read and visualize raster data | 
| 11:30 | Coffee break | 
| 11:45 | Read and visualize raster data | 
| 12:30 | Wrap-up | 
| 13:00 | END |


## 🔧 Exercises


#### Exercise: Discover a STAC catalog
Let’s take a moment to explore the Earth Search STAC catalog, which is the catalog indexing the Sentinel-2 collection that is hosted on AWS. We can interactively browse this catalog using the STAC browser at [this link](https://stacindex.org/catalogs/earth-search#/).

1. Open the link in your web browser. Which (sub-)catalogs are available?
2. Open the Sentinel-2 L2A COGs collection (link ending in `sentinel-s2-l2a-cogs`), and select one item from the list. Each item corresponds to a satellite “scene”, i.e. a portion of the footage recorded by the satellite at a given time. Have a look at the metadata fields and the list of assets. What kind of data do the assets represent?

#### Exercise: Downloading Landsat 8 Assets
In this exercise we put in practice all the skills we have learned in this episode to retrieve images from a different mission: [Landsat 8](https://www.usgs.gov/landsat-missions/landsat-8). In particular, we browse images from the [Harmonized Landsat Sentinel-2 (HLS) project](https://lpdaac.usgs.gov/products/hlsl30v002/), which provides images from NASA’s Landsat 8 and ESA’s Sentinel-2 that have been made consistent with each other. The HLS catalog is indexed in the NASA Common Metadata Repository (CMR) and it can be accessed from the STAC API endpoint at the following URL: https://cmr.earthdata.nasa.gov/stac/LPCLOUD.

- Using `pystac_client`, search for all assets of the Landsat 8 collection (`HLSL30.v2.0`) from February to March 2021, intersecting the point with longitude/latitute coordinates (-73.97, 40.78) deg.
- Visualize an item’s thumbnail (asset key `browse`).

## 🧠 Collaborative Notes
### Access satellite imagery using Python

Example of accessing satellite data through GUI (login needed):
https://scihub.copernicus.eu/

Activate your environment:
`conda activate geospatial`

Create a notebook

#### (Meta)Data access
```python=
api_url = "https://earth-search.aws.element84.com/v0"
```
Import pystac client

```python=
from pystac_client import Client
```

Create our client object
```python=
client = Client.open(api_url)
```

Display nice representation of object 
```python=
client # (And press shift-enter)
```

```python=
collection = "sentinel-s2-l2a-cogs"
```

Import point class
```python=
from shapely.geometry import Point
```
```python=
point = Point(4.89, 52.37)
```

```python=
point
```

```python=
search = client.search(
                    collections=[collection],
                    intersects=point,
                    max_items=10, #default is 100
)
```
Check how many items fit the criteria
```python=
search.matched()
```

Get the scene
```python=
items = search.get_all_items()
```

View the results
```python=
items
```

Inspect results
```python=
len(items)
```

Print all items
```python=
for item in items:
    print(item)
```

Inspect first item
```python=
item = items[0]
```

```python=
item.datetime # Sensing time

item.geometry # Actual footprint of our tile
# from shapely.geometry import Polygon; Polygon(item.geometry["coordinates"][0])
```
View all properties:
```python=
item.properties
```

Let's do something else:

Intersect bounding box
```python=
# Buffer will create a circle, .bounds will retrieve the bounding box (square) around the circle
bbox = point.buffer(0.01).bounds # tuple (xmin, ymin, xmax, ymax)
```

Datetime filter:
```python=
datetime = "2020-03-20/2020-03-30"
```

Cloud cover smaller than x
```python=
query = "eo:cloud_cover<10"
```

Check documentation
```python=
client.search?
```
Note: `shift+tab` while within a function call (between the brackets) will give a quick pop-up of the docs in Jupyter lab

Let's write our new search query
```python=
search_bbox = client.search(
                        collections=[collection],
                        bbox=bbox,
                        datetime=datetime,
                        query=[query]
)
```

Check number of results:
```python=
search_bbox.matched()
```

Retrieve items:
```python=
items = search_bbox.get_all_items()
len(items)
```

```python=
items.save_object("search.json")
```

View the available assets:
```python=
item = items[0]
assets = item.assets
```

List the names/keys of the assets:
```python=
assets.keys()
```

Print descriptions of the assets:
```python=
for key, asset in assets.items():
    print(key, ": ", asset.title)
```

Retrieve link to asset:
```python=
assets['thumbnail'].get_absolute_href()
```

Another one:
```python=
assets["B01"].get_absolute_href()
```

Sometimes you might find additional metadata, e.g.
```python=
assets["B01"].common_metadata.gsd # (Not available)
```

Exercise solution
```python=
nasa_cms_api = "https://cmr.earthdata.nasa.gov/stac/LPCLOUD"
client_nasa_cmr = Client.open(nasa_cms_api)

client_nasa_cmr
```python
search = client.search(collections=["HLSL30.v2.0"], 
                        datetime="2021-02-01/2021-03-31",
                        intersects=Point(-73.97, 40.78),
                       )
```

```python
search.matched()
```

```python
items_nasa_cmr = search.get_all_items()
items_nasa_cmr
```

```python
item = items_nasa_cmr[0]
```

```python
item.assets
```

```python
item.assets["browse"].get_absolute_href()
```

#### Accessing the assets
```python
import rioxarray
```

```python
import pystac
```

Recreate our item collection from our previously saved file
```python
items = pystac.ItemCollection.from_file("search.json")
```

Take the first item:
```python
href = items[0].assets["B09"].get_absolute_href()
```

```python
raster_ams_b9 = rioxarray.open_rasterio(href)
raster_ams_b9
```

SPOILER ALERT: you can plot with `.plot`
```python
raster_ams_b9.plot()
```

## Tips/Tops

### Tip: What could we improve?
- Share the Zoom link earlier
- More sync moments
- Maybe provide contact point for questions about getting environments up and running beforehand.
- For me personally, the speed was actuallly a bit on the slow side. +
- If the jupyter notebook could be available beforehand, that can help us to understand more what you are telling instead of typing-
- Could just keep up typing/copying, a little bit slower until Djura finished would be nice
- I do not usually work on geospatial data (molecular imaging data, which is also high-dimensional raster data). During the code-along part, I was sometimes missing the big picture of what we were doing (e.g. we are making a rectangular bouding box centered on Amsterdam, so that we can then querry corresponding satelite images)
- perhaps allow some time to help with setting up the environment beforehand
- answering a lot of questions in between is actually a bit disruptive 

### Top: What did you like?
- Plenty of room to ask questions
- Nice examples, very helpful
- Already pretty applied and "in the field"
- Good use of breaks
- Exercises are very helpful
- Nice, I can do it now! 
- good speed
- My favorite was that all the necessary Python geospatial libraries were super easy to install in the form of a virtual environment (I have struggled with incompatibilities between geospatial libraries in the past). Basically, good set-up instructions.
-   v- g
-   It’s great I learned a lot, you are interactive 
- very nicely paced.
- Friendly accessible instructors. Dumb questions are ok :) 
- Very clear and good examples and exercises. Very friendly and helpful


## 📚 Resources
- [the Netherlands eScience Center](https://www.esciencecenter.nl)
- [Digital Skills Workshops](https://esciencecenter-digital-skills.github.io/)
- [NL-RSE](https://esciencecenter-digital-skills.github.io/)
- [NL-RSE](https://www.nl-rse.org/)
- https://stacspec.org/en
- https://epsg.io/
- https://geopandas.org/en/stable/docs.html
- [Copernicus open access hub](https://scihub.copernicus.eu/)
- Xarray - https://xarray.dev/
