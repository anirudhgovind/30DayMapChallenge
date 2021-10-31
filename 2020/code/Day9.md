Day9
================
Anirudh Govind
(09 November, 2020)

## Get Data

``` r
# Load Bangalore wards

bangaloreWards <- read_sf(here::here("data/raw-data/bangaloreWardsUTM.shp"))

bangaloreWards <- bangaloreWards %>% 
  st_transform(3857)
```

``` r
# Load in previously downloaded data from OSM

bangaloreBuildings <- readRDS(here::here("data/raw-data/bangaloreBuildings.rds"))

bangaloreBuildings <- bangaloreBuildings %>%
  st_transform(3857)
```

``` r
# Load roads data (previously saved from OSM and cleaned up)

bangaloreRoads <- readRDS(here::here("data/derived-data/bangaloreRoads.rds"))

bangaloreRoads <- bangaloreRoads %>% 
  st_transform(3857)
```

``` r
# Parks

parks <- readRDS(here::here("data/raw-data/parks.rds"))

bangaloreParks <- parks$osm_polygons

bangaloreParks <- bangaloreParks %>% 
  st_transform(3857)
```

``` r
# Lakes and water

bangaloreWater <- readRDS(here::here("data/raw-data/bangaloreWater.rds"))

bangaloreWater <- bangaloreWater %>% 
  st_transform(3857)
```

``` r
# Get leisure pitch data from OSM
# 
# query <- getbb("Bangalore") %>%
#   opq() %>%
#   add_osm_feature("leisure", "pitch")
# 
# str(query)
# 
# pitch <- osmdata_sf(query)
# 
# saveRDS(pitch, here::here("data/raw-data/pitch.rds"))

pitch <- readRDS(here::here("data/raw-data/pitch.rds"))
```

``` r
# Get footway data from OSM

# query <- getbb("Bangalore") %>%
#   opq() %>%
#   add_osm_feature("highway", "footway")
# 
# str(query)
# 
# footway <- osmdata_sf(query)
# 
# saveRDS(footway, here::here("data/raw-data/footway.rds"))

footway <- readRDS(here::here("data/raw-data/footway.rds"))
```

## Wrangle Data

``` r
# Filter the buildings data to keep only necessary info

bangaloreBuildings <- bangaloreBuildings %>% 
  select(osm_id, geometry)

# I want this map to focus on the Basavanagudi area. I'll pick a point in MN Krishna Rao Park
# 
# sunkenahalliCentroid <- mapview(bangaloreWards) %>% 
#   editMap() %>% 
#   pluck("finished") %>% 
#   st_transform(3857)
# 
# saveRDS(sunkenahalliCentroid, here::here("data/raw-data/sunkenahalliCentroid.rds"))

sunkenahalliCentroid <- readRDS(here::here("data/raw-data/sunkenahalliCentroid.rds"))

# Draw a buffer around it of 2km

sunkenahalliBuffer <- st_buffer(sunkenahalliCentroid, 2000)

# Then I clip to the necessary data

sunkenahalliBuildings <- st_intersection(sunkenahalliBuffer, bangaloreBuildings)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# Filter roads to keep only necessary info

sunkenahalliRoads <- st_intersection(sunkenahalliBuffer, bangaloreRoads)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# Filter parks for sunkenahalli

sunkenahalliParks <- st_intersection(sunkenahalliBuffer, bangaloreParks)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# Filter lakes and water for sunkenahalli

sunkenahalliWater <- st_intersection(sunkenahalliBuffer, bangaloreWater)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# Sunkenahalli pitches

bangalorePitch <- pitch$osm_polygons

bangalorePitch <- bangalorePitch %>% 
  st_transform(3857)

sunkenahalliPitch <- st_intersection(sunkenahalliBuffer, bangalorePitch)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# Sunkenahalli footways

bangaloreFootways <- footway$osm_lines

bangaloreFootways <- bangaloreFootways %>% 
  st_transform(3857)

sunkenahalliFootways <- st_intersection(sunkenahalliBuffer, bangaloreFootways)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

## Build Map

``` r
# Here I put the map together

sunkenahalliMap <- tm_shape(sunkenahalliBuffer) +
  tm_borders(col = "#0077b6",
             lwd = 4) + 
  tm_shape(sunkenahalliBuildings) +
  tm_fill(col = "#0077b6",
             lwd = 0.5) +
  tm_layout(bg.color = "white",
            frame = F,
            attr.outside = T,
            outer.margins = 0,
            asp = 0,
            scale = 0.8,
            main.title = "Bangalore: Basavanagudi & surrounding areas",
            main.title.color = "#0077b6",
            main.title.size = 1.75,
            main.title.fontface = 2,
            main.title.fontfamily = "Arial Narrow") + 
  tm_credits("#30DayMapChallenge | Day 9 | Anirudh Govind | Nov 2020\nMap data © OpenStreetMap contributors and available from https://www.openstreetmap.org",
             col = "#0077b6",
             size = 0.8,
             position = c("left", "bottom"),
             fontfamily = "Arial Narrow") + 
  tm_shape(sunkenahalliRoads) +
  tm_lines(col = "#0077b6") +
  tm_shape(sunkenahalliParks) +
  tm_fill(col = "#0077b6") +
  tm_shape(sunkenahalliWater) +
  tm_fill(col = "#0077b6") +
  tm_shape(sunkenahalliPitch) +
  tm_fill(col = "#0077b6") +
  tm_shape(sunkenahalliFootways) +
  tm_lines(col = "#ffffff",
           lty = "dashed")
```

## Export

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = sunkenahalliMap,
          filename = here::here("exports/Day9.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day9.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)
