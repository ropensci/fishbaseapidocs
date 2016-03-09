---
title: API Reference

language_tabs:
  - http
  - shell
  - r
  - ruby

toc_footers:
  - <a href='http://www.fishbase.org/search.php'>Fishbase</a>
  - <a href='http://www.sealifebase.org'>Sealifebase</a>
  - <a href='https://ropensci.org/'>rOpenSci</a>
  - <a href='https://github.com/ropensci/'>rOpenSci @ GitHub</a>
  - <a href='https://github.com/ropensci/fishbaseapidocs'>Contribute to these docs</a>

search: true
---

# Introduction

Welcome to the Fishbase API! You can use our API to access Fishbase data.

We have examples for Shell (using `curl` and `jq`), R, and Ruby! You can view code examples in the dark area to the right (or below if on mobile), and you can switch the programming language of the examples with the tabs in the top right.

<aside class="notice">
We're still in the process of filling out examples...Some routes below have examples, while some do not.
</aside>

# Base URL

[https://fishbase.ropensci.org](https://fishbase.ropensci.org)

# Fishbase vs. Sealifebase

In addition to Fishbase, we also server [Sealifebase](http://www.sealifebase.org/) data from this API.

To use Sealifebase API insted of Fishbase, add `/sealifebase` to the base URL, like:

[https://fishbase.ropensci.org/sealifebase](https://fishbase.ropensci.org/sealifebase)

Everything described below should work the same for both Fishbase and Sealife base APIs.

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

Lists all the routes, and gives some indicaion of what can be passed to them.

```shell
curl "https://fishbase.ropensci.org/heartbeat" |  jq .
```

The above command returns JSON structured like this:

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
curl "https://fishbase.ropensci.org/comnames?" |  jq .
```

> The above command returns JSON structured like this:

```json
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

common_names(species_list = "Bolbometopon muricatum")
```

> returns

```r
Source: local data frame [62 x 6]

               ComName SpecCode C_Code Language        Genus   Species
                 (chr)    (int)  (chr)    (chr)        (chr)     (chr)
1            Aliyakyak     5537    608  Visayan Bolbometopon muricatum
2                Angke     5537    360     Bajo Bolbometopon muricatum
3                Angke     5537    360    Malay Bolbometopon muricatum
4                Angol     5537    608    Bikol Bolbometopon muricatum
5                Bayan     5537    458    Malay Bolbometopon muricatum
6        Bayan bonggol     5537    458    Malay Bolbometopon muricatum
7             Berdebed     5537    585  Palauan Bolbometopon muricatum
8                Boila     5537    090     Gela Bolbometopon muricatum
9     Bulepapegøjefisk     5537    208   Danish Bolbometopon muricatum
10 Bumphead parrotfish     5537    036  English Bolbometopon muricatum
..                 ...      ...    ...      ...          ...       ...
```

```ruby
require 'faraday'
require 'multi_json'

conn = Faraday.new(url: 'https://fishbase.ropensci.org')
res = conn.get 'comnames', {"SpecCode": 5537}
dat = MultiJson.load(res.body)
dat['data'].collect{ |x| x['ComName']}
```

> The above command returns a Hash like:

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

## country

> GET [/country{?limit}{?offset}{?fields}]

Search the country table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## diet

> GET [/diet{?limit}{?offset}{?fields}]

Search the fish diet table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## ecology

> GET [/ecology{?limit}{?offset}{?fields}]

Search the ecology table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## ecosystem

> GET [/ecosystem{?limit}{?offset}{?fields}]

Search the ecosystem table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## fao areas routes

### faoareas

> GET [/faoareas{?limit}{?offset}{?fields}]

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

### faoareas by id

> GET [/faoareas{id}]

List faoareas by id

+ Parameters
    + id (required, integer, `5`) ... faoarea id `number`.

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## faoarref

> GET [/faoarref{?limit}{?offset}{?fields}]

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

### faoarref by id

> GET [/faoarref{id}]

List faoareas by id

+ Parameters
    + id (required, integer, `5`) ... faoarref id `number`.

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## fecundity

> GET [/fecundity{?limit}{?offset}{?fields}]

Search the fecundity table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## fooditems

> GET [/fooditems{?limit}{?offset}{?fields}]

Search the fooditems table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## Genera

> GET [/genera{?genus}{?species}{?limit}{?offset}{?fields}]

List genera

+ Parameters
    + genus (string, optional, `Abalistes`) ... String `name` of a genus.
    + species (string, optional, `filamentosus`) ... String `name` of a specific epithet.

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## Genera by id

> GET [/genera/{id}]

List genera by id

+ Parameters
    + id (required, integer, `5`) ... Genus id `number`.

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## intrcase

> GET [/intrcase{?limit}{?offset}{?fields}]

Search the intrcase table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## listfields

> GET [/listfields{?fields}{?exact}]

List fields across all tables. Optionally, search for particular fields. In addition, toggle the `exact` parameter to search for an exact match.

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

## maturity

> GET [/maturity{?limit}{?offset}{?fields}]

Search the maturity table

+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

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

## taxa

> GET [/taxa{?limit}{?offset}{?fields}]

Search the taxa table

+ Parameters
    + genus (string, optional) Genus name
    + species (string, optional) Specific epithet name
+ Response 200
    + [Headers](#response-headers)
    + [Body](#response-bodies)

# API Clients

## R - rfishbase

rOpenSci maintains this client, and tracks changes in the API closesly

* On GitHub: [https://github.com/ropensci/rfishbase](https://github.com/ropensci/rfishbase)
* On CRAN: [https://cran.rstudio.com/web/packages/rfishbase/](https://cran.rstudio.com/web/packages/rfishbase/)
* Examples are given throughout these API docs.
* See also the [vignette for rfishbase](https://github.com/ropensci/rfishbase/blob/master/vignettes/tutorial.Rmd)

## Ruby

Not made yet ...
