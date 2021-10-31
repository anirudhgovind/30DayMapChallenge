Day3
================
Anirudh Govind
(02 November, 2020)

## Get Data

``` r
# Load Bangalore wards

bangaloreWards <- read_sf(here::here("data/raw-data/bangaloreWardsUTM.shp"))

bangaloreWards <- bangaloreWards %>% 
  st_transform(3857)
```

``` r
# Get traffic light location data from OSM

# query <- getbb("Bangalore") %>% 
#   opq() %>% 
#   add_osm_feature("building")
# 
# str(query)
# 
# buildings <- osmdata_sf(query)
# 
# buildingsData <- buildings$osm_polygons
# 
# saveRDS(buildingsData, here::here("data/raw-data/bangaloreBuildings.rds"))

# Load in previously downloaded data from OSM
# 
# bangaloreBuildings <- readRDS(here::here("data/raw-data/bangaloreBuildings.rds"))
# 
# bangaloreBuildings <- bangaloreBuildings %>% 
#   st_transform(3857)
```

## Wrangle Data

``` r
# # Get centroids of each ward
# 
# wardCentroids <- st_centroid(bangaloreWards)
# 
# # Draw a buffer around each centroid of 500m
# 
# wardBuffers <- wardCentroids %>%
#   st_buffer(500)
# 
# # Union around overlapping buffers
# 
# wardBuffers <- wardBuffers %>%
#   st_union()
# 
# 
# # Intersect the buffers with the polygons
# 
# buildingsInBuffer <- st_intersection(bangaloreBuildings, wardBuffers)
# 
# buildingsInBuffer <- buildingsInBuffer %>% 
#   select(osm_id, geometry)
# 
# buildingsInBuffer <- buildingsInBuffer %>% 
#   st_transform(3857)
# 
# glimpse(buildingsInBuffer)
# 
# buildingsInBuffer <- st_intersection(bangaloreWards, buildingsInBuffer)

# I'll save `buildingsInBuffer` as an RDS to cut down on processing time.
# 
# saveRDS(buildingsInBuffer, here::here("data/derived-data/buildingsInBuffer.rds"))

buildingsInBuffer <- readRDS(here::here("data/derived-data/buildingsInBuffer.rds"))
```

## Build Map and Export

``` r
# 2,7,17,37,59,97,138,173,191

# So for each ward, I need the 500m boundary and then the fabric within it.
mapFunction <- function(wardNumber1, 
                         wardNumber2, 
                         wardNumber3, 
                         wardNumber4, 
                         wardNumber5, 
                         wardNumber6, 
                         wardNumber7, 
                         wardNumber8, 
                         wardNumber9) {
  
  buffers <- bangaloreWards %>% 
  filter(ward_no == wardNumber1 | 
           ward_no == wardNumber2 | 
           ward_no == wardNumber3 | 
           ward_no == wardNumber4 | 
           ward_no == wardNumber5 | 
           ward_no == wardNumber6 | 
           ward_no == wardNumber7 | 
           ward_no == wardNumber8 | 
           ward_no == wardNumber9) %>% 
  st_centroid() %>% 
  st_buffer(500) 
  
  buildings <- buildingsInBuffer %>% 
  filter(ward_no == wardNumber1 |
           ward_no == wardNumber2 |
           ward_no == wardNumber3 |
           ward_no == wardNumber4 |
           ward_no == wardNumber5 |
           ward_no == wardNumber6 |
           ward_no == wardNumber7 |
           ward_no == wardNumber8 |
           ward_no == wardNumber9)
  
  selectedBuildings <- st_intersection(buffers, buildings)
  
  tmObject <- bangaloreWards %>% 
  filter(ward_no == wardNumber1 | 
           ward_no == wardNumber2 | 
           ward_no == wardNumber3 | 
           ward_no == wardNumber4 | 
           ward_no == wardNumber5 | 
           ward_no == wardNumber6 | 
           ward_no == wardNumber7 | 
           ward_no == wardNumber8 | 
           ward_no == wardNumber9) %>% 
  st_centroid() %>% 
  st_buffer(500) %>% 
  tm_shape() +
  tm_borders(col = "#e63946",
             lwd = 3) +
    tm_facets(by = "ward_name",
              ncol = 3,
              nrow = 3) +
  selectedBuildings %>% 
  tm_shape() +
  tm_fill(col = "#e63946") +
    tm_facets(by = "ward_name",
              ncol = 3,
              nrow = 3) +
  tm_layout(bg.color = "white",
            fontface = 2,
            frame = F,
            frame.lwd = NA,
            attr.outside = T,
            outer.margins = 0,
            asp = 0,
            scale = 0.8,
            main.title = "Bangalore in 7.02km²",
            main.title.color = "#e63946",
            main.title.size = 1.75,
            main.title.fontfamily = "Bookman Old Style",
            panel.label.color = "#e63946",
            panel.label.size = 1.35,
            panel.label.fontfamily = "Bookman Old Style",
            panel.label.bg.color = "white") + 
  tm_credits("#30DayMapChallenge | Day 3 | Anirudh Govind | Nov 2020\nMap data © OpenStreetMap contributors and available from https://www.openstreetmap.org",
             col = "#e63946",
             size = 1.2,
             position = c("left", "bottom"),
             fontfamily = "Bookman Old Style")
  
  tmap_save(tm = tmObject,
          filename = here::here("exports/Day3.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
}
```

``` r
# While the sample numbers produce one version of the poster, they contain only nine wards. I'd like to be able to have many versions of the map with a random sequence of ward numbers. I'll use a simple random-ish method to generate 9 ward numbers which I can then paste into my function and run.

noquote(toString(sample(1:198, 9)))
```

    ## [1] 87, 186, 152, 144, 82, 93, 158, 75, 115

``` r
mapFunction(37, 67, 62, 137, 144, 13, 197, 24, 184)
```

    ## Warning in st_centroid.sf(.): st_centroid assumes attributes are constant over
    ## geometries of x

    ## Warning: attribute variables are assumed to be spatially constant throughout all
    ## geometries

    ## Warning in st_centroid.sf(.): st_centroid assumes attributes are constant over
    ## geometries of x

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day3.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)
