
<!-- README.md is generated from README.Rmd. Please edit that file -->

# `gfwr`: Access data from Global Fishing Watch APIs <img src="man/figures/gfwr_hex_rgb.png" align="right" width="200px"/>

<!-- badges: start -->

[![DOI](https://zenodo.org/badge/450635054.svg)](https://zenodo.org/badge/latestdoi/450635054)
[![Project Status: Active - The project has reached a stable, usable
state and is being actively
developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)
[![Licence](https://img.shields.io/badge/license-Apache%202-blue)](https://opensource.org/licenses/Apache-2.0)
<!-- badges: end -->

> **Important**  
> The current version of `gfwr` gives access to Global Fishing Watch API
> [version
> 2](https://globalfishingwatch.org/our-apis/documentation#version-2-api).
> This version is in Maintenance mode, it will be operational and
> available but no new functionalities will be added. This version will
> be Deprecated on April 30 2024. A new version fetching data from
> [version
> 3](https://globalfishingwatch.org/our-apis/documentation#version-3-api)
> is being prepared.

The `gfwr` R package is a simple wrapper for the Global Fishing Watch
(GFW)
[APIs](https://globalfishingwatch.org/our-apis/documentation#introduction).
It provides convenient functions to freely pull GFW data directly into R
in tidy formats.

The package currently works with the following APIs:

- [Vessels
  API](https://globalfishingwatch.org/our-apis/documentation#vessels-api):
  vessel search and identity based on AIS self reported data
- [Events
  API](https://globalfishingwatch.org/our-apis/documentation#events-api):
  encounters, loitering, port visits and fishing events based on AIS
  data
- [Map Visualization (4Wings
  API)](https://globalfishingwatch.org/our-apis/documentation#map-visualization-4wings-api):
  apparent fishing effort based on AIS data

> **Note**:  
> See the [Terms of
> Use](https://globalfishingwatch.org/our-apis/documentation#reference-data)
> page for GFW APIs for information on our API licenses and rate limits.

## Installation

You can install the development version of `gfwr` like so:

``` r
# Check/install remotes
if (!require("remotes"))
  install.packages("remotes")

remotes::install_github("GlobalFishingWatch/gfwr")
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

Then use the `gfw_auth()` helper function to save the information to an
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

The `get_vessel_info()` function allows you to get vessel identity
details from the [GFW Vessels
API](https://globalfishingwatch.org/our-apis/documentation#introduction-vessels-api).
There are three search types: `basic`, `advanced`, and `id`.

- `basic` search takes features like MMSI, IMO, callsign, shipname as
  inputs and identifies all vessels in the specified dataset that match
- `advanced` search allows for the use of fuzzy matching with terms such
  as LIKE. The `id` search allows the user to search using a GFW vessel
- `id` allows the user to specify the `vessel id` (generated by GFW)

> **Note**:  
> `vessel id` is an internal ID generated by GFW to connect data accross
> APIs and involves a combination of vessel and tracking data
> information

The user can also specify which identity databases to use:
`carrier_vessel`, `support_vessel`, `fishing_vessel`, or `all`. With the
latter, all databases are used for the search. This is generally
recommended and is the option set by default.

### Examples

To get information of a vessel with `MMSI = 224224000` using all
datasets:

``` r
get_vessel_info(query = 224224000, 
                search_type = "basic", 
                dataset = "all", 
                key = key)
#> # A tibble: 1 × 17
#>    name callsign firstTransmissionDate flag  geartype id                   imo  
#>   <int> <chr>    <chr>                 <chr> <lgl>    <chr>                <chr>
#> 1     1 EBSJ     2015-10-13T15:47:16Z  ESP   NA       3c99c326d-dd2e-175d… 8733…
#> # ℹ 10 more variables: lastTransmissionDate <chr>, mmsi <chr>, msgCount <int>,
#> #   posCount <int>, shipname <chr>, source <chr>, vesselType <chr>,
#> #   years <list>, dataset <chr>, score <dbl>
```

To combine different fields and do fuzzy matching to search the
`carrier vessel` dataset:

``` r
get_vessel_info(query = "shipname LIKE '%GABU REEFE%' OR imo = '8300949'", 
                search_type = "advanced", dataset = "carrier_vessel", key = key)
#> # A tibble: 3 × 17
#>    name callsign firstTransmissionDate flag  geartype id                   imo  
#>   <int> <chr>    <chr>                 <chr> <lgl>    <chr>                <chr>
#> 1     1 ER2732   2019-02-22T21:46:13Z  MDA   NA       0b7047cb5-58c8-6e63… 8300…
#> 2     2 TJMC996  2022-01-24T09:13:48Z  CMR   NA       1da8dbc23-3c48-d5ce… 8300…
#> 3     3 D6FJ2    2012-01-02T16:50:42Z  COM   NA       58cf536b1-1fca-dac3… 8300…
#> # ℹ 10 more variables: lastTransmissionDate <chr>, mmsi <chr>, msgCount <int>,
#> #   posCount <int>, shipname <chr>, source <chr>, vesselType <chr>,
#> #   years <list>, dataset <chr>, score <dbl>
```

To specify a `vessel id`:

``` r
get_vessel_info(query = "8c7304226-6c71-edbe-0b63-c246734b3c01", 
                search_type = "id", 
                dataset = "carrier_vessel",
                key = key)
#> # A tibble: 1 × 16
#>    name callsign firstTransmissionDate flag  geartype id                   imo  
#>   <int> <chr>    <chr>                 <chr> <lgl>    <chr>                <chr>
#> 1     1 5BWC3    2013-05-15T20:18:31Z  CYP   NA       8c7304226-6c71-edbe… 9076…
#> # ℹ 9 more variables: lastTransmissionDate <chr>, mmsi <chr>, msgCount <int>,
#> #   posCount <int>, shipname <chr>, source <chr>, vesselType <chr>,
#> #   years <list>, dataset <chr>
```

To specify more than one `vessel id`:

> **Note**:  
> No spaces or newlines are permitted between the `vessel ids`

``` r
get_vessel_info(query = 
                  "8c7304226-6c71-edbe-0b63-c246734b3c01,6583c51e3-3626-5638-866a-f47c3bc7ef7c,71e7da672-2451-17da-b239-857831602eca", 
                search_type = "id", key = key)
#> # A tibble: 3 × 16
#>    name callsign firstTransmissionDate flag  geartype          id          imo  
#>   <int> <chr>    <chr>                 <chr> <chr>             <chr>       <chr>
#> 1     1 5BWC3    2013-05-15T20:18:31Z  CYP   <NA>              8c7304226-… 9076…
#> 2     2 DTBY3    2013-09-02T03:59:51Z  KOR   tuna_purse_seines 6583c51e3-… 8919…
#> 3     3 DUQA-7   2017-02-15T05:54:53Z  PHL   tuna_purse_seines 71e7da672-… 8118…
#> # ℹ 9 more variables: lastTransmissionDate <chr>, mmsi <chr>, msgCount <int>,
#> #   posCount <int>, shipname <chr>, source <chr>, vesselType <chr>,
#> #   years <list>, dataset <chr>
```

## Events API

The `get_event()` function allows you to get data on specific vessel
activities from the [GFW Events
API](https://globalfishingwatch.org/our-apis/documentation#events-api).
Event types include: apparent fishing events, potential transshipment
events (two-vessel encounters and loitering by refrigerated carrier
vessels), and port visits. Find more information in our [caveat
documentation](https://globalfishingwatch.org/our-apis/documentation#data-caveat).

### Examples

Let’s say that you don’t know the `vessel id` but you have the MMSI (or
other identity information). You can use `get_vessel_info()` function
first to extract `vessel id` and then use it in the `get_event()`
function:

``` r
vessel_id <- get_vessel_info(query = 224224000, search_type = "basic", key = key)$id
```

To get a list of port visits for that vessel:

``` r
get_event(event_type = "port_visit",
          vessel = vessel_id,
          confidences = "4",
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
#> # ℹ 25 more rows
#> # ℹ 4 more variables: boundingBox <list>, distances <list>, vessel <list>,
#> #   event_info <list>
```

We can also use more than one `vessel id`:

``` r
get_event(event_type = "port_visit",
          vessel = "8c7304226-6c71-edbe-0b63-c246734b3c01,6583c51e3-3626-5638-866a-f47c3bc7ef7c",
          confidences = 4,
          start_date = "2020-01-01",
          end_date = "2020-02-01",
          key = key
          )
#> [1] "Downloading 3 events from GFW"
#> # A tibble: 3 × 11
#>   id      type  start               end                   lat   lon regions     
#>   <chr>   <chr> <dttm>              <dttm>              <dbl> <dbl> <list>      
#> 1 7cd1e3… port… 2019-12-19 23:05:31 2020-01-24 19:05:18  28.1 -15.4 <named list>
#> 2 c2f096… port… 2020-01-26 05:52:47 2020-01-29 14:39:33  20.8 -17.0 <named list>
#> 3 7c06e4… port… 2020-01-31 02:20:08 2020-02-03 15:56:31  28.1 -15.4 <named list>
#> # ℹ 4 more variables: boundingBox <list>, distances <list>, vessel <list>,
#> #   event_info <list>
```

Or get encounters for all vessels in a given date range:

``` r
get_event(event_type = "encounter",
          start_date = "2020-01-01",
          end_date = "2020-01-03",
          key = key
          )
#> [1] "Downloading 70 events from GFW"
#> # A tibble: 70 × 11
#>    id                type  start               end                    lat    lon
#>    <chr>             <chr> <dttm>              <dttm>               <dbl>  <dbl>
#>  1 a3cff76a070a919f… enco… 2019-12-31 08:40:00 2020-01-01 07:40:00  57.5   157. 
#>  2 a3cff76a070a919f… enco… 2019-12-31 08:40:00 2020-01-01 07:40:00  57.5   157. 
#>  3 b059d20534c7fd5f… enco… 2019-12-31 12:00:00 2020-01-01 13:50:00 -17.6   -79.3
#>  4 b059d20534c7fd5f… enco… 2019-12-31 12:00:00 2020-01-01 13:50:00 -17.6   -79.3
#>  5 cd07d7e5d65e81b3… enco… 2019-12-31 12:50:00 2020-01-01 09:50:00 -17.7   -79.2
#>  6 cd07d7e5d65e81b3… enco… 2019-12-31 12:50:00 2020-01-01 09:50:00 -17.7   -79.2
#>  7 13dac0526c993292… enco… 2019-12-31 14:50:00 2020-01-01 20:20:00 -17.6   -79.4
#>  8 13dac0526c993292… enco… 2019-12-31 14:50:00 2020-01-01 20:20:00 -17.6   -79.4
#>  9 2e8b8040d87ad0ae… enco… 2019-12-31 16:00:00 2020-01-01 08:50:00  -3.44 -147. 
#> 10 2e8b8040d87ad0ae… enco… 2019-12-31 16:00:00 2020-01-01 08:50:00  -3.44 -147. 
#> # ℹ 60 more rows
#> # ℹ 5 more variables: regions <list>, boundingBox <list>, distances <list>,
#> #   vessel <list>, event_info <list>
```

When a date range is provided to `get_event()` using both `start_date`
and `end_date`, any event overlapping that range will be returned,
including events that start prior to `start_date` or end after
`end_date`. If just `start_date` or `end_date` are provided, results
will include all events that end after `start_date` or begin prior to
`end_date`, respectively.

> **Note**:  
> Because encounter events are events between two vessels, a single
> event will be represented twice in the data, once for each vessel. To
> capture this information and link the related data rows, the `id`
> field for encounter events includes an additional suffix (1 or 2)
> separated by a period. The `vessel` field will also contain different
> information specific to each vessel.

As another example, let’s combine the Vessels and Events APIs to get
fishing events for a list of 100 USA-flagged trawlers:

``` r
# Download the list of USA trawlers
usa_trawlers <- get_vessel_info(
  query = "flag = 'USA' AND geartype = 'trawlers'", 
  search_type = "advanced", 
  dataset = "fishing_vessel",
  key = key
  )

# Collapse vessel ids into a commas separated list to pass to Events API
usa_trawler_ids <- paste0(usa_trawlers$id[1:100], collapse = ",")
```

Now get the list of fishing events for these trawlers in January, 2020:

``` r
get_event(event_type = "fishing",
          vessel = usa_trawler_ids,
          start_date = "2020-01-01",
          end_date = "2020-02-01",
          key = key
          )
#> [1] "Downloading 106 events from GFW"
#> # A tibble: 106 × 11
#>    id    type  start               end                   lat    lon regions     
#>    <chr> <chr> <dttm>              <dttm>              <dbl>  <dbl> <list>      
#>  1 0678… fish… 2020-01-01 15:56:25 2020-01-02 00:41:57  35.1  -76.0 <named list>
#>  2 4891… fish… 2020-01-02 01:55:51 2020-01-03 00:05:57  35.0  -76.0 <named list>
#>  3 d75a… fish… 2020-01-02 23:31:48 2020-01-03 04:37:19  41.1  -71.4 <named list>
#>  4 8dda… fish… 2020-01-03 00:39:08 2020-01-03 02:49:08  35.0  -76.0 <named list>
#>  5 c85b… fish… 2020-01-03 15:51:15 2020-01-03 18:24:44  39.8  -73.9 <named list>
#>  6 1bee… fish… 2020-01-05 00:35:43 2020-01-05 06:11:43  39.7  -73.9 <named list>
#>  7 379d… fish… 2020-01-05 04:58:45 2020-01-05 06:31:45  43.7 -124.  <named list>
#>  8 0b45… fish… 2020-01-06 06:20:19 2020-01-08 02:46:19  39.6  -73.9 <named list>
#>  9 04d2… fish… 2020-01-06 21:12:01 2020-01-07 02:35:11  34.6  -76.6 <named list>
#> 10 2ad0… fish… 2020-01-07 13:37:54 2020-01-07 15:16:54  34.7  -76.8 <named list>
#> # ℹ 96 more rows
#> # ℹ 4 more variables: boundingBox <list>, distances <list>, vessel <list>,
#> #   event_info <list>
```

When no events are available, the `get_event()` function returns
nothing.

``` r
get_event(event_type = "fishing",
          vessel = usa_trawler_ids[2],
          start_date = "2020-01-01",
          end_date = "2020-01-01",
          key = key
          )
#> [1] "Your request returned zero results"
#> NULL
```

## Map Visualization API

The `get_raster()` function gets a raster from the [4Wings
API](https://globalfishingwatch.org/our-apis/documentation#map-visualization-4wings-api)
and converts the response to a data frame. In order to use it, you
should specify:

- The spatial resolution, which can be `low` (0.1 degree) or `high`
  (0.01 degree)
- The temporal resolution, which can be `daily`, `monthly`, or `yearly`.
- The variable to group by: `vessel_id`, `flag`, `gearType`, or
  `flagAndGearType`
- The date range `note: this must be one (1) year or less`
- The `geojson` region or region code (such as an EEZ code) to filter
  the raster
- The source for the specified region (currently, `eez`, `mpa`, or
  `user_json`)

### Examples

Here’s an example where we enter the geojson data manually:

> **Note**:  
> In `gwfr`, the geojson needs to be enclosed by a `{"geojson": ...}`
> tag. If you have a `geojsonsf::sf_geojson()` object, you can obtain
> the geojson object with a simple concatenation:
> `paste0('{"geojson":', your_geojson,'}')`

``` r

region_json = '{"geojson":{"type":"Polygon","coordinates":[[[-76.11328125,-26.273714024406416],[-76.201171875,-26.980828590472093],[-76.376953125,-27.527758206861883],[-76.81640625,-28.30438068296276],[-77.255859375,-28.767659105691244],[-77.87109375,-29.152161283318918],[-78.486328125,-29.45873118535532],[-79.189453125,-29.61167011519739],[-79.892578125,-29.6880527498568],[-80.595703125,-29.61167011519739],[-81.5625,-29.382175075145277],[-82.177734375,-29.07537517955835],[-82.705078125,-28.6905876542507],[-83.232421875,-28.071980301779845],[-83.49609375,-27.683528083787756],[-83.759765625,-26.980828590472093],[-83.84765625,-26.35249785815401],[-83.759765625,-25.64152637306576],[-83.583984375,-25.16517336866393],[-83.232421875,-24.447149589730827],[-82.705078125,-23.966175871265037],[-82.177734375,-23.483400654325635],[-81.5625,-23.241346102386117],[-80.859375,-22.998851594142906],[-80.15625,-22.917922936146027],[-79.453125,-22.998851594142906],[-78.662109375,-23.1605633090483],[-78.134765625,-23.40276490540795],[-77.431640625,-23.885837699861995],[-76.9921875,-24.28702686537642],[-76.552734375,-24.846565348219727],[-76.2890625,-25.48295117535531],[-76.11328125,-26.273714024406416]]]}}'

get_raster(
  spatial_resolution = "low",
  temporal_resolution = "yearly",
  group_by = "flag",
  date_range = "2021-01-01,2021-12-31",
  region = region_json,
  region_source = "user_json",
  key = key
  )
#> Rows: 5 Columns: 6
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr (1): flag
#> dbl (5): Lat, Lon, Time Range, Vessel IDs, Apparent Fishing Hours
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#> # A tibble: 5 × 6
#>     Lat   Lon `Time Range` flag  `Vessel IDs` `Apparent Fishing Hours`
#>   <dbl> <dbl>        <dbl> <chr>        <dbl>                    <dbl>
#> 1 -24.2 -77.8         2021 ESP              1                     0.42
#> 2 -24.6 -78.4         2021 ESP              2                     0.28
#> 3 -27.3 -82           2021 ESP              1                     0.43
#> 4 -24.7 -78.5         2021 ESP              1                     0.03
#> 5 -24.7 -78.6         2021 ESP              2                     0.96
```

If you want raster data from a particular EEZ, you can use the
`get_region_id()` function to get the EEZ id, enter that code in the
`region` argument of `get_raster()` instead of the geojson data
(ensuring you specify the `region_source` as `"eez"`:

``` r
# use EEZ function to get EEZ code of Cote d'Ivoire
code_eez <- get_region_id(region_name = "CIV", region_source = "eez", key = key)

get_raster(spatial_resolution = "low",
           temporal_resolution = "yearly",
           group_by = "flag",
           date_range = "2021-01-01,2021-10-01",
           region = code_eez$id,
           region_source = "eez",
           key = key)
#> Rows: 573 Columns: 6
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr (1): flag
#> dbl (5): Lat, Lon, Time Range, Vessel IDs, Apparent Fishing Hours
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#> # A tibble: 573 × 6
#>      Lat   Lon `Time Range` flag  `Vessel IDs` `Apparent Fishing Hours`
#>    <dbl> <dbl>        <dbl> <chr>        <dbl>                    <dbl>
#>  1   1.4  -6.6         2021 CPV              1                     1.27
#>  2   2.4  -4           2021 FRA              1                     1.04
#>  3   4.3  -4.1         2021 FRA              2                     3.51
#>  4   5    -5.3         2021 CHN              2                    38.4 
#>  5   5.3  -4           2021 SLV              2                    17.0 
#>  6   4    -4.3         2021 BLZ              1                     4.13
#>  7   5.1  -4.2         2021 BLZ              1                     1.99
#>  8   2    -6           2021 BLZ              1                     4.52
#>  9   1.2  -6.8         2021 BLZ              1                     2.46
#> 10   1.3  -6.7         2021 BLZ              1                     3.46
#> # ℹ 563 more rows
```

You could search for just one word in the name of the EEZ and then
decide which one you want:

``` r
(get_region_id(region_name = "France", region_source = "eez", key = key))
#> # A tibble: 3 × 3
#>      id iso3  label                           
#>   <dbl> <chr> <chr>                           
#> 1  5677 FRA   France                          
#> 2 48976 FRA   Joint regime area Italy / France
#> 3 48966 FRA   Joint regime area Spain / France

# Let's say we're interested in the French Exclusive Economic Zone, 5677
get_raster(spatial_resolution = "low",
           temporal_resolution = "yearly",
           group_by = "flag",
           date_range = "2021-01-01,2021-10-01",
           region = 5677,
           region_source = "eez",
           key = key)
#> Rows: 5611 Columns: 6
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr (1): flag
#> dbl (5): Lat, Lon, Time Range, Vessel IDs, Apparent Fishing Hours
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#> # A tibble: 5,611 × 6
#>      Lat   Lon `Time Range` flag  `Vessel IDs` `Apparent Fishing Hours`
#>    <dbl> <dbl>        <dbl> <chr>        <dbl>                    <dbl>
#>  1  49.1  -5.8         2021 FRA             26                    295. 
#>  2  49.1  -5.9         2021 FRA             21                    180. 
#>  3  49    -5.9         2021 FRA             19                    244. 
#>  4  49    -6.1         2021 FRA             21                    239. 
#>  5  49.1  -5.6         2021 FRA             18                    470. 
#>  6  51     1.6         2021 FRA             24                    317. 
#>  7  50     0           2021 FRA             28                    145. 
#>  8  49.8   0           2021 FRA             62                   1188. 
#>  9  42.6   3.2         2021 ESP              9                     27.8
#> 10  42.9   3.3         2021 FRA             18                    547. 
#> # ℹ 5,601 more rows
```

A similar approach can be used to search for a specific Marine Protected
Area, in this case the Phoenix Island Protected Area (PIPA)

``` r
# use region id function to get MPA code of Phoenix Island Protected Area
code_mpa <- get_region_id(region_name = "Phoenix", region_source = "mpa", key = key)

get_raster(spatial_resolution = "low",
           temporal_resolution = "yearly",
           group_by = "flag",
           date_range = "2015-01-01,2015-06-01",
           region = code_mpa$id[1],
           region_source = "mpa",
           key = key)
#> Rows: 93 Columns: 6
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr (1): flag
#> dbl (5): Lat, Lon, Time Range, Vessel IDs, Apparent Fishing Hours
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#> # A tibble: 93 × 6
#>      Lat   Lon `Time Range` flag  `Vessel IDs` `Apparent Fishing Hours`
#>    <dbl> <dbl>        <dbl> <chr>        <dbl>                    <dbl>
#>  1  -3.9 -176.         2015 KOR              1                     4.88
#>  2  -4   -176.         2015 KOR              1                     1.37
#>  3  -4.1 -176.         2015 KOR              1                     1.57
#>  4  -2.9 -176.         2015 FSM              1                     2.77
#>  5  -3.3 -176.         2015 <NA>             1                     1.45
#>  6  -2.8 -176.         2015 KOR              1                     9.29
#>  7  -3.5 -176.         2015 KOR              2                    12.3 
#>  8  -3.4 -176.         2015 KOR              1                     1.37
#>  9  -3.5 -176.         2015 KOR              1                    10.8 
#> 10  -3.6 -176.         2015 KOR              1                     1.08
#> # ℹ 83 more rows
```

It is also possible to filter rasters to one of the five regional
fisheries management organizations (RFMO) that manage tuna and tuna-like
species. These include `"ICCAT"`, `"IATTC"`,`"IOTC"`, `"CCSBT"` and
`"WCPFC"`.

``` r
get_raster(spatial_resolution = "low",
           temporal_resolution = "daily",
           group_by = "flag",
           date_range = "2021-01-01,2021-01-15",
           region = "ICCAT",
           region_source = "rfmo",
           key = key)
#> Rows: 114979 Columns: 6
#> ── Column specification ────────────────────────────────────────────────────────
#> Delimiter: ","
#> chr  (1): flag
#> dbl  (4): Lat, Lon, Vessel IDs, Apparent Fishing Hours
#> date (1): Time Range
#> 
#> ℹ Use `spec()` to retrieve the full column specification for this data.
#> ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
#> # A tibble: 114,979 × 6
#>      Lat   Lon `Time Range` flag  `Vessel IDs` `Apparent Fishing Hours`
#>    <dbl> <dbl> <date>       <chr>        <dbl>                    <dbl>
#>  1  68.7 -51.4 2021-01-08   GRL              1                     0.25
#>  2  68.8 -51.2 2021-01-05   GRL              1                     4.78
#>  3  68.8 -51.2 2021-01-04   GRL              2                     0.73
#>  4  66.9 -24.7 2021-01-03   ISL             11                    14.7 
#>  5  66.9 -24.6 2021-01-03   ISL             12                    34.5 
#>  6  66.9 -24.5 2021-01-03   ISL             13                    40.2 
#>  7  66.8 -24.3 2021-01-04   ISL             15                    42.2 
#>  8  67   -24   2021-01-04   ISL              3                     6.13
#>  9  66.9 -23.9 2021-01-04   ISL              2                     1.43
#> 10  66.9 -24.1 2021-01-03   ISL              3                     3.4 
#> # ℹ 114,969 more rows
```

The `get_region_id()` function also works in reverse. If a region id is
passed as a `numeric` to the function as the `region_name`, the
corresponding region label or iso3 can be returned. This is especially
useful when events are returned with regions.

``` r
# using same example as above
get_event(event_type = "fishing",
          vessel = usa_trawler_ids,
          start_date = "2020-01-01",
          end_date = "2020-02-01",
          include_regions = TRUE,
          key = key
          ) %>%
  # extract EEZ id code
  dplyr::mutate(eez = as.character(purrr::map(purrr::map(regions, pluck, "eez"), 
                                              paste0, collapse = ","))) %>%
  dplyr::select(id, type, start, end, lat, lon, eez) %>%
  dplyr::rowwise() %>%
  dplyr::mutate(eez_name = get_region_id(region_name = as.numeric(eez),
                                         region_source = "eez",
                                         key = key)$label)
#> [1] "Downloading 106 events from GFW"
#> # A tibble: 106 × 8
#> # Rowwise: 
#>    id           type  start               end                   lat    lon eez  
#>    <chr>        <chr> <dttm>              <dttm>              <dbl>  <dbl> <chr>
#>  1 06783a15944… fish… 2020-01-01 15:56:25 2020-01-02 00:41:57  35.1  -76.0 8456 
#>  2 4891aab6703… fish… 2020-01-02 01:55:51 2020-01-03 00:05:57  35.0  -76.0 8456 
#>  3 d75af335992… fish… 2020-01-02 23:31:48 2020-01-03 04:37:19  41.1  -71.4 8456 
#>  4 8ddaf495862… fish… 2020-01-03 00:39:08 2020-01-03 02:49:08  35.0  -76.0 8456 
#>  5 c85b3f8c738… fish… 2020-01-03 15:51:15 2020-01-03 18:24:44  39.8  -73.9 8456 
#>  6 1bee4c2bbe2… fish… 2020-01-05 00:35:43 2020-01-05 06:11:43  39.7  -73.9 8456 
#>  7 379d452b49e… fish… 2020-01-05 04:58:45 2020-01-05 06:31:45  43.7 -124.  8456 
#>  8 0b45ad5daf1… fish… 2020-01-06 06:20:19 2020-01-08 02:46:19  39.6  -73.9 8456 
#>  9 04d20daaf37… fish… 2020-01-06 21:12:01 2020-01-07 02:35:11  34.6  -76.6 8456 
#> 10 2ad00a03bf5… fish… 2020-01-07 13:37:54 2020-01-07 15:16:54  34.7  -76.8 8456 
#> # ℹ 96 more rows
#> # ℹ 1 more variable: eez_name <chr>
```

## Contributing

We welcome all contributions to improve the package! Please read our
[Contribution
Guide](https://github.com/GlobalFishingWatch/gfwr/blob/main/Contributing.md)
and reach out!
