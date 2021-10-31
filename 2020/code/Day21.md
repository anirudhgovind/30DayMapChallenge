Day21
================
Anirudh Govind
(21 November, 2020)

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
# Load Bangalore tax data

bangaloreWaterTax <- read.csv(here::here("data/raw-data/Water_Revenue_Data_Bengaluru_2013-14to2017-18.csv"))
```

## Wrange Data

``` r
# Clean up water tax data and make it tidy

bangaloreWaterTax <- bangaloreWaterTax %>% 
  select(-City.Name)

# Rename columns

bangaloreWaterTax <- bangaloreWaterTax %>% 
  rename(zone = Zone.Name,
         ward_name = Ward.Name,
         ward_no = Ward.No.,
         b_total_13_14 = Water.tax.billed..in.INR.lakhs..Total.2013.14,
         b_com_13_14 = Water.tax.billed..in.INR.lakhs..Commercial.2013.14,
         b_res_13_14 = Water.tax.billed..in.INR.lakhs..Residential.2013.14,
         b_total_14_15 = Water.tax.billed..in.INR.lakhs..Total.2014.15,
         b_com_14_15 = Water.tax.billed..in.INR.lakhs..Commercial.2014.15,
         b_res_14_15 = Water.tax.billed..in.INR.lakhs..Residential.2014.15,
         b_total_15_16 = Water.tax.billed..in.INR.lakhs..Total.2015.16,
         b_com_15_16 = Water.tax.billed..in.INR.lakhs..Commercial.2015.16,
         b_res_15_16 = Water.tax.billed..in.INR.lakhs..Residential.2015.16,
         b_total_16_17 = Water.tax.billed..in.INR.lakhs..Total.2016.17,
         b_com_16_17 = Water.tax.billed..in.INR.lakhs..Commercial.2016.17,
         b_res_16_17 = Water.tax.billed..in.INR.lakhs..Residential.2016.17,
         b_total_17_18 = Water.tax.billed..in.INR.lakhs..Total.2017.18,
         b_com_17_18 = Water.tax.billed..in.INR.lakhs..Commercial.2017.18,
         b_res_17_18 = Water.tax.billed..in.INR.lakhs..Residential.2017.18,
         c_total_13_14 = Water.tax.collected..in.INR.lakhs..Total.2013.14,
         c_com_13_14 = Water.tax.collected..in.INR.lakhs..Commercial.2013.14,
         c_res_13_14 = Water.tax.collected..in.INR.lakhs..Residential.2013.14,
         c_total_14_15 = Water.tax.collected..in.INR.lakhs..Total.2014.15,
         c_com_14_15 = Water.tax.collected..in.INR.lakhs..Commercial.2014.15,
         c_res_14_15 = Water.tax.collected..in.INR.lakhs..Residential.2014.15,
         c_total_15_16 = Water.tax.collected..in.INR.lakhs..Total.2015.16,
         c_com_15_16 = Water.tax.collected..in.INR.lakhs..Commercial.2015.16,
         c_res_15_16 = Water.tax.collected..in.INR.lakhs..Residential.2015.16,
         c_total_16_17 = Water.tax.collected..in.INR.lakhs..Total.2016.17,
         c_com_16_17 = Water.tax.collected..in.INR.lakhs..Commercial.2016.17,
         c_res_16_17 = Water.tax.collected..in.INR.lakhs..Residential.2016.17,
         c_total_17_18 = Water.tax.collected..in.INR.lakhs..Total.2017.18,
         c_com_17_18 = Water.tax.collected..in.INR.lakhs..Commercial.2017.18,
         c_res_17_18 = Water.tax.collected..in.INR.lakhs..Residential.2017.18)

# Merge and Unite Columns

bangaloreWaterTax <- bangaloreWaterTax %>% 
  unite(b_total_13_14, 
        b_res_13_14, 
        b_com_13_14, 
        c_total_13_14, 
        c_res_13_14, 
        c_com_13_14, 
        col = "2013-2014", 
        sep = "/", 
        remove = TRUE)

bangaloreWaterTax <- bangaloreWaterTax %>% 
  unite(b_total_14_15, 
        b_res_14_15, 
        b_com_14_15, 
        c_total_14_15, 
        c_res_14_15, 
        c_com_14_15, 
        col = "2014-2015", 
        sep = "/", 
        remove = TRUE)

bangaloreWaterTax <- bangaloreWaterTax %>% 
  unite(b_total_15_16, 
        b_res_15_16, 
        b_com_15_16, 
        c_total_15_16, 
        c_res_15_16, 
        c_com_15_16, 
        col = "2015-2016", 
        sep = "/", 
        remove = TRUE)

bangaloreWaterTax <- bangaloreWaterTax %>% 
  unite(b_total_16_17, 
        b_res_16_17, 
        b_com_16_17, 
        c_total_16_17, 
        c_res_16_17, 
        c_com_16_17, 
        col = "2016-2017", 
        sep = "/", 
        remove = TRUE)

bangaloreWaterTax <- bangaloreWaterTax %>% 
  unite(b_total_17_18, 
        b_res_17_18, 
        b_com_17_18, 
        c_total_17_18, 
        c_res_17_18, 
        c_com_17_18, 
        col = "2017-2018", 
        sep = "/", 
        remove = TRUE)

# Gather columns

bangaloreWaterTax <- bangaloreWaterTax %>% 
  gather(`2013-2014`, 
         `2014-2015`, 
         `2015-2016`, 
         `2016-2017`, 
         `2017-2018`, 
         key = "year", 
         value = "revenue")

# Separate

bangaloreWaterTax <- bangaloreWaterTax %>% 
  separate(revenue, 
           sep = "/", 
           into = c("b_tot", "b_res", "b_com", "c_tot", "c_res", "c_com"))

# Keep only necessary columns

bangaloreWaterTax <- bangaloreWaterTax %>% 
  select(zone, ward_no, year, b_tot, b_res, b_com, c_tot, c_res, c_com)
```

``` r
# Join to spatial data

bangaloreWaterTaxSF <- left_join(bangaloreWaterTax, bangaloreWards, 
                                 by = c("ward_no" = "ward_no"))

bangaloreWaterTaxSF <- bangaloreWaterTaxSF %>% 
  st_as_sf() %>% 
  st_transform(3857)

# Filter for the most recent four years

bangaloreWaterTaxSF <- bangaloreWaterTaxSF %>% 
  filter(year == "2014-2015" |
         year == "2015-2016" |
         year == "2016-2017" |
         year == "2017-2018")

bangaloreWaterTaxSF <- bangaloreWaterTaxSF %>% 
  select(zone, ward_no, year, b_tot, c_tot, geometry) %>% 
  mutate(b_tot = as.numeric(b_tot)) %>% 
  mutate(b_tot = round(b_tot, 2)) %>% 
  mutate(c_tot = as.numeric(c_tot)) %>% 
  mutate(c_tot = round(c_tot, 2)) %>% 
  mutate(diff = b_tot - c_tot)
```

    ## Warning: Problem with `mutate()` input `b_tot`.
    ## i NAs introduced by coercion
    ## i Input `b_tot` is `as.numeric(b_tot)`.

    ## Warning in mask$eval_all_mutate(dots[[i]]): NAs introduced by coercion

    ## Warning: Problem with `mutate()` input `c_tot`.
    ## i NAs introduced by coercion
    ## i Input `c_tot` is `as.numeric(c_tot)`.

    ## Warning in mask$eval_all_mutate(dots[[i]]): NAs introduced by coercion

``` r
mean(bangaloreWaterTaxSF$diff, na.rm = T)
```

    ## [1] 3.147513

``` r
max(bangaloreWaterTaxSF$diff, na.rm = T)
```

    ## [1] 172.69

``` r
min(bangaloreWaterTaxSF$diff, na.rm = T)
```

    ## [1] -71.93

## Build Map

``` r
# Base layer

waterTaxBaseLayerL <- tm_shape(bangaloreWardBoundary) +
  tm_borders(col = "#000000",
             lwd = 4) + 
  tm_layout(bg.color = "#ffffff",
            frame = F,
            attr.outside = T,
            outer.margins = 0,
            asp = 0,
            scale = 0.8,
            main.title = "Bangalore: Water Tax (Billed - Collected)",
            main.title.color = "#000000",
            main.title.size = 1.75,
            main.title.fontface = 2,
            main.title.fontfamily = "Arial Narrow") + 
  tm_credits("#30DayMapChallenge | Day 20 | Anirudh Govind | Nov 2020 |\nBSCL, 2019, Water Revenue Data 2013-14 to 2017-18, https://bit.ly/3kShtrE. Published under National Data Sharing and Accessibility Policy (NDSAP): https://bit.ly/3nLNB1S",
             col = "#000000",
             size = 1,
             position = c("left", "bottom"),
             fontfamily = "Arial Narrow") +
  tmap_options(max.categories = 12)

# Ward layer

waterTaxWards <- tm_shape(bangaloreWards) +
  tm_borders(col = "#000000",
             lwd = 1.5)

# Tax info layer

waterTaxL <- tm_shape(bangaloreWaterTaxSF) +
  tm_fill(col = "diff", 
          palette = "RdBu",
          midpoint = 0,
          breaks = c(-75,-50,-25,0,25,50,75,100,125,150,175),
          legend.show = F,
          legend.is.portrait = F) +
  tm_facets(by = "year",
            ncol = 2,
            nrow = 2) +
  tm_layout(panel.label.color = "#000000",
            panel.label.size = 1.2,
            panel.label.fontfamily = "Arial Narrow",
            panel.label.bg.color = "white",
            panel.label.fontface = 2,
            frame.lwd = NA)

bangaloreWaterTaxMapL <- waterTaxBaseLayerL + 
  waterTaxL + 
  waterTaxWards
```

## Export Map

``` r
# Export the map as an image to upload onto twitter

tmap_save(tm = bangaloreWaterTaxMapL,
          filename = here::here("exports/Day21.png"),
          dpi = 450,
          width = 200,
          height = 200,
          units = "mm")
```

    ## Warning: The shape bangaloreWaterTaxSF contains empty units.

    ## Map saved to G:\00_Git Repos\30DayMapChallenge\exports\Day21.png

    ## Resolution: 3543.307 by 3543.307 pixels

    ## Size: 7.874016 by 7.874016 inches (450 dpi)
