Day4
================
Anirudh Govind
(02 November, 2020)

## Get Data

``` r
# Load Bangalore wards

bangaloreWardBoundary <- read_sf(here::here("data/raw-data/bangaloreWardBoundary.shp"))

bangaloreWardBoundary <- bangaloreWardBoundary%>% 
  st_transform(3857)
```

``` r
# Load in previously downloaded data from OSM

bangaloreBuildings <- readRDS(here::here("data/raw-data/bangaloreBuildings.rds"))

bangaloreBuildings <- bangaloreBuildings %>%
  st_transform(3857)
```

## Wrangle Data

``` r
# Keep only necessary info for the buildings

bangaloreBuildings <- bangaloreBuildings %>% 
  select(osm_id, geometry)
```

``` r
# Create a hexagonal grid across Bangalore

bangaloreGrid <- st_make_grid(bangaloreWardBoundary, cellsize = 1000, square = FALSE) %>% 
  st_sf(gridID = 1:length(.), crs = 3857)

# I also want a boundary line for all the grids

bangaloreGridBoundary <- st_union(bangaloreGrid)
```

``` r
# Join building polygons to the grid

buildingsGrid <- st_join(bangaloreBuildings, bangaloreGrid) %>% 
  st_set_geometry(NULL) %>% 
  group_by(gridID) %>% 
  summarise(buildingCount = n())
```

    ## `summarise()` ungrouping output (override with `.groups` argument)

``` r
# Sanity Check

buildingsGrid %>% 
  ggplot(aes(buildingCount)) +
  geom_histogram(bins = 300)
```

![](Day4_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

``` r
# Join buildingsGrid to bangaloreGrid

bangaloreBuildingsGrid <- left_join(bangaloreGrid, buildingsGrid, by = c("gridID" = "gridID"))

# Exclude NAs

bangaloreBuildingsGrid <- bangaloreBuildingsGrid %>% 
  filter(!is.na(gridID))

# What is the spread of buildings like

mean(bangaloreBuildingsGrid$buildingCount, na.rm = T)
```

    ## [1] 654.1403

``` r
median(bangaloreBuildingsGrid$buildingCount, na.rm = T)
```

    ## [1] 399

``` r
max(bangaloreBuildingsGrid$buildingCount, na.rm = T)
```

    ## [1] 3655

``` r
min(bangaloreBuildingsGrid$buildingCount, na.rm = T)
```

    ## [1] 1

``` r
# Count number of buildings in bangalore

bangaloreBuildingsGrid %>%
  summarise(sum = sum(buildingCount, na.rm = T))
```

    ## Simple feature collection with 1 feature and 1 field
    ## geometry type:  POLYGON
    ## dimension:      XY
    ## bbox:           xmin: 8622319 ymin: 1440197 xmax: 8659319 ymax: 1476859
    ## projected CRS:  WGS 84 / Pseudo-Mercator
    ##      sum                              .
    ## 1 573681 POLYGON ((8637819 1440197, ...

``` r
# Apparently, Bangalore has 5,73,681 buildings.


buildingsGrid %>% 
  ggplot(aes(gridID, buildingCount)) +
  geom_point() +
  ylim(0, 2000) +
  coord_flip() +
  ggtitle("Distribution of buildingCounts")
```

    ## Warning: Removed 58 rows containing missing values (geom_point).

![](Day4_files/figure-gfm/unnamed-chunk-5-2.png)<!-- -->

``` r
# Would make sense to include a histogram in the plot to show the spread of points. I'd say intervals of 300 would be good. Only 58 values are above 2000.

# I also want to know the area of each grid

bangaloreGrid %>% 
  slice(1) %>% 
  st_area()
```

    ## 866025.4 [m^2]

``` r
bangaloreGridBoundary %>% 
  st_area()
```

    ## 835714515 [m^2]

## Build Map

``` r
# Define Palette

myPalette <- c("#ffebee",
               "#ffcdd2",
               "#ef9a9a",
               "#e57373",
               "#ef5350",
               "#f44336",
               "#e53935",
               "#d32f2f",
               "#c62828",
               "#b71c1c")

# Build map

bangaloresBuildingDistributionMap <- tm_shape(bangaloreBuildingsGrid) +
  tm_fill(col = "buildingCount",
          colorNA = NULL,
          palette = myPalette,
          breaks = c(0,400,800,1200,1600,2000,2400,2800,3200,4000),
          showNA = TRUE,
          title = "") +
  tm_borders(col = "white",
             lwd = 1.2) + 
  tm_text(text = "buildingCount",
          size = 0.45,
          col = "black") +
  tm_shape(bangaloreGridBoundary) +
  tm_borders(col = "#b71c1c",
             lwd = 2.7) +
  tm_layout(bg.color = "#ffffff",
            frame = F,
            attr.outside = T,
            outer.margins = 0,
            asp = 0,
            scale = 0.8,
            main.title = "How are Bangalore's 5,73,681* buildings distributed across the city?",
            main.title.color = "#b71c1c",
            main.title.size = 1.70,
            main.title.fontface = 2,
            main.title.fontfamily = "Arial Narrow",
            title = "*Approximately.\nEach hexagon has an area of 0.86km².",
            title.color = "#c62828",
            title.size = 0.8,
            title.position = c("right", "bottom"),
            legend.show = F) + 
  tm_credits("#30DayMapChallenge | Day 4 | Anirudh Govind | Nov 2020\nMap data © OpenStreetMap contributors and available from https://www.openstreetmap.org",
             col = "#b71c1c",
             size = 0.8,
             position = c("left", "bottom"),
             fontfamily = "Arial Narrow")

bangaloresBuildingDistributionMap
```

![](Day4_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

## Export

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = bangaloresBuildingDistributionMap,
          filename = here::here("exports/Day4.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day4.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)
