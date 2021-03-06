Day14
================
Anirudh Govind
(14 November, 2020)

## Load Data

``` r
# Load Bangalore ward boundary

bangaloreWardBoundary <- read_sf(here::here("data/raw-data/bangaloreWardBoundary.shp"))

bangaloreWardBoundary <- bangaloreWardBoundary%>% 
  st_transform(3857)
```

``` r
# Load Bangalore wards

bangaloreWards <- read_sf(here::here("data/raw-data/bangaloreWardsUTM.shp"))

bangaloreWards <- bangaloreWards %>% 
  st_transform(3857)
```

``` r
# Load lake redevelopment data. This is data prepared during my masters thesis.

lakesKLCDAByWard_sf <- readRDS(here::here("data/derived-data/lakesKLCDAByWard_sf.rds"))

lakesKLCDA <- lakesKLCDAByWard_sf %>% 
  st_transform(3857) %>% 
  ungroup() %>% 
  st_as_sf()
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

``` r
# Load previously prepared data about Bangalore lakes

bangaloreWater <- readRDS(here::here("data/derived-data/bangaloreWater.rds"))
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

## Build Map

``` r
# The map will be built on a previously made map showing lakes. In this one I will use symbols to show which lakes are still around and which ones are gone.

# Define palette

#"#e63946" red

#"#fcbf49" amber

redevelopedLakesMap <- lakesKLCDA %>% 
  filter(redeveloped == "yes") %>% 
  tm_shape() +
  tm_symbols(col = "#e63946",
             shape = 4,
             border.lwd = 3)

# Define palette

palette <- c("#caf0f8",
             "#90e0ef",
             "#00b4d8",
             "#0077b6",
             "#03045e")


# Put the map together

bangaloresLakesMap <- tm_shape(bangaloreWater) +
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
            main.title = "Bangalore's Lakes",
            main.title.color = "#00b4d8",
            main.title.size = 1.70,
            main.title.fontface = 2,
            main.title.fontfamily = "Arial Narrow",
            title = "X's indicate lakes which are no longer present\nor visible through recent satellite imagery as\nseen by the author in 2020",
            title.color = "#00b4d8",
            title.size = 0.8,
            title.position = c("right", "bottom"),
            legend.show = F) + 
  tm_credits("#30DayMapChallenge | Day 14 | Anirudh Govind | Nov 2020\nMap data © OpenStreetMap contributors and available from https://www.openstreetmap.org\nList of lakes and locations from Ministry of Environment & Forests, Govt. of India Copyright (c) 2011. All rights reserved and available from http://www.karenvis.nic.in/Content/GeospatialData_8077.aspx",
             col = "#00b4d8",
             size = 0.8,
             position = c("left", "bottom"),
             fontfamily = "Arial Narrow")

bangaloresLakesStatusMap <- bangaloresLakesMap + redevelopedLakesMap
```

## Export

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = bangaloresLakesStatusMap,
          filename = here::here("exports/Day14.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day14.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)
