Day18
================
Anirudh Govind
(18 November, 2020)

## Load Data

``` r
# Get landuse data

# query <- getbb("Bangalore") %>%
#   opq() %>%
#   add_osm_feature("landuse")
# 
# str(query)
# 
# landuse <- osmdata_sf(query)
# 
# saveRDS(landuse, here::here("data/raw-data/landuse.rds"))

landuse <- readRDS(here::here("data/raw-data/landuse.rds"))

landuse <- landuse$osm_polygons

landuse <- landuse %>% 
  st_transform(3857)
```

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
# Load train station data

publicTransportStops <- readRDS(here::here("data/raw-data/publicTransportStops.rds"))

# Keep only relevant info

publicTransportStops <- publicTransportStops$osm_points

# Transform

publicTransportStops <- publicTransportStops %>% 
  st_transform(3857)
```

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
# Get landuse around the station

mgLanduse <- st_intersection(landuse, mgBuffer)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
mgLanduse <- mgLanduse %>% 
  select(osm_id, landuse, geometry)

mgLanduse %>% 
  group_by(landuse) %>% 
  count()
```

    ## # A tibble: 7 x 3
    ## # Groups:   landuse [7]
    ##   landuse         n                                                     geometry
    ##   <chr>       <int>                                               <GEOMETRY [m]>
    ## 1 commercial     24 MULTIPOLYGON (((8639174 1456468, 8639246 1456444, 8639226 1~
    ## 2 constructi~     1 POLYGON ((8639312 1456903, 8639262 1456919, 8639279 1456981~
    ## 3 government      1 POLYGON ((8639549 1456810, 8639607 1456796, 8639618 1456787~
    ## 4 grass           7 MULTIPOLYGON (((8639180 1456555, 8639195 1456544, 8639180 1~
    ## 5 military        5 MULTIPOLYGON (((8639371 1457093, 8639386 1457097, 8639641 1~
    ## 6 residential     1 POLYGON ((8638938 1456817, 8639007 1456797, 8638993 1456752~
    ## 7 retail          6 MULTIPOLYGON (((8639374 1456647, 8639354 1456693, 8639328 1~

``` r
mgLanduse <- mgLanduse %>% 
  mutate(fillColour = case_when(landuse == "commercial" ~ "#dee2e6",
                                landuse == "construction" ~ "#ced4da",
                                landuse == "government" ~ "#adb5bd",
                                landuse == "grass" ~ "#6c757d",
                                landuse == "military" ~ "#495057",
                                landuse == "residential" ~ "#343a40",
                                landuse == "retail" ~ "#212529"))

mgLanduse <- mgLanduse %>% 
  mutate(fillColourAlt = case_when(landuse == "commercial" ~ "#f94144",
                                landuse == "construction" ~ "#f3722c",
                                landuse == "government" ~ "#f8961e",
                                landuse == "grass" ~ "#f9c74f",
                                landuse == "military" ~ "#90be6d",
                                landuse == "residential" ~ "#43aa8b",
                                landuse == "retail" ~ "#577590"))
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
# Context buildings

mgContextBuildings <- st_intersection(bangaloreBuildings, mgBuffer)
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

## Build Map

``` r
# Palette

# c("#000000", "#14213d", "#fca311", "#e5e5e5", "#ffffff")

# Context Buildings

mgMapContext <- mgMapAreaBuildings %>% 
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
            title = "Landuse around Metro Station (Indicative colours)",
            title.color = "#000000",
            title.size = 1,
            title.position = c("left", "TOP"),
            title.fontface = 2) + 
  tm_credits("#30DayMapChallenge | Day 18 | Anirudh Govind | Nov 2020\nMap data © OpenStreetMap contributors and available from https://www.openstreetmap.org",
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

mgMap500Buildings <- mgContextBuildings %>% 
  tm_shape() +
  tm_fill(col = "#000000",
          alpha = 0.35)

# Metro station

mapMGMetroStation <- mgContextBuildings %>% 
  filter(osm_id == 348854341) %>% 
  tm_shape() +
  tm_fill(col = "#8b1c7a")

# Land Use

mapMGRoadLanduse <- mgLanduse %>% 
  tm_shape() +
  tm_fill(col = "fillColour",
          alpha = 0.7)

mapMGRoadLanduseAlt <- mgLanduse %>% 
  tm_shape() +
  tm_fill(col = "fillColourAlt",
          alpha = 0.7)

mapMGRoad <- mgMapContext + 
  mgMapContextRoads + 
  mgMap500Roads + 
  mgMap500Buffer + 
  mgMap500Buildings +
  mapMGMetroStation +
  mapMGRoadLanduse

mapMGRoadAlt <- mgMapContext + 
  mgMapContextRoads + 
  mgMap500Roads + 
  mgMap500Buffer + 
  mgMap500Buildings +
  mapMGMetroStation +
  mapMGRoadLanduseAlt
```

## Export

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = mapMGRoad,
          filename = here::here("exports/Day18-1.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day18-1.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = mapMGRoadAlt,
          filename = here::here("exports/Day18-2.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day18-2.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)
