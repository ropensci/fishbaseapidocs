---
title: API Reference

language_tabs:
  - http
  - shell
  - r
  - ruby

toc_footers:
  - <a href='http://www.fishbase.org/search.php' target="_blank">Fishbase</a>
  - <a href='http://www.sealifebase.org' target="_blank">Sealifebase</a>
  - <a href='https://ropensci.org/' target="_blank">rOpenSci</a>
  - <a href='https://github.com/ropensci/' target="_blank">rOpenSci @ GitHub</a>
  - <a href='https://github.com/ropensci/fishbaseapidocs' target="_blank">Contribute to these docs</a>
  - <a href='https://ropensci.github.io/fishbasestatus/' target="_blank">API Status!</a>

search: true
---

# Introduction

Welcome to the Fishbase API! You can use our API to access Fishbase data.

We have examples for Shell (using `curl` and `jq`), R, and Ruby! You can view code examples in the dark area to the right (or below if on mobile), and you can switch the programming language of the examples with the tabs in the top right.

<aside class="notice">
We're still in the process of filling out examples...Some routes below have examples, while some do not.
</aside>

<aside class="warning">
Check on the status of the API at our <a href="https://ropensci.github.io/fishbasestatus/">status page</a>
</aside>

# Base URL

<https://fishbase.ropensci.org/>

# Fishbase vs. Sealifebase

In addition to Fishbase, we also server [Sealifebase](http://www.sealifebase.org/) data from this API.

To use Sealifebase API insted of Fishbase, add `/sealifebase` to the base URL, like:

[https://fishbase.ropensci.org/sealifebase](https://fishbase.ropensci.org/sealifebase)

Everything described below should work the same for both Fishbase and Sealife base APIs.

# Database versions

<aside class="warning">
Database versions are supported in the Fishbase API only at this time. We may support multiple versions of Sealifebase in the future.
</aside>

The Fishbase API supports multiple different versions of the underlying database.

You can request a different database version with a HTTP header like `Accept: application/vnd.ropensci.v3+json`

Where `v3` represents "version 3". Likewise, `v2` is "version 2", and `v1` is "version 1". These versions correspond to specific dates associated with different database versions. The current versions for Fishbase are:

- 201505
- 201604
- 201703

The date format `YYYYMM` is four digits for year, then two digits for month, with no spaces/characters between them.

By default, we return the latest date version.

See the [`/versions`](#versions) route for description of the different versions and their names.

In the R client `rfishbase` the database version will be controlled by a parameter or option/env var, so users won't have to pass headers themselves.


# HTTP methods

This is essentially a `read only` API. That is, we only allow `GET` (and `HEAD`) requests on this API.

Requests of all other types will be rejected with appropriate `405` code, including `POST`, `PUT`, `COPY`, `HEAD`, `DELETE`, etc.

# Response Codes

* 200 (OK) - request good!
* 302 (Found) - the root `/`, redirects to `/heartbeat`, and `/docs` redirects to these documents
* 400 (Bad request) - When you have a malformed request, fix it and try again
* 404 (Not found) - When you request a route that does not exist, fix it and try again
* 405 (Method not allowed) - When you use a prohibited HTTP method (we only allow `GET` and `HEAD`)
* 500 (Internal server error) - Server got itself in trouble; get in touch with us. (in [Issues](https://github.com/ropensci/fishbaseapi/issues))

> `400` responses will look something like

```
HTTP/1.1 400 Bad Request
Cache-Control: public, must-revalidate, max-age=60
Connection: close
Content-Length: 61
Content-Type: application/json
Date: Thu, 26 Feb 2015 23:27:57 GMT
Server: nginx/1.7.9
Status: 400 Bad Request
X-Content-Type-Options: nosniff

{
    "error": "invalid request",
    "message": "maximum limit is 5000"
}
```

> `404` responses will look something like

```
HTTP/1.1 404 Not Found
Cache-Control: public, must-revalidate, max-age=60
Connection: close
Content-Length: 27
Content-Type: application/json
Date: Thu, 26 Feb 2015 23:27:16 GMT
Server: nginx/1.7.9
Status: 404 Not Found
X-Cascade: pass
X-Content-Type-Options: nosniff

{
    "error": "route not found"
}
```

> `405` responses will look something like (with an empty body)

```
HTTP/1.1 405 Method Not Allowed
Access-Control-Allow-Methods: HEAD, GET
Access-Control-Allow-Origin: *
Cache-Control: public, must-revalidate, max-age=60
Connection: close
Content-Length: 0
Content-Type: application/json; charset=utf8
Date: Mon, 27 Jul 2015 20:48:27 GMT
Server: nginx/1.9.3
Status: 405 Method Not Allowed
X-Content-Type-Options: nosniff
```

> `500` responses will look something like

```
HTTP/1.1 500 Internal Server Error
Cache-Control: public, must-revalidate, max-age=60
Connection: close
Content-Length: 24
Content-Type: application/json
Date: Thu, 26 Feb 2015 23:19:57 GMT
Server: nginx/1.7.9
Status: 500 Internal Server Error
X-Content-Type-Options: nosniff

{
    "error": "server error"
}
```

Occassionally you may get a MySQL error message in your `error` slot. Get in touch
with us if that happens as that shouldn't be happening, and we'll need to fix
something on our end.

# Response headers

Response headers give the following noteable things:

* allowed methods on a route
* allowed origins
* cache information
* body content length
* content type (JSON)
* date
* server info
* http status code

> `200` response header will look something like

```
Access-Control-Allow-Methods: HEAD, GET
Access-Control-Allow-Origin: *
Cache-Control: public, must-revalidate, max-age=60
Connection: close
Content-Length: 10379
Content-Type: application/json; charset=utf8
Date: Mon, 09 Mar 2015 23:01:23 GMT
Server: nginx/1.7.10
Status: 200 OK
X-Content-Type-Options: nosniff
```

# Response bodies

> Response bodies generally look like:

```json
{
    "count": 1,
    "returned": 1,
    "data": [
      {
        "AnaCat": "potamodromous",
        "AquacultureRef": 12108,
        "Aquarium": "never/rarely",
        "AquariumFishII": " ",
        "AquariumRef": null,
        "Author": "(Linnaeus, 1758)"
      }
    ],
    "error": null
}
```

Responses have 4 slots:

* count: Number records found
* returned: Number records returned
* error: If an error did not occur this is `null`, otherwise, an error message.
* data: The hash of data if any data returned. If no data found, this is an empty hash (hash of length zero)

Responses with errors also have the same 4 slots, but no data returned, and there should be an
informative error message.

# Media Types

We serve up only JSON in this API. All responses will have `Content-Type: application/json; charset=utf8`.

# Pagination

The query parameters `limit` (default = 10) and `offset` (default = 0) can be used
to toggle the number of results you get back and what record you start at. This
doesn't apply to some routes, which don't accept any parameters (e.g., [docs](#docs)).

The response body from the server will include data on records found in `count` and number returned in `returned`:

* `"count": 1056`
* `"returned": 10`

Ideally, we'd put in a helpful [links object](http://jsonapi.org/format/#fetching-pagination) - hopefully we'll get that done in the future.

# Authentication

We don't require any. Cheers :)

# Parameters

## Common parameters

+ limit (integer, optional) `number` of records to return.
    + Default: `10`
+ offset (integer, optional) Record `number` to start at.
    + Default: `0`
+ fields (string, optional) Comma-separated `string` of fieds to return.
    + Example: `SpecCode,Vulnerability`

Above parameters common to all routes except:

* [root](#root)
* [heartbeat](#heartbeat)
* [docs](#docs)

In addition, these routes do not support `limit` or `offset`:

* [listfields](#listfields)

## Additional parameters

Right now, any field that is returned from a route can also be queried on, except for the [/taxa route](#taxa), which only accepts `species` and `genus` in addition to the common parameters. All of the fields from each route are too long to list here - inspect data returned from a small data request, then change your query as desired.

Right now, parameters that are not found are silently dropped. For example, if you query with `/species?foo=bar` in a query, and `foo` is not a field in `species` route, then the `foo=bar` part is ignored. We may in the future error when parameters are not found.

# Routes

<aside class="notice">
Shell examples below using <code>curl</code> sometimes use the command line tool <code>jq</code>,
a tool to parse JSON. If you don't want to use <code>jq</code> - just remove the pipe and
everything after it. If you do, see <a href="https://stedolan.github.io/jq/">stedolan.github.io/jq</a>
for installation and usage.
</aside>

## Root

> GET [/]

`GET https://fishbase.ropensci.org`

```shell
curl -L "https://fishbase.ropensci.org"
```

Redirects to `https://fishbase.ropensci.org/heartbeat`

<aside class="notice">
Note that when using <code>curl</code> you need to use the <code>-L</code> flag
to follow redirects.
</aside>

## Heartbeat

> GET [/heartbeat]

Lists all the routes, and gives some indicaion of what can be passed to them.

```shell
curl "https://fishbase.ropensci.org/heartbeat" |  jq .
```

```shell
{
  "routes": [
    "/docs/:table?",
    "/heartbeat",
    "/mysqlping",
    "/listfields",
    "/comnames?<params>",
    "/country?<params>"
  ]
}
```

```r
library("rfishbase")
library("httr")
library("jsonlite")

res <- rfishbase::heartbeat()
jsonlite::fromJSON(httr::content(res, "text"))
```

```r
$routes
 [1] "/docs/:table?"            "/heartbeat"               "/mysqlping"               "/listfields"
 [5] "/comnames?<params>"       "/country?<params>"
```

```ruby
require 'faraday'
require 'multi_json'

conn = Faraday.new(url: 'https://fishbase.ropensci.org')
res = conn.get 'heartbeat'
MultiJson.load(res.body)
```

The above command returns a Hash like:

```ruby
=> {"routes"=>
  ["/docs/:table?",
   "/heartbeat",
   "/mysqlping",
   "/listfields",
   "/comnames?<params>",
   "/country?<params>"]}
```

## docs

> GET [/docs]

Get brief description of each table in the Fishbase database.

+ Response 200 (application/json)
    + [Headers](#response-headers)
    + [Body](#response-bodies)


## versions

> GET [/versions]

Get Fishbase database versions.

+ Response 200 (application/json)
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/versions" |  jq .
```

```shell
{
  "data": [
    {
      "v1": {
        "name": "201505",
        "date_released": "2015-05-20"
      },
      "v2": {
        "name": "201604",
        "date_released": "2016-04-28"
      },
      "v3": {
        "name": "201703",
        "date_released": "2017-03-31"
      }
    }
  ],
  "error": null
}
```

## docs by table

> GET [/docs/{table}]

Get all field names in a table. In the future, the returned data will include metadata on what each field means (understandable to a human), and what kind of data each field holds.

+ Response 200 (application/json)
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## comnames

> GET [/comnames{?limit}{?offset}{?fields}]

Search the common names table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/comnames?limit=1" |  jq .

{
  "count": 319060,
  "returned": 1,
  "data": [
    {
      "autoctr": 236404,
      "ComName": " ???????",
      "Transliteration": "Varimeen",
      "StockCode": 135,
      "SpecCode": 121,
      "C_Code": "356",
      "Language": "Telugu",
      "Script": "Telugu",
...
```

```r
library("rfishbase")
common_names(species_list = "Bolbometopon muricatum")
```

```r
# A tibble: 64 x 6
            ComName SpecCode C_Code         Language        Genus   Species
              <chr>    <int>  <chr>            <chr>        <chr>     <chr>
 1        Aliyakyak     5537    608          Visayan Bolbometopon muricatum
 2            Angke     5537    360 Bahasa Indonesia Bolbometopon muricatum
 3            Angke     5537    360             Bajo Bolbometopon muricatum
 4            Angke     5537    360            Malay Bolbometopon muricatum
 5            Angol     5537    608            Bikol Bolbometopon muricatum
 6            Bayan     5537    458            Malay Bolbometopon muricatum
 7    Bayan bonggol     5537    458            Malay Bolbometopon muricatum
 8         Berdebed     5537    585          Palauan Bolbometopon muricatum
 9            Boila     5537    090             Gela Bolbometopon muricatum
10 Bulepapegøjefisk     5537    208           Danish Bolbometopon muricatum
# ... with 54 more rows
```

```ruby
require 'faraday'
require 'multi_json'

conn = Faraday.new(url: 'https://fishbase.ropensci.org')
res = conn.get 'comnames', {"SpecCode": 5537}
dat = MultiJson.load(res.body)
dat['data'].collect{ |x| x['ComName']}
```

```ruby
=> ["Aliyakyak",
 "Angke",
 "Angke",
 "Angol",
 "Bayan",
 "Bayan bonggol",
 "Berdebed",
 "Boila",
 "Bulepapegøjefisk",
 "Bumphead parrotfish"]
```

## countref

> GET [/countref{?limit}{?offset}{?fields}]

Count ref

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/countref/?limit=1" |  jq .
```


## country

> GET [/country{?limit}{?offset}{?fields}]

Search the country table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/country/?limit=1" |  jq .
```

## diet

> GET [/diet{?limit}{?offset}{?fields}]

Search the fish diet table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/diet/?limit=1" |  jq .
```

## ecology

> GET [/ecology{?limit}{?offset}{?fields}]

Search the ecology table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/ecology/?limit=1" |  jq .
```

## ecosystem

> GET [/ecosystem{?limit}{?offset}{?fields}]

Search the ecosystem table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/ecosystem/?limit=1" |  jq .
```

## fao areas routes

### faoareas

> GET [/faoareas{?limit}{?offset}{?fields}]

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/faoareas/?limit=1" |  jq .

{
  "count": 60522,
  "returned": 1,
  "data": [
    {
      "autoctr": 483,
      "AreaCode": 1,
      "SpecCode": 2,
      "StockCode": 1,
      "Status": "endemic",
      "Entered": 2,
      "DateEntered": "1990-10-19T00:00:00.000Z",
      "Modified": 675,
      "DateModified": "2011-02-11T17:16:45.000Z",
      "Expert": null,
      "DateChecked": null,
      "TS": null
    }
  ],
  "error": null
}
```

### faoareas by id

> GET [/faoareas{id}]

List faoareas by id

+ Parameters
    + id (required, integer, `5`) ... faoarea id `number`.

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/faoareas/4" |  jq .
```

## faoarref

> GET [/faoarref{?limit}{?offset}{?fields}]

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/faoarref?limit=1" |  jq .
```

### faoarref by id

> GET [/faoarref{id}]

List faoareas by id

+ Parameters
    + id (required, integer, `5`) ... faoarref id `number`.

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/faoarref/1" |  jq .
```

## fecundity

> GET [/fecundity{?limit}{?offset}{?fields}]

Search the fecundity table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/fecundity?limit=1" |  jq .
```

## fooditems

> GET [/fooditems{?limit}{?offset}{?fields}]

Search the fooditems table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/fooditems?limit=1" |  jq .

{
  "count": 53598,
  "returned": 1,
  "data": [
    {
      "autoctr": 38,
      "StockCode": 1,
      "SpecCode": 2,
      "Locality": "Sudd swamps, River Nile.",
      "C_Code": "736",
      "FoodsRefNo": 6160,
      "FoodI": "detritus",
      "PreyStage": "n.a./others",
      "FoodII": "detritus",
      "FoodIII": "debris",
      "Commoness": 23,
      "CommonessII": "very common (21-50%)",
      "Foodgroup": "unidentified",
      "Foodname": "< 1 mm organic debris",
      "PreySpecCode": null,
      "PreySpecCodeSLB": null,
      "AlphaCode": null,
      "PreyTroph": null,
      "PreySeTroph": null,
      "TrophRefNo": null,
      "PredatorStage": "juv./adults",
      "Locality2": "Locality: Sudd swamps, River Nile; no. of fish examined=48; size=2.3-35.8 cm SL. Also Ref. 42341.",
      "Entered": 34,
      "Dateentered": "1993-10-07T00:00:00.000Z",
      "Modified": 34,
      "Datemodified": "2003-02-24T00:00:00.000Z",
      "Expert": null,
      "Datechecked": null,
      "TS": null
    }
  ],
  "error": null
}
```

## Genera

> GET [/genera{?genus}{?species}{?limit}{?offset}{?fields}]

List genera

+ Parameters
    + genus (string, optional, `Abalistes`) ... String `name` of a genus.
    + species (string, optional, `filamentosus`) ... String `name` of a specific epithet.

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/genera?genus=Abalistes" |  jq .count

10991
```

## Genera by id

> GET [/genera/{id}]

List genera by id

+ Parameters
    + id (required, integer, `5`) ... Genus id `number`.

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/genera/5/" |  jq .
```

## intrcase

> GET [/intrcase{?limit}{?offset}{?fields}]

Search the intrcase table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/intrcase?limit=1" |  jq .
```

## listfields

> GET [/listfields{?fields}{?exact}]

List fields across all tables. Optionally, search for particular fields. In addition, toggle the `exact` parameter to search for an exact match.

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/listfields" |  jq .
```

## maturity

> GET [/maturity{?limit}{?offset}{?fields}]

Search the maturity table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/maturity?limit=1" |  jq .
```

## morphdat

> GET [/morphdat{?limit}{?offset}{?fields}]

Search the morphdat table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## morphmet

> GET [/morphmet{?limit}{?offset}{?fields}]

Search the morphmet table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## occurrence

> GET [/occurrence{?limit}{?offset}{?fields}]

Search the occurrence table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## oxygen

> GET [/oxygen{?limit}{?offset}{?fields}]

Search the oxygen table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## popchar

> GET [/popchar{?limit}{?offset}{?fields}]

Search the popchar table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## popgrowth

> GET [/popgrowth{?limit}{?offset}{?fields}]

Search the popgrowth table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## poplf

> GET [/poplf{?limit}{?offset}{?fields}]

Search the poplf table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)


## popll

> GET [/popll{?limit}{?offset}{?fields}]

Search the popll table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## popqb

> GET [/popqb{?limit}{?offset}{?fields}]

Search the popqb table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)


## poplw

> GET [/poplw{?limit}{?offset}{?fields}]

Search the poplw table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## predats

> GET [/predats{?limit}{?offset}{?fields}]

Search the predats table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)


## ration

> GET [/ration{?limit}{?offset}{?fields}]

Search the ration table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## refrens

> GET [/refrens{?limit}{?offset}{?fields}]

Search the refrens table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## reproduc

> GET [/reproduc{?limit}{?offset}{?fields}]

Search the reproduc table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## species

> GET [/species{?genus}{?species}{?limit}{?offset}{?fields}]

List species

+ Parameters
    + genus (string, optional) ... String `name` of a genus.
    + species (string, optional) ... String `name` of a specific epithet.

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/species?genus=Labroides&species=bicolor" | jq .
```

```r
tibble::as_tibble(species("Labroides bicolor"))

# A tibble: 1 x 100
            sciname     Genus Species SpeciesRefNo              Author
              <chr>     <chr>   <chr>        <int>               <chr>
1 Labroides bicolor Labroides bicolor         2334 Fowler & Bean, 1928
# ... with 95 more variables: FBname <chr>, PicPreferredName <chr>,
#   PicPreferredNameM <lgl>, PicPreferredNameF <lgl>, PicPreferredNameJ <lgl>,
#   FamCode <int>, Subfamily <lgl>, GenCode <int>, SubGenCode <lgl>,
#   BodyShapeI <chr>, Source <chr>, AuthorRef <lgl>, Remark <lgl>,
#   TaxIssue <int>, Fresh <int>, Brack <int>, Saltwater <int>,
#   DemersPelag <chr>, AnaCat <lgl>, MigratRef <lgl>, DepthRangeShallow <int>,
#   DepthRangeDeep <int>, DepthRangeRef <int>, DepthRangeComShallow <int>,
#   DepthRangeComDeep <int>, DepthComRef <int>, LongevityWild <lgl>,
#   LongevityWildRef <lgl>, LongevityCaptive <lgl>, LongevityCapRef <lgl>,
#   Vulnerability <dbl>, Length <dbl>, LTypeMaxM <chr>, LengthFemale <lgl>,
#   LTypeMaxF <lgl>, MaxLengthRef <int>, CommonLength <lgl>, LTypeComM <lgl>,
#   CommonLengthF <lgl>, LTypeComF <lgl>, CommonLengthRef <lgl>, Weight <lgl>,
#   WeightFemale <lgl>, MaxWeightRef <lgl>, Pic <lgl>, PictureFemale <lgl>,
#   LarvaPic <lgl>, EggPic <lgl>, ImportanceRef <lgl>, Importance <lgl>,
#   PriceCateg <chr>, PriceReliability <chr>, Remarks7 <lgl>,
#   LandingStatistics <lgl>, Landings <lgl>, MainCatchingMethod <lgl>,
#   II <lgl>, MSeines <int>, MGillnets <int>, MCastnets <int>, MTraps <int>,
#   MSpears <int>, MTrawls <int>, MDredges <int>, MLiftnets <int>,
#   MHooksLines <int>, MOther <int>, UsedforAquaculture <chr>, LifeCycle <lgl>,
#   AquacultureRef <lgl>, UsedasBait <chr>, BaitRef <lgl>, Aquarium <chr>,
#   AquariumFishII <chr>, AquariumRef <int>, GameFish <int>, GameRef <lgl>,
#   Dangerous <chr>, DangerousRef <lgl>, Electrogenic <chr>, ElectroRef <lgl>,
#   Complete <lgl>, GoogleImage <int>, Comments <chr>, Profile <lgl>,
#   PD50 <dbl>, Emblematic <int>, Entered <int>, DateEntered <chr>,
#   Modified <int>, DateModified <chr>, Expert <int>, DateChecked <chr>,
#   TS <lgl>, SpecCode <int>
```

## Species by id

> GET [/species/{id}]

List species by id

+ Parameters
    + id (integer, required, `5`) ... Species id `number`.

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## spawning

> GET [/spawning{?limit}{?offset}{?fields}]

Search the spawning table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## speed

> GET [/speed{?limit}{?offset}{?fields}]

Search the speed table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## stocks

> GET [/stocks{?limit}{?offset}{?fields}]

Search the stocks table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## swimming

> GET [/swimming{?limit}{?offset}{?fields}]

Search the swimming table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## synonyms

> GET [/synonyms{?limit}{?offset}{?fields}]

Search the synonyms table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/synonyms?SpecCode=5537" | jq '[.data[] | {genus: .SynGenus, species: .SynSpecies}]'
```

```shell
[
  {
    "genus": "Bolbomatopon",
    "species": "muricatum"
  },
  {
    "genus": "Bolbometopon",
    "species": "muricantum"
  },
  {
    "genus": "Bolbometopon",
    "species": "muricatum"
  },
  {
    "genus": "Bolbometopon",
    "species": "muricatus"
  },
  {
    "genus": "Bulbometopon",
    "species": "muricatum"
  },
  {
    "genus": "Callyodon",
    "species": "muricatus"
  },
  {
    "genus": "Scarus",
    "species": "muricatus"
  }
]
```

```r
synonyms("Callyodon murica")

# A tibble: 1 x 12
  SynCode SpecCode  SynGenus SynSpecies  Status Valid       Synonymy
    <int>    <int>     <chr>      <chr>   <chr> <lgl>          <chr>
1   59804     5537 Callyodon  muricatus synonym FALSE senior synonym
# ... with 5 more variables: Combination <chr>, Misspelling <lgl>,
#   CoL_ID <lgl>, TSN <lgl>, WoRMS_ID <int>
```

## taxa

> GET [/taxa{?limit}{?offset}{?fields}]

Search the taxa table

+ Parameters
    + genus (string, optional) Genus name
    + species (string, optional) Specific epithet name
+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

```shell
curl "https://fishbase.ropensci.org/taxa?Species=parmiferus" | jq .

{
  "count": 1,
  "returned": 1,
  "data": [
    {
      "SpecCode": 48051,
      "Genus": "Asprocottus",
      "Species": "parmiferus",
      "SpeciesRefNo": 26334,
      "Author": "Taliev, 1955",
      "FBname": null,
      "FamCode": 584,
      "GenCode": 5868,
      "SubGenCode": null,
      "Remark": null,
      "SubFamily": null,
      "Family": "Abyssocottidae",
      "Order": "Scorpaeniformes",
      "Class": "Actinopterygii"
    }
  ],
  "error": null
}
```

# API Clients

## R - rfishbase

rOpenSci maintains this client, and tracks changes in the API closesly

* On GitHub: [https://github.com/ropensci/rfishbase](https://github.com/ropensci/rfishbase)
* On CRAN: [https://cran.rstudio.com/web/packages/rfishbase/](https://cran.rstudio.com/web/packages/rfishbase/)
* Examples are given throughout these API docs.
* See also the [vignette for rfishbase](https://github.com/ropensci/rfishbase/blob/master/vignettes/tutorial.Rmd)

## Ruby

Not made yet ...
