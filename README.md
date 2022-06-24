
<!-- README.md is generated from README.Rmd. Please edit that file -->

# gfwr

<!-- badges: start -->
<!-- badges: end -->
<!-- Add link to API documentation page once it's ready-->

The `gfwr` R package is a simple wrapper for the Global Fishing Watch
(GFW) [APIs](https://api-doc.dev.globalfishingwatch.org/#introduction).
It provides convenient functions to pull GFW data directly into R in
tidy formats.

The package currently works with the following APIs:

-   Vessel API: vessel search and identity based on AIS self reported
    data
-   Events API: encounters, loitering, port visits and fishing events
    based on AIS data
-   Map Visualization (4Wings API): apparent fishing effort based on AIS
    data

## Installation

You can install the development version of gfwr like so:

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
the [GFW API Portal](https://globalfishingwatch.org/ocean-engine/tokens/signup). Save
this token to your `.Renviron` file (using `usethis::edit_r_environ()`) by adding a string
named `"GFW_TOKEN"` to the file (`"GFW_TOKEN" = "PASTE_YOUR_TOKEN_HERE"`). Save 
the `.Renviron` file and restart the R session to make the edit effective.  

Then use the `get_auth` helper function to save the information in an object
in your R workspace every time you need to extract the token and pass it
to subsequent `gfwr` functions.

So you can do:

``` r
key <- get_auth()

# or this
key <- Sys.getenv("GFW_TOKEN")
```

## Vessels API

The `get_vessel_info` function allows you to get vessel identity
details. There are three search types: `basic`, `advanced`, and `id`.

-   `basic` search takes features like MMSI, IMO, callsign, shipname as
    inputs and identifies all vessels in the specified dataset that
    match
-   `advanced` search allows for the use of fuzzy matching with terms
    such as LIKE. The `id` search allows the user to search using a GFW
    vessel
-   `id` allows the user to specify the `vessel id` (generated by GFW)

> **Note**: <br> `vessel id` is an internal ID generated by GFW to
> connect data accross APIs and involves a combination of vessel and
> tracking data information

The user can also specify which identity databases to use:
`carrier_vessel`, `support_vessel`, `fishing_vessel`, or `all`. With the
latter, all databases are used for the search. This is generally
recommended and is the option set by default.

### Examples

To get information of a vessel with MMSI = 224224000 using all datasets:

``` r
get_vessel_info(query = 224224000, search_type = "basic", 
                dataset = "all", key = key)
#> # A tibble: 1 × 17
#>    name callsign firstTransmissionD… flag  geartype id    imo   lastTransmissio…
#>   <int> <chr>    <chr>               <chr> <lgl>    <chr> <chr> <chr>           
#> 1     1 EBSJ     2015-10-13T15:47:1… ESP   NA       3c99… 8733… 2019-10-15T12:1…
#> # … with 9 more variables: mmsi <chr>, msgCount <int>, posCount <int>,
#> #   shipname <chr>, source <chr>, vesselType <chr>, years <list>,
#> #   dataset <chr>, score <dbl>
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
> **Note**: 
<br>
> No spaces or newlines are permitted between the `vessel ids` 

``` r
get_vessel_info(query = 
                  "8c7304226-6c71-edbe-0b63-c246734b3c01,6583c51e3-3626-5638-866a-f47c3bc7ef7c,71e7da672-2451-17da-b239-857831602eca", 
                search_type = 'id', key = key)
```

## Events API

The `get_event` function allows you to get data from vessel activities
such as: apparent fishing events, encounters, loitering, and port
visits. Find more information in our [caveat
documentation](https://api-doc.dev.globalfishingwatch.org/#data-caveat).

<!-- I don't think we have tested loitering, or encounters yet-->
<!-- #' Base function to get event from API and convert response to data frame -->
<!-- #' -->
<!-- #' @param event_type Type of event to get data of. It can be "port_visit" or "fishing" -->
<!-- #' @param vessel VesselID. How to get this? -->
<!-- #' @param include_regions Whether to include regions? Ask engineering if this can always be false -->
<!-- #' @param start_date Start of date range to search events -->
<!-- #' @param end_date End of date range to search events -->
<!-- #' @param key Authorization token. Can be obtained with gfw_auth function -->

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
                      key = key
                      )
#> # A tibble: 49 × 11
#>    id    type  start               end                   lat    lon regions     
#>    <chr> <chr> <dttm>              <dttm>              <dbl>  <dbl> <list>      
#>  1 4478… port… 2015-10-15 10:40:07 2015-10-15 11:45:44  5.21  -4.01 <named list>
#>  2 b725… port… 2015-11-04 05:22:13 2015-11-07 10:46:28  5.23  -4.00 <named list>
#>  3 f03f… port… 2015-12-06 11:48:38 2015-12-10 16:19:37  5.24  -4.08 <named list>
#>  4 cbd7… port… 2016-01-09 06:47:57 2016-01-13 14:30:33  5.24  -4.00 <named list>
#>  5 6265… port… 2016-02-25 14:26:38 2016-03-01 13:21:21  5.25  -4.00 <named list>
#>  6 4a7f… port… 2016-03-03 05:47:02 2016-03-03 11:46:33  5.20  -4.02 <named list>
#>  7 617d… port… 2016-03-31 04:43:41 2016-04-02 09:07:10  5.23  -4.00 <named list>
#>  8 3c26… port… 2016-04-20 06:50:58 2016-04-20 19:47:10 14.7  -17.4  <named list>
#>  9 104e… port… 2016-04-24 07:14:33 2016-04-24 11:54:59 14.7  -17.4  <named list>
#> 10 8f19… port… 2016-05-18 19:31:04 2016-05-22 14:20:05  5.20  -4.01 <named list>
#> # … with 39 more rows, and 4 more variables: boundingBox <list>,
#> #   distances <list>, vessel <list>, port_visit <list>
```

We can also use more than one `vessel id`:

``` r
get_event(event_type='port_visit',
                      vessel = '8c7304226-6c71-edbe-0b63-c246734b3c01,6583c51e3-3626-5638-866a-f47c3bc7ef7c',
                      include_regions = NULL,
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

## Map Visualization API

The `get_raster` function gets a raster from the 4Wings API and converts
the response to a data frame. In order to use it, you should specify:

-   The spatial resolution, which can be `low` (0.1 degree) or `high`
    (0.01 degree)
-   The temporal resolution, which can be `daily`, `monthly`, or
    `yearly`.
-   The variable to group by: `vessel_id`, `flag`, `geartype`, or
    `flagAndGearType`
-   The date range
-   The output format (currently supporting csv)
-   The `geojson` shape or region code (such as an EEZ code) to filter the raster

### Examples

Here’s an example where we enter the geojson data manually:

``` r
shape_json = '{"geojson":{"type":"Polygon","coordinates":[[[-76.11328125,-26.273714024406416],[-76.201171875,-26.980828590472093],[-76.376953125,-27.527758206861883],[-76.81640625,-28.30438068296276],[-77.255859375,-28.767659105691244],[-77.87109375,-29.152161283318918],[-78.486328125,-29.45873118535532],[-79.189453125,-29.61167011519739],[-79.892578125,-29.6880527498568],[-80.595703125,-29.61167011519739],[-81.5625,-29.382175075145277],[-82.177734375,-29.07537517955835],[-82.705078125,-28.6905876542507],[-83.232421875,-28.071980301779845],[-83.49609375,-27.683528083787756],[-83.759765625,-26.980828590472093],[-83.84765625,-26.35249785815401],[-83.759765625,-25.64152637306576],[-83.583984375,-25.16517336866393],[-83.232421875,-24.447149589730827],[-82.705078125,-23.966175871265037],[-82.177734375,-23.483400654325635],[-81.5625,-23.241346102386117],[-80.859375,-22.998851594142906],[-80.15625,-22.917922936146027],[-79.453125,-22.998851594142906],[-78.662109375,-23.1605633090483],[-78.134765625,-23.40276490540795],[-77.431640625,-23.885837699861995],[-76.9921875,-24.28702686537642],[-76.552734375,-24.846565348219727],[-76.2890625,-25.48295117535531],[-76.11328125,-26.273714024406416]]]}}'

get_raster(spatial_resolution = 'low',
           temporal_resolution = 'yearly',
           group_by = 'flag',
           date_range = '2020-01-01,2021-10-01',
           shape = shape_json,
           key = key)
```

If you want raster data from a particular EEZ, you can use the
`get_eez_code` function to get the EEZ code and enter that code in the
`shape` argument of `get_raster` instead of the geojson data:

``` r
# use EEZ function to get EEZ code of Cote d'Ivoire
code_eez <- get_eez_code(eez_name = 'CIV', key = key)

get_raster(spatial_resolution = 'low',
           temporal_resolution = 'yearly',
           group_by = 'flag',
           date_range = '2021-01-01,2021-10-01',
           shape = code_eez$id,
           key = key)
```

You could search for just one word in the name of the EEZ and then
decide which one you want:

``` r
(get_eez_code(eez_name = 'French', key = key))
#> # A tibble: 3 × 3
#>      id iso3  label                                    
#>   <int> <chr> <chr>                                    
#> 1  5677 FRA   French Exclusive Economic Zone           
#> 2  8462 FRA   French Guiana Exclusive Economic Zone    
#> 3  8440 FRA   French Polynesian Exclusive Economic Zone

# Let's say we're interested in the French Exclusive Economic Zone, 5677
get_raster(spatial_resolution = 'low',
           temporal_resolution = 'yearly',
           group_by = 'flag',
           date_range = '2021-01-01,2021-10-01',
           shape = 5677,
           key = key)
#> Rows: 5500 Columns: 5
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr (1): flag
#> dbl (4): Lat, Lon, Time Range, Apparent Fishing hours
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#> # A tibble: 5,500 × 5
#>      Lat   Lon `Time Range` flag  `Apparent Fishing hours`
#>    <dbl> <dbl>        <dbl> <chr>                    <dbl>
#>  1  47.3  -5.8         2021 FRA                     1446. 
#>  2  49.5  -3.7         2021 GBR                      307. 
#>  3  48.2  -8.3         2021 FRA                      415. 
#>  4  48.8  -5.3         2021 FRA                      218. 
#>  5  48.8  -5.1         2021 FRA                      431. 
#>  6  47.4  -6.5         2021 FRA                       62.6
#>  7  47.9  -5.7         2021 ESP                       26.4
#>  8  50    -0.9         2021 FRA                       39.0
#>  9  50.2  -0.3         2021 FRA                       64.1
#> 10  50.2  -0.1         2021 FRA                       95.3
#> # … with 5,490 more rows
```
