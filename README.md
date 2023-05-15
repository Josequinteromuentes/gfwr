
<!-- README.md is generated from README.Rmd. Please edit that file -->

# `gfwr`: Access data from Global Fishing Watch APIs <img src="man/figures/gfwr_hex_rgb.png" align="right" width="200px"/>

<!-- badges: start -->

[![DOI](https://zenodo.org/badge/450635054.svg)](https://zenodo.org/badge/latestdoi/450635054)
[![Project Status: Active - The project has reached a stable, usable
state and is being actively
developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)
[![Licence](https://img.shields.io/badge/license-Apache%202-blue)](https://opensource.org/licenses/Apache-2.0)
<!-- badges: end -->

The `gfwr` R package is a simple wrapper for the Global Fishing Watch
(GFW)
[APIs](https://globalfishingwatch.org/our-apis/documentation#introduction).
It provides convenient functions to freely pull GFW data directly into R
in tidy formats.

The package currently works with the following APIs:

-   [Vessels
    API](https://globalfishingwatch.org/our-apis/documentation#vessels-api):
    vessel search and identity based on AIS self reported data
-   [Events
    API](https://globalfishingwatch.org/our-apis/documentation#events-api):
    encounters, loitering, port visits and fishing events based on AIS
    data
-   [Map Visualization (4Wings
    API)](https://globalfishingwatch.org/our-apis/documentation#map-visualization-4wings-api):
    apparent fishing effort based on AIS data

> **Note**:  
> See the [Terms of
> Use](https://globalfishingwatch.org/our-apis/documentation#reference-data)
> page for GFW APIs for information on our API licenses and rate limits.

## Installation

You can install the development version of `gfwr` like so:

``` r
devtools::install_github("GlobalFishingWatch/gfwr")
```

Once everything is installed, you can load and use `gfwr` in your
scripts with `library(gfwr)`

``` r
library(gfwr)
```

## Authorization

The use of `gfwr` requires a GFW API token, which users can request from
the [GFW API Portal](https://globalfishingwatch.org/our-apis/tokens).
Save this token to your `.Renviron` file (using
`usethis::edit_r_environ()`) by adding a variable named `GFW_TOKEN` to
the file (`GFW_TOKEN = "PASTE_YOUR_TOKEN_HERE"`). Save the `.Renviron`
file and restart the R session to make the edit effective.

Then use the `gfw_auth` helper function to save the information to an
object in your R workspace every time you need to extract the token and
pass it to subsequent `gfwr` functions.

So you can do:

``` r
key <- gfw_auth()
```

or this

``` r
key <- Sys.getenv("GFW_TOKEN")
```

> **Note**:  
> `gfwr` functions are set to use `key = gfw_auth()` by default.

## Vessels API

The `get_vessel_info` function allows you to get vessel identity details
from the [GFW Vessels
API](https://globalfishingwatch.org/our-apis/documentation#introduction-vessels-api).
There are three search types: `basic`, `advanced`, and `id`.

-   `basic` search takes features like MMSI, IMO, callsign, shipname as
    inputs and identifies all vessels in the specified dataset that
    match
-   `advanced` search allows for the use of fuzzy matching with terms
    such as LIKE. The `id` search allows the user to search using a GFW
    vessel
-   `id` allows the user to specify the `vessel id` (generated by GFW)

> **Note**:  
> `vessel id` is an internal ID generated by GFW to connect data accross
> APIs and involves a combination of vessel and tracking data
> information

The user can also specify which identity databases to use:
`carrier_vessel`, `support_vessel`, `fishing_vessel`, or `all`. With the
latter, all databases are used for the search. This is generally
recommended and is the option set by default.

### Examples

To get information of a vessel with MMSI = 224224000 using all datasets:

``` r
get_vessel_info(query = 224224000, 
                search_type = "basic", 
                dataset = "all", 
                key = key)
#> # A tibble: 1 × 17
#>    name callsign first…¹ flag  geart…² id    imo   lastT…³ mmsi  msgCo…⁴ posCo…⁵
#>   <int> <chr>    <chr>   <chr> <lgl>   <chr> <chr> <chr>   <chr>   <int>   <int>
#> 1     1 EBSJ     2015-1… ESP   NA      3c99… 8733… 2019-1… 2242… 1887249   73677
#> # … with 6 more variables: shipname <chr>, source <chr>, vesselType <chr>,
#> #   years <list>, dataset <chr>, score <dbl>, and abbreviated variable names
#> #   ¹​firstTransmissionDate, ²​geartype, ³​lastTransmissionDate, ⁴​msgCount,
#> #   ⁵​posCount
```

To combine different fields and do fuzzy matching to search the
`carrier vessel` dataset:

``` r
get_vessel_info(query = "shipname LIKE '%GABU REEFE%' OR imo = '8300949'", 
                search_type = "advanced", dataset = "carrier_vessel", key = key)
```

To specify a `vessel id`:

``` r
get_vessel_info(query = "8c7304226-6c71-edbe-0b63-c246734b3c01", 
                search_type = "id", dataset = "carrier_vessel", key = key)
```

To specify more than one `vessel id`:

> **Note**: <br> No spaces or newlines are permitted between the
> `vessel ids`

``` r
get_vessel_info(query = 
                  "8c7304226-6c71-edbe-0b63-c246734b3c01,6583c51e3-3626-5638-866a-f47c3bc7ef7c,71e7da672-2451-17da-b239-857831602eca", 
                search_type = 'id', key = key)
```

## Events API

The `get_event` function allows you to get data on specific vessel
activities from the [GFW Events
API](https://globalfishingwatch.org/our-apis/documentation#events-api).
Event types include: apparent fishing events, potential transshipment
events (two-vessel encounters and loitering by refrigerated carrier
vessels), and port visits. Find more information in our [caveat
documentation](https://globalfishingwatch.org/our-apis/documentation#data-caveat).

### Examples

Let’s say that you don’t know the `vessel id` but you have the MMSI (or
other identity information). You can use `get_vessel_info` function
first to extract `vessel id` and then use it in the `get_event`
function:

``` r
vessel_id <- get_vessel_info(query = 224224000, search_type = "basic", key = key)$id
```

To get a list of port visits for that vessel:

``` r
get_event(event_type='port_visit',
          vessel = vessel_id,
          confidences = '4',
          key = key
          )
#> [1] "Downloading 35 events from GFW"
#> # A tibble: 35 × 11
#>    id    type  start               end                   lat    lon regions     
#>    <chr> <chr> <dttm>              <dttm>              <dbl>  <dbl> <list>      
#>  1 b725… port… 2015-11-04 05:22:13 2015-11-07 10:46:28  5.23  -4.00 <named list>
#>  2 f03f… port… 2015-12-06 11:48:38 2015-12-10 16:19:37  5.24  -4.08 <named list>
#>  3 cbd7… port… 2016-01-09 06:47:57 2016-01-13 14:30:33  5.24  -4.00 <named list>
#>  4 6265… port… 2016-02-25 14:26:38 2016-03-01 13:21:21  5.25  -4.00 <named list>
#>  5 4a7f… port… 2016-03-03 05:47:02 2016-03-03 11:46:33  5.20  -4.02 <named list>
#>  6 617d… port… 2016-03-31 04:43:41 2016-04-02 09:07:10  5.23  -4.00 <named list>
#>  7 3c26… port… 2016-04-20 06:50:58 2016-04-20 19:47:10 14.7  -17.4  <named list>
#>  8 104e… port… 2016-04-24 07:14:33 2016-04-24 11:54:59 14.7  -17.4  <named list>
#>  9 8f19… port… 2016-05-18 19:31:04 2016-05-22 14:20:05  5.20  -4.01 <named list>
#> 10 bf64… port… 2016-06-26 15:08:16 2016-06-30 10:39:03  5.20  -4.07 <named list>
#> # … with 25 more rows, and 4 more variables: boundingBox <list>,
#> #   distances <list>, vessel <list>, event_info <list>
```

We can also use more than one `vessel id`:

``` r
get_event(event_type='port_visit',
          vessel = '8c7304226-6c71-edbe-0b63-c246734b3c01,6583c51e3-3626-5638-866a-f47c3bc7ef7c',
          confidences = 4,
          start_date = "2020-01-01",
          end_date = "2020-02-01",
          key = key
          )
```

Or get encounters for all vessels in a given date range:

``` r
get_event(event_type='encounter',
          start_date = "2020-01-01",
          end_date = "2020-02-01",
          key = key
          )
```

As another example, let’s combine the Vessels and Events APIs to get
fishing events for a list of 10 USA-flagged trawlers:

``` r
# Download the list of USA trawlers
usa_trawlers <- get_vessel_info(
  query = "flag = 'USA' AND geartype = 'trawlers'", 
  search_type = "advanced", 
  dataset = "fishing_vessel",
  key = key
  )

# Collapse vessel ids into a commas separated list to pass to Events API
usa_trawler_ids <- paste0(usa_trawlers$id[1:10], collapse = ',')
```

Now get the list of fishing events for these trawlers in January, 2020:

``` r
get_event(event_type='fishing',
          vessel = usa_trawler_ids,
          start_date = "2020-01-01",
          end_date = "2020-02-01",
          key = key
          )
#> [1] "Downloading 2 events from GFW"
#> # A tibble: 2 × 11
#>   id      type  start               end                   lat   lon regions     
#>   <chr>   <chr> <dttm>              <dttm>              <dbl> <dbl> <list>      
#> 1 777fb1… fish… 2020-01-09 23:15:22 2020-01-09 23:52:24  28.1 -94.0 <named list>
#> 2 f19d61… fish… 2020-01-10 12:59:23 2020-01-10 16:55:15  28.1 -93.9 <named list>
#> # … with 4 more variables: boundingBox <list>, distances <list>, vessel <list>,
#> #   event_info <list>
```

When no events are available, the `get_event()` function returns
nothing.

``` r
get_event(event_type='fishing',
          vessel = usa_trawler_ids,
          start_date = "2020-01-01",
          end_date = "2020-01-01",
          key = key
          )
#> [1] "Your request returned zero results"
#> NULL
```

## Map Visualization API

The `get_raster` function gets a raster from the [4Wings
API](https://globalfishingwatch.org/our-apis/documentation#map-visualization-4wings-api)
and converts the response to a data frame. In order to use it, you
should specify:

-   The spatial resolution, which can be `low` (0.1 degree) or `high`
    (0.01 degree)
-   The temporal resolution, which can be `daily`, `monthly`, or
    `yearly`.
-   The variable to group by: `vessel_id`, `flag`, `gearType`, or
    `flagAndGearType`
-   The date range `note: this must be one (1) year or less`
-   The `geojson` region or region code (such as an EEZ code) to filter
    the raster
-   The source for the specified region (currently, `eez`, `mpa`, or
    `user_json`)

### Examples

Here’s an example where we enter the geojson data manually:

``` r

region_json = '{"geojson":{"type":"Polygon","coordinates":[[[-76.11328125,-26.273714024406416],[-76.201171875,-26.980828590472093],[-76.376953125,-27.527758206861883],[-76.81640625,-28.30438068296276],[-77.255859375,-28.767659105691244],[-77.87109375,-29.152161283318918],[-78.486328125,-29.45873118535532],[-79.189453125,-29.61167011519739],[-79.892578125,-29.6880527498568],[-80.595703125,-29.61167011519739],[-81.5625,-29.382175075145277],[-82.177734375,-29.07537517955835],[-82.705078125,-28.6905876542507],[-83.232421875,-28.071980301779845],[-83.49609375,-27.683528083787756],[-83.759765625,-26.980828590472093],[-83.84765625,-26.35249785815401],[-83.759765625,-25.64152637306576],[-83.583984375,-25.16517336866393],[-83.232421875,-24.447149589730827],[-82.705078125,-23.966175871265037],[-82.177734375,-23.483400654325635],[-81.5625,-23.241346102386117],[-80.859375,-22.998851594142906],[-80.15625,-22.917922936146027],[-79.453125,-22.998851594142906],[-78.662109375,-23.1605633090483],[-78.134765625,-23.40276490540795],[-77.431640625,-23.885837699861995],[-76.9921875,-24.28702686537642],[-76.552734375,-24.846565348219727],[-76.2890625,-25.48295117535531],[-76.11328125,-26.273714024406416]]]}}'

get_raster(
  spatial_resolution = 'low',
  temporal_resolution = 'yearly',
  group_by = 'flag',
  date_range = '2021-01-01,2021-12-31',
  region = region_json,
  region_source = 'user_json',
  key = key
  )
```

If you want raster data from a particular EEZ, you can use the
`get_region_id` function to get the EEZ id, enter that code in the
`region` argument of `get_raster` instead of the geojson data (ensuring
you specify the `region_source` as `'eez'`:

``` r
# use EEZ function to get EEZ code of Cote d'Ivoire
code_eez <- get_region_id(region_name = 'CIV', region_source = 'eez', key = key)

get_raster(spatial_resolution = 'low',
           temporal_resolution = 'yearly',
           group_by = 'flag',
           date_range = '2021-01-01,2021-10-01',
           region = code_eez$id,
           region_source = 'eez',
           key = key)
```

You could search for just one word in the name of the EEZ and then
decide which one you want:

``` r
(get_region_id(region_name = 'France', region_source = 'eez', key = key))
#> # A tibble: 3 × 3
#>      id iso3  label                           
#>   <dbl> <chr> <chr>                           
#> 1  5677 FRA   France                          
#> 2 48976 FRA   Joint regime area Italy / France
#> 3 48966 FRA   Joint regime area Spain / France

# Let's say we're interested in the French Exclusive Economic Zone, 5677
get_raster(spatial_resolution = 'low',
           temporal_resolution = 'yearly',
           group_by = 'flag',
           date_range = '2021-01-01,2021-10-01',
           region = 5677,
           region_source = 'eez',
           key = key)
#> Rows: 5444 Columns: 6
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr (1): flag
#> dbl (5): Lat, Lon, Time Range, Vessel IDs, Apparent Fishing hours
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#> # A tibble: 5,444 × 6
#>      Lat   Lon `Time Range` flag  `Vessel IDs` `Apparent Fishing hours`
#>    <dbl> <dbl>        <dbl> <chr>        <dbl>                    <dbl>
#>  1  48.9  -5.8         2021 FRA             15                    209. 
#>  2  49    -5.6         2021 FRA             18                    280. 
#>  3  42.7   3.6         2021 FRA             11                    382. 
#>  4  42.7   3.7         2021 FRA              7                    102. 
#>  5  43.5   4.1         2021 FRA             23                    557. 
#>  6  43.3   4.5         2021 FRA             19                    413. 
#>  7  43     4.8         2021 FRA             11                     63.9
#>  8  43.3   4.2         2021 FRA             32                    952. 
#>  9  43.4   4           2021 FRA             36                   2099. 
#> 10  43.1   4.5         2021 FRA             20                    154. 
#> # … with 5,434 more rows
```

A similar approach can be used to search for a specific Marine Protected
Area, in this case the Phoenix Island Protected Area (PIPA)

``` r
# use region id function to get MPA code of Phoenix Island Protected Area
code_mpa <- get_region_id(region_name = 'Phoenix', region_source = 'mpa', key = key)

get_raster(spatial_resolution = 'low',
           temporal_resolution = 'yearly',
           group_by = 'flag',
           date_range = '2015-01-01,2015-06-01',
           region = code_mpa$id[1],
           region_source = 'mpa',
           key = key)
```

It is also possible to filter rasters to one of the five regional
fisheries management organizations (RFMO) that manage tuna and tuna-like
species. These include `"ICCAT"`, `"IATTC"`,`"IOTC"`, `"CCSBT"` and
`"WCPFC"`.

``` r
get_raster(spatial_resolution = 'low',
           temporal_resolution = 'daily',
           group_by = 'flag',
           date_range = '2021-01-01,2021-01-15',
           region = 'ICCAT',
           region_source = 'rfmo',
           key = key)
#> Rows: 112699 Columns: 6
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr  (1): flag
#> dbl  (4): Lat, Lon, Vessel IDs, Apparent Fishing hours
#> date (1): Time Range
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#> # A tibble: 112,699 × 6
#>      Lat   Lon `Time Range` flag  `Vessel IDs` `Apparent Fishing hours`
#>    <dbl> <dbl> <date>       <chr>        <dbl>                    <dbl>
#>  1  68.7 -51.4 2021-01-08   GRL              1                    0.250
#>  2  68.8 -51.2 2021-01-05   GRL              1                    4.78 
#>  3  68.8 -51.2 2021-01-04   GRL              2                    0.728
#>  4  66.9 -24.7 2021-01-03   ISL             11                   14.7  
#>  5  66.9 -24.6 2021-01-03   ISL             12                   34.5  
#>  6  66.9 -24.5 2021-01-03   ISL             13                   40.2  
#>  7  66.8 -24.3 2021-01-04   ISL             15                   42.2  
#>  8  67   -24   2021-01-04   ISL              3                    6.13 
#>  9  66.9 -23.9 2021-01-04   ISL              2                    1.43 
#> 10  66.9 -24.1 2021-01-03   ISL              3                    3.40 
#> # … with 112,689 more rows
```

The `get_region_id` function also works in reverse. If a region id is
passed as a `numeric` to the function as the `region_name`, the
corresponding region label or iso3 can be returned. This is especially
useful when events are returned with regions.

``` r
# using same example as above
get_event(event_type = 'fishing',
          vessel = usa_trawler_ids,
          start_date = "2020-01-01",
          end_date = "2020-02-01",
          include_regions = TRUE,
          key = key
          ) %>%
  # extract EEZ id code
  dplyr::mutate(eez = as.character(purrr::map(purrr::map(regions, pluck, 'eez'), 
                                              paste0, collapse = ','))) %>%
  dplyr::select(id, type, start, end, lat, lon, eez) %>%
  dplyr::rowwise() %>%
  dplyr::mutate(eez_name = get_region_id(region_name = as.numeric(eez),
                                         region_source = 'eez',
                                         key = key)$label)
#> [1] "Downloading 2 events from GFW"
#> # A tibble: 2 × 8
#> # Rowwise: 
#>   id     type  start               end                   lat   lon eez   eez_n…¹
#>   <chr>  <chr> <dttm>              <dttm>              <dbl> <dbl> <chr> <chr>  
#> 1 777fb… fish… 2020-01-09 23:15:22 2020-01-09 23:52:24  28.1 -94.0 8456  United…
#> 2 f19d6… fish… 2020-01-10 12:59:23 2020-01-10 16:55:15  28.1 -93.9 8456  United…
#> # … with abbreviated variable name ¹​eez_name
```

## Contributing

We welcome all contributions to improve the package! Please read our
[Contribution
Guide](https://github.com/GlobalFishingWatch/gfwr/blob/main/Contributing.md)
and reach out!
