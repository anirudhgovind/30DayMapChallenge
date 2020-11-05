Day5
================
Anirudh Govind
(06 November, 2020)

## Get Data

``` r
# Get water data from OSM

# query <- getbb("Bangalore") %>% 
#   opq() %>% 
#   add_osm_feature("natural", "water")
# 
# str(query)
# 
# osmWater <- osmdata_sf(query)
# 
# bangaloreWater <- osmWater$osm_polygons
# 
# saveRDS(bangaloreWater, here::here("data/raw-data/bangaloreWater.rds"))

bangaloreWater <- readRDS(here::here("data/raw-data/bangaloreWater.rds"))

bangaloreWater <- bangaloreWater %>% 
  st_transform(3857)
```

``` r
# Load Bangalore wards

bangaloreWardBoundary <- read_sf(here::here("data/raw-data/bangaloreWardBoundary.shp"))

bangaloreWardBoundary <- bangaloreWardBoundary%>% 
  st_transform(3857)
```

``` r
# Load roads data (previously saved from OSM and cleaned up)

bangaloreRoads <- readRDS(here::here("data/derived-data/bangaloreRoads.rds"))
```

``` r
# Load unclipped roads data (previously saved from OSM)

trunkRoads <- readRDS(here::here("data/raw-data/roadsTrunk.rds"))

motorRoads <- readRDS(here::here("data/raw-data/roadsMotorway.rds"))

primaryRoads <- readRDS(here::here("data/raw-data/roadsPrimary.rds"))

secondaryRoads <- readRDS(here::here("data/raw-data/roadsSecondary.rds"))
```

## Wrangle Data

``` r
# Filter roads data to keep a smaller subset

bangaloreRoadsFilter <- bangaloreRoads %>% 
  filter(highway == "trunk" | 
           highway == "motorway" | 
           highway == "primary" | 
           highway == "secondary")
```

``` r
# Keep only relevant data

bangaloreWater <- bangaloreWater %>% 
  select(osm_id, name, geometry)

# Calculate area of all natural waters

bangaloreWater %>% 
  mutate(area = st_area(.)) %>% 
  mutate(area = as.numeric(area)) %>% 
  mutate(totalArea = sum(area)) %>% 
  mutate(totalArea = round(totalArea, 2))
```

    ## Simple feature collection with 484 features and 4 fields
    ## geometry type:  POLYGON
    ## dimension:      XY
    ## bbox:           xmin: 8622202 ymin: 1440142 xmax: 8658091 ymax: 1480676
    ## projected CRS:  WGS 84 / Pseudo-Mercator
    ## First 10 features:
    ##      osm_id                name                       geometry        area
    ## 1  23000626       Hulimavu Lake POLYGON ((8639214 1445064, ...  445291.796
    ## 2  24430851      Gottigere Tank POLYGON ((8637195 1442645, ...  113972.875
    ## 3  24543535                <NA> POLYGON ((8644662 1446419, ...    5819.507
    ## 4  27993463                <NA> POLYGON ((8651536 1450115, ...  212260.619
    ## 5  27993527                <NA> POLYGON ((8650771 1450449, ...   20888.724
    ## 6  27993889     Ambalipura Lake POLYGON ((8645403 1449689, ...   36331.251
    ## 7  28043727        Varthur Lake POLYGON ((8654746 1453420, ... 1527993.910
    ## 8  28069377 Horamavu Agara Lake POLYGON ((8644760 1464013, ...  153583.906
    ## 9  28069384                <NA> POLYGON ((8640989 1468461, ...   23054.653
    ## 10 28069395                <NA> POLYGON ((8639392 1470307, ...  532738.903
    ##    totalArea
    ## 1   39848847
    ## 2   39848847
    ## 3   39848847
    ## 4   39848847
    ## 5   39848847
    ## 6   39848847
    ## 7   39848847
    ## 8   39848847
    ## 9   39848847
    ## 10  39848847

``` r
# Total area = 39848847 Sqm

# Calculate ara of natural waters within municipal boundary

bangaloreWater %>% 
  st_intersection(., bangaloreWardBoundary) %>% 
  mutate(area = st_area(.)) %>% 
  mutate(area = as.numeric(area)) %>% 
  mutate(totalArea = sum(area)) %>% 
  mutate(totalArea = round(totalArea, 2))
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

    ## Simple feature collection with 394 features and 5 fields
    ## geometry type:  GEOMETRY
    ## dimension:      XY
    ## bbox:           xmin: 8624168 ymin: 1442299 xmax: 8658091 ymax: 1473438
    ## projected CRS:  WGS 84 / Pseudo-Mercator
    ## First 10 features:
    ##      osm_id                name id                       geometry        area
    ## 1  23000626       Hulimavu Lake  0 POLYGON ((8639214 1445064, ...  445291.796
    ## 2  24430851      Gottigere Tank  0 POLYGON ((8637195 1442645, ...  113972.875
    ## 3  24543535                <NA>  0 POLYGON ((8644662 1446419, ...    5819.507
    ## 4  27993463                <NA>  0 POLYGON ((8651536 1450115, ...  192932.444
    ## 5  27993527                <NA>  0 POLYGON ((8650771 1450449, ...   20888.724
    ## 6  27993889     Ambalipura Lake  0 POLYGON ((8645403 1449689, ...   36331.251
    ## 7  28043727        Varthur Lake  0 POLYGON ((8654746 1453420, ... 1527993.910
    ## 8  28069377 Horamavu Agara Lake  0 POLYGON ((8644760 1464013, ...  153583.906
    ## 9  28069384                <NA>  0 POLYGON ((8640989 1468461, ...   23054.653
    ## 10 28069395                <NA>  0 POLYGON ((8639392 1470307, ...  532738.903
    ##    totalArea
    ## 1   21425261
    ## 2   21425261
    ## 3   21425261
    ## 4   21425261
    ## 5   21425261
    ## 6   21425261
    ## 7   21425261
    ## 8   21425261
    ## 9   21425261
    ## 10  21425261

``` r
# Total area within municipal boundary = 21425261 Sqm
```

## Build Map

``` r
# Define palette

palette <- c("#caf0f8",
             "#90e0ef",
             "#00b4d8",
             "#0077b6",
             "#03045e")

# Put the map together

bangaloresNaturalWatersMap <- tm_shape(bangaloreWater) +
  tm_fill(col = "#00b4d8") +
  tm_shape(bangaloreWardBoundary) +
  tm_borders(col = "#E5E5E5",
             lwd = 3,
             alpha = 0.8) +
  tm_shape(trunkRoads) +
  tm_lines(col = "#E5E5E5",
           alpha = 0.6) +
  tm_shape(motorRoads) +
  tm_lines(col = "#E5E5E5",
           alpha = 0.6) +
  tm_shape(primaryRoads) +
  tm_lines(col = "#E5E5E5",
           alpha = 0.6) +
  tm_shape(secondaryRoads) +
  tm_lines(col = "#E5E5E5",
           alpha = 0.6) +
  tm_layout(bg.color = "#ffffff",
            frame = F,
            attr.outside = T,
            inner.margins = 0,
            outer.margins = 0,
            asp = 0,
            scale = 0.8,
            main.title = "Bangalore's Natural Waters",
            main.title.color = "#00b4d8",
            main.title.size = 1.70,
            main.title.fontface = 2,
            main.title.fontfamily = "Arial Narrow",
            title = "          39.84km² = Area of natural waters\n          21.42km² = Area of natural waters in admin boundary",
            title.color = "#00b4d8",
            title.size = 0.8,
            title.position = c("right", "bottom"),
            legend.show = F) + 
  tm_credits("#30DayMapChallenge | Day 5 | Anirudh Govind | Nov 2020\nMap data © OpenStreetMap contributors and available from https://www.openstreetmap.org",
             col = "#00b4d8",
             size = 0.8,
             position = c("left", "bottom"),
             fontfamily = "Arial Narrow")
```

## Export

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = bangaloresNaturalWatersMap,
          filename = here::here("exports/Day5.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day5.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)
