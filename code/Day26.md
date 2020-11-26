Day26
================
Anirudh Govind
(26 November, 2020)

## Get Data

``` r
# Load Bangalore wards

bangaloreWardBoundary <- read_sf(here::here("data/raw-data/bangaloreWardBoundary.shp"))

bangaloreWardBoundary <- bangaloreWardBoundary%>% 
  st_transform(3857)

bangaloreWardBoundarySP <- as_Spatial(bangaloreWardBoundary)
```

``` r
# Load contour data

bangaloreDEM <- raster(here::here("data/raw-data/mosaic.tif"))

bangaloreHillshade <- raster(here::here("data/raw-data/hillshade.tif"))
```

``` r
# Get elevation data

bangaloreElevation <- get_elev_raster(bangaloreWardBoundarySP,
                                      z = 14)
```

    ## Note: Your request will download approximately 271Mb.

    ## Mosaicing & Projecting

    ## Note: Elevation units are in meters.
    ## Note: The coordinate reference system is:
    ##  +proj=merc +a=6378137 +b=6378137 +lat_ts=0 +lon_0=0 +x_0=0 +y_0=0 +k=1 +units=m +nadgrids=@null +no_defs

## Wrangle Data

``` r
# Simplify data

bangaloreContours <- rasterToContour(bangaloreDEM,
                                     maxpixels = 10000000000)

# Convert to sf and transform

bangaloreContours <- bangaloreContours %>% 
  st_as_sf(.) %>% 
  st_transform(3857)

# Clip data to Bangalore's extents

bangaloreContours <- st_intersection(bangaloreContours, bangaloreWardBoundary)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# Using the "tanaka" package

bangaloreContoursTan <- tanaka_contour(bangaloreDEM,
                                       mask = bangaloreWardBoundary)
```

``` r
# Clip the hillshade raster to Bangalore's Boundary

bangaloreHillshadeCrop <- crop(bangaloreHillshade, extent(bangaloreWardBoundary))

bangaloreHillshadeCrop <- mask(bangaloreHillshadeCrop, bangaloreWardBoundary)
```

## Build Map

``` r
mapBangaloreContours <- tm_shape(bangaloreWardBoundary) +
  tm_borders(lwd = 3,
             col = "#000000") + 
  tm_shape(bangaloreHillshadeCrop) +
  tm_raster(palette = gray(0:100 / 100), 
            n = 100, 
            legend.show = FALSE) + 
  tm_shape(bangaloreContoursTan) +
  tm_borders(col = "#000000",
           lwd = 1.2) +
  tm_shape(bangaloreContours) +
  tm_lines(col = "#ffffff",
           lwd = 1.2)  + 
  tm_layout(bg.color = "white",
            frame = F,
            frame.lwd = NA,
            attr.outside = T,
            outer.margins = 0,
            asp = 0,
            scale = 0.8,
            main.title = "Bangalore's Contours",
            main.title.color = "#000000",
            main.title.size = 1.75,
            main.title.fontfamily = "Arial Narrow",
            title = "Contour Lines in black from a DEM Raster\nContour lines in white from a Hillshade Raster",
            title.color = "#000000",
            title.size = 0.8,
            title.position = c("left", "TOP"),
            title.fontface = 1) + 
  tm_credits("#30DayMapChallenge | Day 26 | Anirudh Govind | Nov 2020\nDEM and Hillshade Data via terradactile, accessible at https://terradactile.sparkgeo.com/",
             col = "#000000",
             size = 0.9,
             position = c("left", "bottom"),
             fontfamily = "Arial Narrow")
```

## Export

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = mapBangaloreContours,
          filename = here::here("exports/Day26.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day26.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)
