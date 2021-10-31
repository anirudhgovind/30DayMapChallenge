Day20
================
Anirudh Govind
(20 November, 2020)

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
# Load Bangalore Population Density

bangalorePopulationDensity <- raster(here::here("data/raw-data/ind_pd_2020_1km_UNadj.tif"))
```

## Wrangle Data

``` r
# Transform/ Make CRS consistent

bangaloreWardBoundary <- st_transform(bangaloreWardBoundary, CRS("+proj=longlat +datum=WGS84 +no_defs +ellps=WGS84 +towgs84=0,0,0"))

# Clip data

bangalorePopulationDensityCrop <- crop(bangalorePopulationDensity, bangaloreWardBoundary)

bangalorePopulationDensityMask <- mask(bangalorePopulationDensityCrop, bangaloreWardBoundary)

# Sanity checks

bangalorePopulationDensityData <- raster::extract(bangalorePopulationDensityMask,
                                                  bangaloreWardBoundary)

bangalorePopulationDensityData <- as.data.frame(bangalorePopulationDensityData)

# Rename data

bangalorePopulationDensityData <- bangalorePopulationDensityData %>% 
  rename(popDensity = `c.9044.794921875..9241.0068359375..9014.2119140625..10318.2880859375..`)

# Check Spread

max(bangalorePopulationDensityData$popDensity, na.rm = T)
```

    ## [1] 64664.96

``` r
min(bangalorePopulationDensityData$popDensity, na.rm = T)
```

    ## [1] 48.74282

``` r
median(bangalorePopulationDensityData$popDensity, na.rm = T)
```

    ## [1] 2146.317

``` r
bangalorePopulationDensityData %>% 
  ggplot() +
  geom_histogram(aes(popDensity), 
                 bins = 150) +
  geom_vline(xintercept = 500)
```

![](Day20_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

## Build Map

``` r
# Put layers together

bangalorePopulationDensityMap <- tm_shape(bangaloreWardBoundary) +
  tm_borders(col = "#000000",
             lwd = 3) + 
  tm_shape(bangalorePopulationDensityMask) +
  tm_raster(palette = "Blues",
            colorNA = "#ffffff",
            colorNULL = "#ffffff",
            breaks = c(0,500,1000,2000,4000,8000,16000,32000,64000),
            title = "Population Density",
            style = "cont",
            showNA = F) +
  tm_shape(bangaloreWards) +
  tm_borders(col = "#000000",
             lwd = 1.5) +
  tm_layout(bg.color = "#ffffff",
            frame = F,
            attr.outside = T,
            outer.margins = 0,
            asp = 0,
            scale = 0.8,
            main.title = "Bangalore's Estimated Population Density",
            main.title.color = "#000000",
            main.title.size = 1.75,
            main.title.fontface = 2,
            main.title.fontfamily = "Arial Narrow") + 
  tm_credits("#30DayMapChallenge | Day 20 | Anirudh Govind | Nov 2020 | WorldPop (www.worldpop.org - School of Geography and Environmental Science\nUniversity of Southampton; Department of Geography and Geosciences, University of Louisville;Departement de Geographie, Universite de Namur) and\nCenter for International Earth Science Information Network (CIESIN), Columbia University (2018). Global High Resolution Population Denominators Project\nFunded by The Bill and Melinda Gates Foundation (OPP1134076). https://dx.doi.org/10.5258/SOTON/WP00675",
             col = "#000000",
             size = 0.7,
             position = c("left", "bottom"),
             fontfamily = "Arial Narrow")
```

## Export Map

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = bangalorePopulationDensityMap,
          filename = here::here("exports/Day20.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Warning: Values have found that are higher than the highest break

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day20.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)
