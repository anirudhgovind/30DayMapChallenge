Day13
================
Anirudh Govind
(13 November, 2020)

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
# Load Bangalore Population

bangalorePopulation <- raster(here::here("data/raw-data/ind_ppp_2020_UNadj_constrained.tif"))
```

## Wrangle Data

``` r
# Transform/ Make CRS consistent

bangaloreWardBoundary <- st_transform(bangaloreWardBoundary, CRS("+proj=longlat +datum=WGS84 +no_defs +ellps=WGS84 +towgs84=0,0,0"))

# Clip data

bangalorePopulationCrop <- crop(bangalorePopulation, bangaloreWardBoundary)

bangalorePopulationMask <- mask(bangalorePopulationCrop, bangaloreWardBoundary)
```

``` r
# Sanity check. The viz looks weird. The data looks off.

bangalorePopulationData <- raster::extract(bangalorePopulationMask, bangaloreWardBoundary)

bangalorePopulationData <- as.data.frame(bangalorePopulationData)

# Rename data

bangalorePopulationData <- bangalorePopulationData %>% 
  rename(pop = `c.NA..NA..NA..NA..NA..NA..NA..NA..NA..NA..NA..NA..NA..NA..NA..`)

# Bind Data

bangaloreWardPop <- bind_cols(bangaloreWardBoundary, bangalorePopulationData)

# Check population as per this

bangaloreWardPop %>% 
  summarise(total = sum(pop, na.rm = T))
```

    ## Simple feature collection with 1 feature and 1 field
    ## geometry type:  POLYGON
    ## dimension:      XY
    ## bbox:           xmin: 77.4601 ymin: 12.83401 xmax: 77.78405 ymax: 13.14367
    ## CRS:            +proj=longlat +datum=WGS84 +no_defs +ellps=WGS84 +towgs84=0,0,0
    ## # A tibble: 1 x 2
    ##       total                                                             geometry
    ##       <dbl>                                                        <POLYGON [°]>
    ## 1 11539210. ((77.5145 12.87053, 77.51363 12.87037, 77.51334 12.8705, 77.51297 1~

``` r
# According to this, Bangalore has a population of 11,539,210. Which is quite a bit lower than what it actually is.

max(bangaloreWardPop$pop, na.rm = T)
```

    ## [1] 569.9608

``` r
min(bangaloreWardPop$pop, na.rm = T)
```

    ## [1] 5.194422

``` r
mean(bangaloreWardPop$pop, na.rm = T)
```

    ## [1] 181.4512

``` r
bangaloreWardPop %>% 
  filter(pop == 0) %>% 
  view()
```

## Build Map

``` r
# Put layers together

bangalorePopulationMap <- tm_shape(bangaloreWardBoundary) +
  tm_borders(col = "#000000",
             lwd = 3) + 
  tm_shape(bangalorePopulationMask) +
  tm_raster(palette = "OrRd",
            colorNA = "#ffffff",
            breaks = c(1,60,120,180,240,300,360,420,480,540,600),
            title = "Population Counts") +
  tm_shape(bangaloreWards) +
  tm_borders(col = "#000000",
             lwd = 1.5) +
  tm_layout(bg.color = "#ffffff",
            frame = F,
            attr.outside = T,
            outer.margins = 0,
            asp = 0,
            scale = 0.8,
            main.title = "Bangalore's Estimated Population",
            main.title.color = "#000000",
            main.title.size = 1.75,
            main.title.fontface = 2,
            main.title.fontfamily = "Arial Narrow") + 
  tm_credits("#30DayMapChallenge | Day 13 | Anirudh Govind | Nov 2020\nBondarenko M., Kerr D., Sorichetta A., and Tatem, A.J. 2020. Census/projection-disaggregated gridded population datasets, adjusted to match the corresponding UNPD 2020 estimates,\nfor 183 countries in 2020 using Built-Settlement Growth Model (BSGM) outputs. WorldPop, University of Southampton, UK. doi:10.5258/SOTON/WP00685",
             col = "#000000",
             size = 0.8,
             position = c("left", "bottom"),
             fontfamily = "Arial Narrow")
```

## Export Map

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = bangalorePopulationMap,
          filename = here::here("exports/Day13.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day13.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)
