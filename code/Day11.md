Day11
================
Anirudh Govind
(11 November, 2020)

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
# Join buildingsGrid to bangaloreGrid

bangaloreBuildingsGrid <- left_join(bangaloreGrid, buildingsGrid, by = c("gridID" = "gridID"))

# Exclude NAs

bangaloreBuildingsGrid <- bangaloreBuildingsGrid %>% 
  filter(!is.na(gridID))
```

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

# Layers

plot <- bangaloreBuildingsGrid %>% 
  ggplot(aes(fill = buildingCount)) +
  geom_sf() +
  scale_fill_viridis_c(option = "C") +
  ggtitle("Bangalore's Building Distribution")

# Make 3D Plot

plot_gg(plot,
        width = 8,
        height = 8,
        scale = 300,
        multicore = T,
        raytrace = T)
```
