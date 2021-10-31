Day28
================
Anirudh Govind
(28 November, 2020)

## Get Data

``` r
# Load Bangalore wards

bangaloreWardBoundary <- read_sf(here::here("data/raw-data/bangaloreWardBoundary.shp"))

bangaloreWardBoundary <- bangaloreWardBoundary%>% 
  st_transform(3857)
```

``` r
# Get area name data from OSM

# query <- getbb("Bangalore") %>% 
#   opq() %>% 
#   add_osm_feature("place", "suburb")
# 
# str(query)
# 
# areaNames <- osmdata_sf(query)
# 
# saveRDS(areaNames, here::here("data/raw-data/areaNames.rds"))

areaNames <- readRDS(here::here("data/raw-data/areaNames.rds"))
```

## Wrangle Data

``` r
# Keep only relevant data

areaNames <- areaNames$osm_points

areaNames <- areaNames %>% 
  select(osm_id, name, geometry)

# Exclude areas without data for names

areaNames <- areaNames %>% 
  filter(name != "NA")

# Exlcude data errors

areaNames <- areaNames %>% 
  filter(!name == "Book World") %>% 
  filter(!name == "Taaza Mitai") %>% 
  filter(!name == "Nalli Silks")

areaNames <- areaNames %>% 
  st_transform(3857)
```

``` r
# Draw a Bangalore-sized circle

bangaloreCentroid <- st_centroid(bangaloreWardBoundary)
```

    ## Warning in st_centroid.sf(bangaloreWardBoundary): st_centroid assumes attributes
    ## are constant over geometries of x

``` r
bangaloreBuffer <- st_buffer(bangaloreCentroid, 15500, endCapStyle = "SQUARE")

st_area(bangaloreBuffer)
```

    ## 9.61e+08 [m^2]

``` r
st_area(bangaloreWardBoundary)
```

    ## 753051105 [m^2]

``` r
# By trial and error, a radiusm of 15500m looks about right. Areas of the buffer and the ward boundary are similar

tm_shape(bangaloreWardBoundary) +
  tm_borders() +
  tm_shape(bangaloreBuffer) +
  tm_borders() +
  tm_shape(areaNames) +
  tm_dots()
```

![](Day28_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

``` r
# Include Bangalore's Centroid in the List of areaNames

bangaloreCentroid <- bangaloreCentroid %>% 
  rename(osm_id = id) %>%
  mutate(osm_id = as.character(osm_id)) %>% 
  mutate(name = "Bangalore") %>% 
  select(osm_id, name, geometry)

# A few area names are outside this circle and I will exclude them

areaNames <- st_intersection(areaNames, bangaloreWardBoundary)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
areaNames <- areaNames %>% 
  select(-id)

glimpse(bangaloreCentroid)
```

    ## Rows: 1
    ## Columns: 3
    ## $ osm_id   <chr> "0"
    ## $ name     <chr> "Bangalore"
    ## $ geometry <POINT [m]> POINT (8638788 1457312)

``` r
glimpse(areaNames)
```

    ## Rows: 214
    ## Columns: 3
    ## $ osm_id   <chr> "308715244", "347261886", "347262164", "378717492", "41086...
    ## $ name     <chr> "Vidyaranyapura", "Domlur", "Kodihalli", "Brookefields", "...
    ## $ geometry <POINT [m]> POINT (8633687 1468490), POINT (8642644 1455445), PO...

``` r
areaNames <- bind_rows(areaNames, bangaloreCentroid)

# Draw voronoi polygons

areaNamesUnion <- st_union(areaNames)

areaNamesVoronoi <- st_voronoi(areaNamesUnion)

# Clip polygons

areaNamesVoronoi <- st_intersection(st_cast(areaNamesVoronoi), bangaloreBuffer)

areaNamesVoronoi <- areaNamesVoronoi %>% 
  st_as_sf() %>% 
  mutate(id = row_number())

# I'm going to try and smoothen the different polygons

areaNamesVoronoiSmooth <- smooth(areaNamesVoronoi, method = "chaikin")

# Attach names to each of the polygons

areaNamesNew <- st_intersection(areaNames, areaNamesVoronoi) %>% 
  st_set_geometry(NULL)
```

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

``` r
# Get centroids for each of the polygons

areaNamesCentroids <- st_centroid(areaNamesVoronoi) %>% 
  st_as_sf()
```

    ## Warning in st_centroid.sf(areaNamesVoronoi): st_centroid assumes attributes are
    ## constant over geometries of x

``` r
# Join the centroids to the area names

areaNamesNew <- left_join(areaNamesNew, areaNamesCentroids, by = c("id" = "id"))

areaNamesNew <- areaNamesNew %>% 
  st_as_sf()

# Now link the names back to voronoi polygons so that I can size text based on the size of the polygon

areaNamesNew <- st_join(areaNamesVoronoi, areaNamesNew)

areaNamesNew <- areaNamesNew %>% 
  filter(name != "NA")

# Define function to wrap text

# Core wrapping function
wrap.it <- function(x, len)
{ 
  sapply(x, function(y) paste(strwrap(y, len), 
                              collapse = "\n"), 
         USE.NAMES = FALSE)
}

# Call this function with a list or vector
wrap.labels <- function(x, len)
{
  if (is.list(x))
  {
    lapply(x, wrap.it, len)
  } else {
    wrap.it(x, len)
  }
}

# Wrap text

areaNamesNew <- areaNamesNew %>% 
  mutate(name2 = wrap.labels(name, 5))

# Calculate areas

areaNamesVoronoiSmooth <- areaNamesVoronoiSmooth %>% 
  mutate(area = st_area(.))

max(areaNamesVoronoiSmooth$area)
```

    ## 50835715 [m^2]

``` r
min(areaNamesVoronoiSmooth$area)
```

    ## 267344.1 [m^2]

``` r
mean(areaNamesVoronoiSmooth$area)
```

    ## 4065309 [m^2]

## Build Map

``` r
# Put layers together

myPalette <- c("#fec89a",
               "#f9dcc4",
               "#f8edeb",
               "#fcd5ce",
               "#ffb5a7")

# Highlight bangalire

mapBangalore <- areaNamesVoronoiSmooth %>% 
  filter(id == 110) %>% 
  tm_shape() +
  tm_borders(col = "#e77377",
             lwd = 3) +
  tm_fill(col = "#e77377") 


mapBangaloreText <- areaNamesNew %>% 
  filter(id.x == 110) %>% 
  tm_shape() +
  tm_text(text = "name2",
          size = 0.55,
          print.tiny = T,
          just = c("center", "center"),
          showNA = F,
          textNA = " ",
          case = "upper",
          fontface = 2)

mapBangaloresAreas <- tm_shape(bangaloreBuffer) +
  tm_fill(col = "#000000") +
  tm_borders(col = "#000000",
             lwd = 8) +
  tm_shape(areaNamesVoronoiSmooth) +
  tm_fill(col = "MAP_COLORS",
          palette = myPalette,
          legend.show = F) +
  # tm_borders(col = "#000000") +
  tm_shape(areaNamesNew) +
  tm_text(text = "name2",
          size = "AREA",
          print.tiny = T,
          just = c("center", "center"),
          showNA = F,
          textNA = " ") + 
  tm_layout(bg.color = "#ffffff",
            frame = F,
            frame.lwd = NA,
            attr.outside = T,
            outer.margins = 0,
            asp = 0,
            scale = 0.8,
            main.title = "Bangalore: A Conceptual Map",
            main.title.color = "#000000",
            main.title.size = 1.75,
            main.title.fontfamily = "Arial Narrow") + 
  tm_credits("#30DayMapChallenge | Day 28 | Anirudh Govind | Nov 2020\nMap data © OpenStreetMap contributors and available from https://www.openstreetmap.org",
             col = "#000000",
             size = 0.9,
             position = c("left", "bottom"),
             fontfamily = "Arial Narrow")

mapConceptual <- mapBangaloresAreas + mapBangalore + mapBangaloreText
```

## Export Map

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = mapConceptual,
          filename = here::here("exports/Day28.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Warning: Unable to color with 5 colors. Adjacent polygons may have the same
    ## color.

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day28.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)
