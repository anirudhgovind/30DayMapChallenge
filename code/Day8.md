Day8
================
Anirudh Govind
(08 November, 2020)

# Common Data

## Get Data

``` r
# Load Bangalore ward boundaries

bangaloreWardBoundary <- read_sf(here::here("data/raw-data/bangaloreWardBoundary.shp"))

bangaloreWardBoundary <- bangaloreWardBoundary %>% 
  st_transform(3857)
```

``` r
# Load roads data (previously saved from OSM and cleaned up)

bangaloreRoads <- readRDS(here::here("data/derived-data/bangaloreRoads.rds"))

bangaloreRoads <- bangaloreRoads %>% 
  st_transform(3857)
```

``` r
# Load in previously downloaded data from OSM

bangaloreBuildings <- readRDS(here::here("data/raw-data/bangaloreBuildings.rds"))

bangaloreBuildings <- bangaloreBuildings %>% 
  select(osm_id, geometry)

bangaloreBuildings <- bangaloreBuildings %>%
  st_transform(3857)
```

``` r
# Get train station data. I need the RV Road Station
# name == "Rashtriya Vidyalaya Road"
# 
# query <- getbb("Bangalore") %>%
#   opq() %>%
#   add_osm_feature("public_transport", "stop_position")
# 
# str(query)
# 
# publicTransportStops <- osmdata_sf(query)
# 
# saveRDS(publicTransportStops, here::here("data/raw-data/publicTransportStops.rds"))

publicTransportStops <- readRDS(here::here("data/raw-data/publicTransportStops.rds"))
```

# Rashtriya Vidyalaya Road

## Wrangle Data

``` r
# Keep only relevant info

publicTransportStops <- publicTransportStops$osm_points

# Filter to rv road station

rvRoadStation <- publicTransportStops %>% 
  filter(name == "Rashtriya Vidyalaya Road")

rvRoadStation <- rvRoadStation %>% 
  select(osm_id, geometry)

# Since the dots are side by side, I'll select just one

rvRoadStation <- rvRoadStation %>% 
  slice(1)

# Transform for consistency

rvRoadStation <- rvRoadStation %>% 
  st_transform(3857)
```

``` r
# Create a 500m buffer around RV Road Metro Station

rvBuffer <- st_buffer(rvRoadStation, 500)

rvBuffer <- rvBuffer %>% 
  mutate(desc = "500m")
```

``` r
# Create buffer around mapping area

rvMapAreaBuffer <- st_buffer(rvRoadStation, 750)
```

``` r
# get buildings in the mapArea

rvMapAreaBuildings <- st_intersection(bangaloreBuildings, rvMapAreaBuffer)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# Clip roads to the map buffer area

rvMapAreaRoads <- st_intersection(rvMapAreaBuffer, bangaloreRoads)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
tm_shape(rvMapAreaRoads) +
  tm_lines()
```

![](Day8_files/figure-gfm/unnamed-chunk-9-1.png)<!-- -->

``` r
# Context roads

rvContextRoads <- st_difference(rvMapAreaRoads, rvBuffer)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# 500m area roads

rvFiveWalkableRoads <- st_intersection(rvMapAreaRoads, rvBuffer)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

## Create Isochrone

``` r
# rvIsochrone <- mb_isochrone(rvRoadStation,
#                             profile = "walking",
#                             time = c(6),
#                             access_token = myToken)
# 
# saveRDS(rvIsochrone, here::here("data/derived-data/rvIsochrone.rds"))

rvIsochrone <- readRDS(here::here("data/derived-data/rvIsochrone.rds"))
```

## Wrangle Data

``` r
# Transform isochrone for consistency

rvIsochrone <- rvIsochrone %>% 
  st_transform(3857)

# Isochrone roads

rvIsoWalkableRoads <- st_intersection(rvMapAreaRoads, rvIsochrone)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# Isochrone buildings

rvIsoBuildings <- st_intersection(rvMapAreaBuildings, rvIsochrone)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
rvContextBuildings <- st_difference(rvMapAreaBuildings, rvIsochrone)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# 500m radius buildings

rvBuildingsFive <- st_intersection(rvContextBuildings, rvBuffer)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

## Build Map

``` r
# Palette

c("#000000", "#14213d", "#fca311", "#e5e5e5", "#ffffff")
```

    ## [1] "#000000" "#14213d" "#fca311" "#e5e5e5" "#ffffff"

``` r
# Context Buildings

rvMapContext <- rvContextBuildings %>% 
  tm_shape() +
  tm_fill(col = "#000000",
          alpha = 0.1) +
  tm_layout(bg.color = "#ffffff",
            frame = F,
            frame.lwd = NA,
            attr.outside = F,
            outer.margins = 0,
            asp = 0,
            scale = 0.8,
            main.title = "Bangalore: Rashtriya Vidyalaya Road Metro Station",
            main.title.color = "#000000",
            main.title.size = 1.75,
            main.title.fontfamily = "Arial Narrow",
            main.title.fontface = 2,
            title = "Variations in accessibility: 500m radius vs. route-based",
            title.color = "#000000",
            title.size = 1,
            title.position = c("left", "TOP"),
            title.fontface = 2) + 
  tm_credits("#30DayMapChallenge | Day 8 | Anirudh Govind | Nov 2020\nMap data © OpenStreetMap contributors and available from https://www.openstreetmap.org",
             col = "#000000",
             size = 0.8,
             position = c("left", "bottom"),
             fontfamily = "Arial Narrow")

# Roads Context

rvMapContextRoads <- tm_shape(rvContextRoads) +
  tm_lines(col = "#000000",
           alpha = 0.2)

# Roads 500m

rvMap500Roads <- tm_shape(rvFiveWalkableRoads) +
  tm_lines(col = "#000000")

# Buffer 500m

rvMap500Buffer <- tm_shape(rvBuffer) +
  tm_borders(col = "#000000",
             lwd = 3.0,
             lty = "dashed")

# Buildings 500m

rvMap500Buildings <- rvBuildingsFive %>% 
  tm_shape() +
  tm_fill(col = "#000000",
          alpha = 0.35)

# Isochrone

rvMapIsoBoundary <- rvIsochrone %>% 
  tm_shape() +
  tm_borders(col = "#000000",
           lwd = 3.0,
           lty = "dotted")

# Isochrone buildings

rvMapIsoBuildings <- rvIsoBuildings %>% 
  filter(osm_id != 331975433) %>% 
  tm_shape() +
  tm_fill(col = "#000000",
          alpha = 0.8)

# Metro station

mapRVMetroStation <- rvIsoBuildings %>% 
  filter(osm_id == 331975433) %>% 
  tm_shape() +
  tm_fill(col = "#fca311")

mapRVRoad <- rvMapContext + 
  rvMapContextRoads + 
  rvMap500Roads + 
  rvMap500Buffer + 
  rvMap500Buildings +
  rvMapIsoBuildings +
  rvMapIsoBoundary + 
  mapRVMetroStation
```

## Export

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = mapRVRoad,
          filename = here::here("exports/Day8-1.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day8-1.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)

# MG Road Station

## Wrangle Data

``` r
# Filter metro stations

metroStations <- publicTransportStops %>% 
  filter(network == "Namma Metro")

# Filter to rv road station

mgRoadStation <- metroStations %>% 
  filter(name == "Mahatma Gandhi Road")

mgRoadStation <- mgRoadStation %>% 
  select(osm_id, geometry)

# Since the dots are side by side, I'll select just one

mgRoadStation <- mgRoadStation %>% 
  slice(1)

# Transform for consistency

mgRoadStation <- mgRoadStation %>% 
  st_transform(3857)
```

``` r
# Create a 500m buffer around MG Road Metro Station

mgBuffer <- st_buffer(mgRoadStation, 500)
```

``` r
# Create buffer around mapping area

mgMapAreaBuffer <- st_buffer(mgRoadStation, 750)
```

``` r
# get buildings in the mapArea

mgMapAreaBuildings <- st_intersection(bangaloreBuildings, mgMapAreaBuffer)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# Clip roads to the map buffer area

mgMapAreaRoads <- st_intersection(mgMapAreaBuffer, bangaloreRoads)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# Context roads

mgContextRoads <- st_difference(mgMapAreaRoads, mgBuffer)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# 500m area roads

mgFiveWalkableRoads <- st_intersection(mgMapAreaRoads, mgBuffer)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

## Create Isochrone

``` r
# mgIsochrone <- mb_isochrone(mgRoadStation,
#                             profile = "walking",
#                             time = c(6),
#                             access_token = myToken)
# 
# saveRDS(mgIsochrone, here::here("data/derived-data/mgIsochrone.rds"))

mgIsochrone <- readRDS(here::here("data/derived-data/mgIsochrone.rds"))
```

## Wrangle Data

``` r
# Transform isochrone for consistency

mgIsochrone <- mgIsochrone %>% 
  st_transform(3857)

# Isochrone roads

mgIsoWalkableRoads <- st_intersection(mgMapAreaRoads, mgIsochrone)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# Isochrone buildings

mgIsoBuildings <- st_intersection(mgMapAreaBuildings, mgIsochrone)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
mgContextBuildings <- st_difference(mgMapAreaBuildings, mgIsochrone)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# 500m radius buildings

mgBuildingsFive <- st_intersection(mgContextBuildings, mgBuffer)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

## Build Map

``` r
# Palette

c("#000000", "#14213d", "#fca311", "#e5e5e5", "#ffffff")
```

    ## [1] "#000000" "#14213d" "#fca311" "#e5e5e5" "#ffffff"

``` r
# Context Buildings

mgMapContext <- mgContextBuildings %>% 
  tm_shape() +
  tm_fill(col = "#000000",
          alpha = 0.1) +
  tm_layout(bg.color = "#ffffff",
            frame = F,
            frame.lwd = NA,
            attr.outside = F,
            outer.margins = 0,
            asp = 0,
            scale = 0.8,
            main.title = "Bangalore: Mahatma Gandhi Road Metro Station",
            main.title.color = "#000000",
            main.title.size = 1.75,
            main.title.fontfamily = "Arial Narrow",
            main.title.fontface = 2,
            title = "Variations in accessibility: 500m radius vs. route-based",
            title.color = "#000000",
            title.size = 1,
            title.position = c("left", "TOP"),
            title.fontface = 2) + 
  tm_credits("#30DayMapChallenge | Day 8 | Anirudh Govind | Nov 2020\nMap data © OpenStreetMap contributors and available from https://www.openstreetmap.org",
             col = "#000000",
             size = 0.8,
             position = c("left", "bottom"),
             fontfamily = "Arial Narrow")

# Roads Context

mgMapContextRoads <- tm_shape(mgContextRoads) +
  tm_lines(col = "#000000",
           alpha = 0.2)

# Roads 500m

mgMap500Roads <- tm_shape(mgFiveWalkableRoads) +
  tm_lines(col = "#000000")

# Buffer 500m

mgMap500Buffer <- tm_shape(mgBuffer) +
  tm_borders(col = "#000000",
             lwd = 3.0,
             lty = "dashed")

# Buildings 500m

mgMap500Buildings <- mgBuildingsFive %>% 
  tm_shape() +
  tm_fill(col = "#000000",
          alpha = 0.35)

# Isochrone

mgMapIsoBoundary <- mgIsochrone %>% 
  tm_shape() +
  tm_borders(col = "#000000",
           lwd = 3.0,
           lty = "dotted")

# Isochrone buildings

mgMapIsoBuildings <- mgIsoBuildings %>% 
  filter(osm_id != 348854341) %>% 
  tm_shape() +
  tm_fill(col = "#000000",
          alpha = 0.8)

# Metro station

mapMGMetroStation <- mgIsoBuildings %>% 
  filter(osm_id == 348854341) %>% 
  tm_shape() +
  tm_fill(col = "#8b1c7a")

mapMGRoad <- mgMapContext + 
  mgMapContextRoads + 
  mgMap500Roads + 
  mgMap500Buffer + 
  mgMap500Buildings +
  mgMapIsoBuildings +
  mgMapIsoBoundary + 
  mapMGMetroStation
```

## Export

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = mapMGRoad,
          filename = here::here("exports/Day8-2.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day8-2.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)
