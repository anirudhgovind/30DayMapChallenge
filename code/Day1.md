Day1
================
Anirudh Govind
(06 November, 2020)

## Get Data

``` r
# Load Bangalore ward boundaries

bangaloreWardBoundary <- read_sf(here::here("data/raw-data/bangaloreWardBoundary.shp"))

bangaloreWardBoundary <- bangaloreWardBoundary%>% 
  st_transform(3857)
```

``` r
# Get traffic light location data from OSM

# query <- getbb("Bangalore") %>% 
#   opq() %>% 
#   add_osm_feature("highway", "traffic_signals")
# 
# str(query)
# 
# trafficSignals <- osmdata_sf(query)
# 
# saveRDS(trafficSignals, here::here("data/raw-data/trafficSignals.rds"))

trafficSignals <- readRDS(here::here("data/raw-data/trafficSignals.rds"))
```

``` r
# Load roads data (previously saved from OSM and cleaned up)

bangaloreRoads <- readRDS(here::here("data/derived-data/bangaloreRoads.rds"))
```

## Wrangle Data

``` r
# Filter point data of traffic signal locations

trafficSignalsData <- trafficSignals$osm_points

# Discard unnecessary data

trafficSignalsData <- trafficSignalsData %>% 
  select(osm_id, geometry)

# Convert CRS for consistency

trafficSignalsData <- trafficSignalsData %>% 
  st_transform(3857)
```

``` r
# Exclude any signals outside the ward boundary

trafficSignalsData <- st_intersection(trafficSignalsData, bangaloreWardBoundary)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# Filter roads data to keep a smaller subset

bangaloreRoadsFilter <- bangaloreRoads %>% 
  filter(highway == "trunk" | 
           highway == "motorway" | 
           highway == "primary" | 
           highway == "secondary")
```

## Build Map

``` r
# Define Palette

# c(#000000, #14213D, #FCA311, #E5E5E5, #FFFFFF)

# Build map in layers

# First layer will be the ward map boundary with a black background. I'll also add the title and credits to this layer.

mapBoundary <- tm_shape(bangaloreWardBoundary) +
  tm_fill(col = "#FCA311") + 
  tm_borders(col = "#E5E5E5",
             lwd = 2) +
  tm_layout(bg.color = "#FCA311",
            frame = F,
            attr.outside = T,
            outer.margins = 0,
            asp = 0,
            scale = 0.8,
            main.title = "Bangalore's Traffic Lights",
            main.title.color = "#14213D",
            main.title.size = 1.75,
            main.title.fontface = 2,
            main.title.fontfamily = "Arial Narrow") + 
  tm_credits("#30DayMapChallenge | Day 1 | Anirudh Govind | Nov 2020\nMap data © OpenStreetMap contributors and available from https://www.openstreetmap.org",
             col = "#14213D",
             size = 0.8,
             position = c("left", "bottom"),
             fontfamily = "Arial Narrow")

# The signals (as dots) looked too abstract and vague. I'll add in the roads as a very faint layer so the context is clearer.

mapRoads <- tm_shape(bangaloreRoadsFilter) +
  tm_lines(col = "#E5E5E5",
           alpha = 0.35)

# The signals will be shown as dots. The size needs to be just right so that ones which are close by don't overlap. AH! So, looking at the map in "view" mode shows that there are multiple signals at each intersection. These overlap when seen in "plot" mode. I don't think its necessary to show all of them distinctly. But, it'll be interesting to know how many such clusters there are.

mapSignals <- tm_shape(trafficSignalsData) +
  tm_dots(col = "#14213D",
          size = 0.1875,
          shape = 20,
          jitter = 0)

BangaloresTrafficSignals <- mapBoundary + mapRoads + mapSignals

BangaloresTrafficSignals
```

![](Day1_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

## Export

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = BangaloresTrafficSignals,
          filename = here::here("exports/Day1.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day1.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)
