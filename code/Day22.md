Day22
================
Anirudh Govind
(22 November, 2020)

## Get Data

``` r
# Load Bangalore ward boundaries

bangaloreWardBoundary <- read_sf(here::here("data/raw-data/bangaloreWardBoundary.shp"))

bangaloreWardBoundary <- bangaloreWardBoundary%>% 
  st_transform(3857)
```

``` r
# Get bus stop data from OSM
#  
# query <- getbb("Bangalore") %>%
#   opq() %>%
#   add_osm_feature("highway", "bus_stop")
# 
# str(query)
# 
# busStops <- osmdata_sf(query)
# 
# saveRDS(busStops, here::here("data/raw-data/busStops.rds"))

busStops <- readRDS(here::here("data/raw-data/busStops.rds"))
```

``` r
# Load roads data (previously saved from OSM and cleaned up)

bangaloreRoads <- readRDS(here::here("data/derived-data/bangaloreRoads.rds"))
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

## Wrangle data

``` r
# Intersect roads and ward boundary

bangaloreRoads <- st_intersection(bangaloreRoads, bangaloreWardBoundary)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# I'll exclude residential roads from this

bangaloreRoads <- bangaloreRoads %>% 
  filter(highway != "residential")
```

``` r
# Extract point locations

bangaloreBusStops <- busStops$osm_points

# Transform data

bangaloreBusStops <- bangaloreBusStops %>% 
  st_transform(3857)

# Clip to boundary

bangaloreBusStops <- st_intersection(bangaloreBusStops, bangaloreWardBoundary)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# Get all the metro station locations

publicTransportStops <- publicTransportStops$osm_points

metroStations <- publicTransportStops %>% 
  filter(network == "Namma Metro")

# Keep only relevant info

metroStations <- metroStations %>% 
  select(osm_id, name, geometry) %>% 
  st_transform(3857)

metroStations <- metroStations %>% 
  arrange(name)

# I'm just going to keep one point for each station

metroStations <- metroStations %>% 
  group_by(name) %>% 
  slice(1)

# The Majestic station is repeated so I'll filter manually by name

metroStations <- metroStations %>% 
  filter(name != "Nadaprabhu Kempegowda Station, Majestic (Green Line)")

metroStations <- metroStations %>% 
  ungroup() %>% 
  st_as_sf() %>% 
  st_transform(3857)
```

``` r
# Now I want to identify the bus stops which are in a 500m buffer of the metro stations. First, I'll draw a buffer around all the metro stations

metroStationsBuffer <- st_buffer(metroStations, 1000)

metroStationsBufferU <- st_union(metroStationsBuffer)

# Find the bus stops within a 500 radius

busStopsInBuffer <- st_intersection(bangaloreBusStops, metroStationsBuffer)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
busStopsInBuffer <- busStopsInBuffer %>% 
  select(osm_id, geometry, `name.1`) %>% 
  rename(stationName = `name.1`)

busStopsInBuffer %>% 
  group_by(stationName) %>% 
  count(osm_id) %>% 
  summarise(count = sum(n)) %>% 
  arrange(count) %>% 
  view()
```

    ## `summarise()` ungrouping output (override with `.groups` argument)

## Build Map

``` r
# Put map layers together

mapBoundary <- tm_shape(bangaloreWardBoundary) + 
  tm_borders(col = "#000000",
             lwd = 3.0) + 
  tm_layout(bg.color = "white",
            frame = F,
            frame.lwd = NA,
            attr.outside = T,
            outer.margins = 0,
            asp = 0,
            scale = 0.8,
            main.title = "How many bus stops are less than 1km away from a metro station?",
            main.title.color = "#000000",
            main.title.size = 1.75,
            main.title.fontfamily = "Arial Narrow",
            title = "906 bus stops are within 1km of a metro station",
            title.color = "#000000",
            title.size = 0.9,
            title.position = c("left", "TOP"),
            title.fontface = 2) + 
  tm_credits("#30DayMapChallenge | Day 22 | Anirudh Govind | Nov 2020\nMap data © OpenStreetMap contributors and available from https://www.openstreetmap.org",
             col = "#000000",
             size = 0.8,
             position = c("left", "bottom"),
             fontfamily = "Arial Narrow")

# Roads in buffer

mapTrunk <- bangaloreRoads %>% 
  filter(highway == "trunk") %>% 
  tm_shape() +
  tm_lines(col = "#e5e5e5",
           lwd = 0.1,
          alpha = 0.5)

mapMotorway <- bangaloreRoads %>% 
  filter(highway == "motorway") %>% 
  tm_shape() +
  tm_lines(col = "#e5e5e5",
           lwd = 0.1,
          alpha = 0.5)

mapPrimary <- bangaloreRoads %>% 
  filter(highway == "primary") %>% 
  tm_shape() +
  tm_lines(col = "#e5e5e5",
           lwd = 0.1,
          alpha = 0.5)

mapSecondary <- bangaloreRoads %>% 
  filter(highway == "secondary") %>% 
  tm_shape() +
  tm_lines(col = "#e5e5e5",
           lwd = 0.1,
          alpha = 0.5)

# Metro stations

mapStations <- tm_shape(metroStations) +
  tm_dots(size = 0.5,
          col = "#000000",
          shape = 22,
          jitter = 0)

# Station buffer

mapStationsBuffer <- tm_shape(metroStationsBufferU) +
  tm_borders(col = "#fd151b",
             lwd = 1.5)

# Bus stops in buffer

mapStops <- tm_shape(busStopsInBuffer) +
  tm_dots(size = 0.1,
          col = "#fd151b",
          shape = 20,
          jitter = 0)

# Other bus stops

mapBusStops <- tm_shape(bangaloreBusStops) +
  tm_dots(size = 0.1,
          col = "#000000",
          shape = 20,
          jitter = 0,
          alpha = 0.2)

mapDistance <- mapBoundary + mapTrunk + mapMotorway + mapPrimary + mapSecondary + mapStationsBuffer + mapStations  + mapStops + mapBusStops
```

## Export

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = mapDistance,
          filename = here::here("exports/Day22.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day22.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)
