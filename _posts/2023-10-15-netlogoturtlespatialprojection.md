---
title: "Transform Netlogo turtle coordinates into spatial projection"
tags:
  - NetLogo 
  - spatial 
  - tutorial
  - Rcode 
#date: 2023-6-15 / 2023-10-15
output:
  md_document:
    variant: markdown_github
    preserve_yaml: true
editor_options: 
  chunk_output_type: console
---

Learn how to transform the relative coordinates of the individuals from
Netlogo into coordinates from real maps. This code is especially
designed for spatially explicit netlogo models that were set to store
the individual coordinates (xcor, ycor) in the output for the turtle
data.

# How to project Netlogo Turtle coordinates into a real map

When using spatial data in Netlogo, the coordinates of a raster get
transformed to relative coordinates. This means, the cell in the bottom
left gets coordinate (1,1), the one on top of it is (1,2), and so on.

After running a model, usually we want to reproject the output back to
the spatial data coordinates used, either for post-simulation analyses
or for plotting.

This code shows how to project the turtles’ coordinates back into a map,
when a raster was used to create the Netlogo landscape.

For this we need: - The raster used as netlogo input - The turtle
coordinates in the output

## 0. Load libraries

## 1. Create data

In this example we create a raster and some turtle data to use. With
real data, you will load your raster here and make sure it has a
PROJECTED coordinate system. Turtle data will have different formats
depending how it was created, the basic data we need for this is the
identity of the turtle and the coordinates.

``` r
## Create raster with 100 cells for the example
myraster <- rast(nrows = 100, ncols = 100, 
                 xmin = 4541100, xmax = 4542100, 
                 ymin = 3265800, ymax = 3266800)
## give random values to the raster
myraster <- init(myraster, sample(1:1000))

## assign projection
crs(myraster) <- "epsg:3035"
plot(myraster)
```

<img src="/assets/images_tutorials/get raster and turtle data-1.png" style="display: block; margin: auto;" />

``` r
## turtle data - example data
who <- seq(1,10)
xcoord <- sample(1:100, 10) # create random integers for x coordinate
ycoord <- sample(1:100, 10) # create random integers for y coordinate

turtle_variables <- cbind.data.frame(who, xcoord, ycoord)
head(turtle_variables)
```

    ##   who xcoord ycoord
    ## 1   1     15     43
    ## 2   2     82     67
    ## 3   3     26     55
    ## 4   4     40     68
    ## 5   5      5     26
    ## 6   6      1     69

Now that we have our data, let’s extract the map coordinates as
reference and transform the turtle ones. This process will work with any
PROJECTED coordinate system.

## 2. Get reference coordinates

We need the bottom left corner of the map as a reference point and the
resolution of the map

``` r
# we are going to trasnform the cell relative numbering to real coordinates, starting left down as this is where netlogo starts numbering the cells
start_left <- xmin(myraster)
start_down <- ymin(myraster)
my_res <- res(myraster)[1]
```

## 3. Transform turtle coordinates into map coordinates

Now we use the reference point to transform our coordinates into the
projected coordinates and the resolution to correct for the size of the
cells

``` r
turtle_spatial <- turtle_variables %>% 
  mutate(spatial_xcoord = start_left + ((xcoord * my_res) + my_res/2), 
         #divided by 2 to locate in the center of the cell
         spatial_ycoord = start_down + ((ycoord * my_res) + my_res/2))


## make spatial points
turtle_sf <- st_as_sf(turtle_spatial, 
                      coords = c("spatial_xcoord", "spatial_ycoord"), 
                      crs = crs(myraster))
```

## 4. Plot with ggplot2

``` r
## make myraster a dataframe
map_df <- as.data.frame(myraster, xy = TRUE)
names(map_df) <- c("x","y","hs")
head(map_df)
```

    ##         x       y  hs
    ## 1 4541105 3266795 933
    ## 2 4541115 3266795 207
    ## 3 4541125 3266795 733
    ## 4 4541135 3266795  93
    ## 5 4541145 3266795 685
    ## 6 4541155 3266795 491

``` r
## plot map with turtles on top
plot_mymap <- ggplot() +
  geom_tile(data = map_df, aes(x = x, y = y, fill = hs), alpha = 0.4) +
  scale_fill_viridis_c(na.value = "transparent",
                       name = "Habitat suitability") +
  geom_sf(data = turtle_sf, col = "red") +
  annotation_scale() +
  annotation_north_arrow(location = "tl", 
    pad_x = unit(0.4, "in"), pad_y = unit(0.4, "in")) +
  ylab("latitude") +
  xlab("longitude") +
  theme_bw() +
  theme(
    plot.background = element_blank()
  )

plot_mymap
```

<img src="/assets/images_tutorials/plot turtle ggplot-1.png" style="display: block; margin: auto;" />
