---
title: "Extracting data from Rasters"
classes: wide
#categories:
#  - Blog
tags:
  - tutorial
  - Rcode
  - GIS
  - spatial analysis
---

To extract data from raster data in R we are going to use a point layer and a raster.

We start by loading the necesary libraries:
```r
library(sf)
library(terra)
```

First we load the points with our locations and the raster from which we want to obtain the information. In this example we use tree cover data.
```r
my_points <- st_read("my_points.gpkg")
tree_cover <- rast("tree_cover_100m_3035.tif")>
```

Now, we extract the tree cover at the point location and add it to the original point layer
```r
tree_cover_ext <- extract(tree_cover, my_points)
my_points$tree_cover <- tree_cover_ext$tree_cover>
```

Plot the new variable
```r
plot(my_points["tree_cover"])
```

