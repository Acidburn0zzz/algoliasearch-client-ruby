Algolia Search API Client for Ruby
==================

This Ruby client let you easily use the Algolia Search API from your backend.
The service is currently in Beta, you can request an invite on our [website](http://www.algolia.com/pricing/).

Table of Content
-------------
**Get started**

1. [Setup](#setup) 
1. [Quick Start](#quick-start)

**Commands reference**

1. [Search](#search)
1. [Add a new object](#add-a-new-object-in-the-index)
1. [Update an object](#update-an-existing-object-in-the-index)
1. [Get an object](#get-an-object)
1. [Delete an object](#delete-an-object)
1. [Index settings](#index-settings)
1. [List indexes](#list-indexes)
1. [Delete an index](#delete-an-index)
1. [Wait indexing](#wait-indexing)
1. [Batch writes](#batch-writes)
1. [Security / User API Keys](#security--user-api-keys)

Setup
-------------
To setup your project, follow these steps:

 2. Install AlgoliaSearch using <code>gem install algoliasearch</code>.
 2. Initialize the client with your ApplicationID, API-Key and list of hostnames (you can find all of them on your Algolia account).

```ruby
require 'rubygems'
require 'algoliasearch'

Algolia.init :hosts             => ["YourHostname-1.algolia.com", 
                                    "YourHostname-2.algolia.com", 
                                    "YourHostname-3.algolia.com"],
             :application_id    => "YourApplicationID",
             :api_key           => "YourAPIKey"
```

Quick Start
-------------
This quick start is a 30 seconds tutorial where you can discover how to index and search objects.

Without any prior-configuration, you can index the 1000 world's biggest cities in the ```cities``` index with the following code:
```ruby
index = Algolia::Index.new("cities")
batch = JSON.parse(File.read("1000-cities.json"))
index.add_objects(batch["objects"])
```
The [1000-cities.json](https://github.com/algolia/algoliasearch-client-ruby/blob/master/1000-cities.json) file contains city names extracted from [Geonames](http://www.geonames.org).

You can then start to search for a city name (even with typos):
```ruby
puts index.search('san fran').to_json
puts index.search('loz anqel').to_json
```

Settings can be customized to tune the index behavior. For example you can add a custom sort by population to the already good out-of-the-box relevance to raise bigger cities above smaller ones. To update the settings, use the following code:
```ruby
index.set_settings({"customRanking" => ["desc(population)", "asc(name)"]})
```

And then search for all cities that start with an "s":
```ruby
puts index.search('s').to_json
```

Search
-------------
> **Opening note:** If you are building a web application, you may be more interested in using our [javascript client](https://github.com/algolia/algoliasearch-client-js) to send queries. It brings two benefits: (i) your users get a better response time by avoiding to go threw your servers, and (ii) it will offload your servers of unnecessary tasks.

To perform a search, you just need to initialize the index and perform a call to the search function.<br/>
You can use the following optional arguments:

 * **attributes**: a string that contains the names of attributes to retrieve separated by a comma.<br/>By default all attributes are retrieved.
 * **attributesToHighlight**: a string that contains the names of attributes to highlight separated by a comma.<br/>By default indexed attributes are highlighted. Numerical attributes cannot be highlighted. A **matchLevel** is returned for each highlighted attribute and can contain: "full" if all the query terms were found in the attribute, "partial" if only some of the query terms were found, or "none" if none of the query terms were found.
 * **attributesToSnippet**: a string that contains the names of attributes to snippet alongside the number of words to return (syntax is 'attributeName:nbWords'). Attributes are separated by a comma (Example: "attributesToSnippet=name:10,content:10").<br/>By default no snippet is computed.
 * **minWordSizeForApprox1**: the minimum number of characters in a query word to accept one typo in this word.<br/>Defaults to 3.
 * **minWordSizeForApprox2**: the minimum number of characters in a query word to accept two typos in this word.<br/>Defaults to 7.
 * **getRankingInfo**: if set to 1, the result hits will contain ranking information in _rankingInfo attribute.
 * **page**: *(pagination parameter)* page to retrieve (zero base).<br/>Defaults to 0.
 * **hitsPerPage**: *(pagination parameter)* number of hits per page.<br/>Defaults to 10.
 * **aroundLatLng**: search for entries around a given latitude/longitude (specified as two floats separated by a comma).<br/>For example `aroundLatLng=47.316669,5.016670`).<br/>You can specify the maximum distance in meters with the **aroundRadius** parameter (in meters).<br/>At indexing, you should specify geoloc of an object with the _geoloc attribute (in the form `{"_geoloc":{"lat":48.853409, "lng":2.348800}}`)
 * **insideBoundingBox**: search entries inside a given area defined by the two extreme points of a rectangle (defined by 4 floats: p1Lat,p1Lng,p2Lat,p2Lng).<br/>For example `insideBoundingBox=47.3165,4.9665,47.3424,5.0201`).<br/>At indexing, you should specify geoloc of an object with the _geoloc attribute (in the form `{"_geoloc":{"lat":48.853409, "lng":2.348800}}`)
 * **queryType**: select how the query words are interpreted:
  * **prefixAll**: all query words are interpreted as prefixes (default behavior).
  * **prefixLast**: only the last word is interpreted as a prefix. This option is recommended if you have a lot of content to speedup the processing.
  * **prefixNone**: no query word is interpreted as a prefix. This option is not recommended.
 * **tags**: filter the query by a set of tags. You can AND tags by separating them by commas. To OR tags, you must add parentheses. For example, `tags=tag1,(tag2,tag3)` means *tag1 AND (tag2 OR tag3)*.<br/>At indexing, tags should be added in the _tags attribute of objects (for example `{"_tags":["tag1","tag2"]}` )

```ruby
index = Algolia::Index.new("cities")
res = index.search("query string")
res = index.search("query string", { "attributes" => "population,name", "hitsPerPage" => 20})
```

The server response will look like:

```javascript
{
    "hits":[
            { "name": "Betty Jane Mccamey",
              "company": "Vita Foods Inc.",
              "email": "betty@mccamey.com",
              "objectID": "6891Y2usk0",
              "_highlightResult": {"name": {"value": "Betty <em>Jan</em>e Mccamey", "matchLevel": "full"}, 
                                   "company": {"value": "Vita Foods Inc.", "matchLevel": "none"},
                                   "email": {"value": "betty@mccamey.com", "matchLevel": "none"} }
            },
            { "name": "Gayla Geimer Dan", 
              "company": "Ortman Mccain Co", 
              "email": "gayla@geimer.com", 
              "objectID": "ap78784310" 
              "_highlightResult": {"name": {"value": "Gayla Geimer <em>Dan</em>", "matchLevel": "full" },
                                   "company": {"value": "Ortman Mccain Co", "matchLevel": "none" },
                                   "email": {"highlighted": "gayla@geimer.com", "matchLevel": "none" } }
            }],
    "page":0,
    "nbHits":2,
    "nbPages":1,
    "hitsPerPage:":20,
    "processingTimeMS":1,
    "query":"jan"
}
```

Add a new object in the Index
-------------

Each entry in an index has a unique identifier called `objectID`. You have two ways to add en entry in the index:

 1. Using automatic `objectID` assignement, you will be able to retrieve it in the answer.
 2. Passing your own `objectID`

You don't need to explicitely create an index, it will be automatically created the first time you add an object.
Objects are schema less, you don't need any configuration to start indexing. The settings section provide details about advanced settings.

Example with automatic `objectID` assignement:

```ruby
res = index.add_object({"name" => "San Francisco", 
                        "population" => 805235})
puts "ObjectID=" + res["objectID"]
```

Example with manual `objectID` assignement:

```ruby
res = index.add_object({"name" => "San Francisco", 
                        "population" => 805235}, "myID")
puts "ObjectID=" + res["objectID"]
```

Update an existing object in the Index
-------------

You have two options to update an existing object:

 1. Replace all its attributes.
 2. Replace only some attributes.

Example to replace all the content of an existing object:

```ruby
index.save_object({"name" => "San Francisco", 
                   "population" => 805235, 
                   "objectID" => "myID"})
```

Example to update only the population attribute of an existing object:

```ruby
index.partial_update_object({"population" => 805235, 
                             "objectID" => "myID"})
```

Get an object
-------------

You can easily retrieve an object using its `objectID` and optionnaly a list of attributes you want to retrieve (using comma as separator):

```ruby
# Retrieves all attributes
index.get_object("myID")
# Retrieves name and population attributes
res = index.get_object("myID", "name,population")
# Retrieves only the name attribute
res = index.get_object("myID", "name")
```

Delete an object
-------------

You can delete an object using its `objectID`:

```ruby
index.delete_object("myID")
```

Index Settings
-------------

You can retrieve all settings using the `getSettings` function. The result will contains the following attributes:

 * **minWordSizeForApprox1**: (integer) the minimum number of characters to accept one typo (default = 3).
 * **minWordSizeForApprox2**: (integer) the minimum number of characters to accept two typos (default = 7).
 * **hitsPerPage**: (integer) the number of hits per page (default = 10).
 * **attributesToRetrieve**: (array of strings) default list of attributes to retrieve in objects.
 * **attributesToHighlight**: (array of strings) default list of attributes to highlight.
 * **attributesToSnippet**: (array of strings) default list of attributes to snippet alongside the number of words to return (syntax is 'attributeName:nbWords')<br/>By default no snippet is computed.
 * **attributesToIndex**: (array of strings) the list of fields you want to index.<br/>By default all textual attributes of your objects are indexed, but you should update it to get optimal results.<br/>This parameter has two important uses:
  * *Limits the attributes to index*.<br/>For example if you store a binary image in base64, you want to store it and be able to retrieve it but you don't want to search in the base64 string.
  * *Controls part of the ranking*.<br/>Matches in attributes at the beginning of the list will be considered more important than matches in attributes further down the list. 
 * **ranking**: (array of strings) controls the way hits are sorted.<br/>We have five available criteria:
  * **typo**: sort according to number of typos,
  * **geo**: sort according to decreasing distance when performing a geo-location based search,
  * **proximity**: sort according to the proximity of query words in hits, 
  * **attribute**: sort according to the order of attributes defined by **attributesToIndex**,
  * **custom**: sort according to a user defined formula set in **customRanking** attribute.
  <br/>The default order is `["typo", "geo", "proximity", "attribute", "custom"]`. We strongly recommend to keep this configuration.
 * **customRanking**: (array of strings) lets you specify part of the ranking.<br/>The syntax of this condition is an array of strings containing attributes prefixed by asc (ascending order) or desc (descending order) operator.
 For example `"customRanking" => ["desc(population)", "asc(name)"]`
 * **queryType**: select how the query words are interpreted:
  * **prefixAll**: all query words are interpreted as prefixes (default behavior).
  * **prefixLast**: only the last word is interpreted as a prefix. This option is recommended if you have a lot of content to speedup the processing.
  * **prefixNone**: no query word is interpreted as a prefix. This option is not recommended.

You can easily retrieve settings or update them:

```ruby
res = index.get_settings
puts settings.to_json
```

```ruby
index.set_settings({"customRanking" => ["desc(population)", "asc(name)"]})
```

List indexes
-------------
You can list all your indexes with their associated information (number of entries, disk size, etc.) with the `list_indexes` method:

```ruby
Algolia.list_indexes
```

Delete an index
-------------
You can delete an index using its name:

```ruby
index = Algolia::Index.new("cities")
index.delete
```

Wait indexing
-------------

All write operations return a `taskID` when the job is securely stored on our infrastructure but not when the job is published in your index. Even if it's extremely fast, you can easily ensure indexing is complete using the same method with a `!`.

For example, to wait for indexing of a new object:
```ruby
res = index.add_object!({"name" => "San Francisco", 
                         "population" => 805235})
```

If you want to ensure multiple objects have been indexed, you can only check the biggest taskID.

Batch writes
-------------

You may want to perform multiple operations with one API call to reduce latency.
We expose two methods to perform batch:
 * `addObjects`: add an array of object using automatic `objectID` assignement
 * `saveObjects`: add or update an array of object that contains an `objectID` attribute

Example using automatic `objectID` assignement:
```ruby
res = index.add_objects([{"name" => "San Francisco", 
                          "population" => 805235},
                         {"name" => "Los Angeles",
                          "population" => 3792621}])
```

Example with user defined `objectID` (add or update):
```ruby
res = index.save_objects([{"name" => "San Francisco", 
                           "population" => 805235,
                           "objectID" => "SFO"},
                          {"name" => "Los Angeles",
                           "population" => 3792621,
                           "objectID" => "LA"}])
```

Security / User API Keys
-------------

The admin API key provides full control of all your indexes. 
You can also generate user API keys to control security. 
These API keys can be restricted to a set of operations or/and restricted to a given index.

To list existing keys, you can use `list_user_keys` method:
```ruby
# Lists global API Keys
Algolia.list_user_keys
# Lists API Keys that can access only to this index
index.list_user_keys
```

Each key is defined by a set of rights that specify the authorized actions. The different rights are:
 * **search**: allows to search,
 * **addObject**: allows to add/update an object in the index,
 * **deleteObject**: allows to delete an existing object,
 * **deleteIndex**: allows to delete index content,
 * **settings**: allows to get index settings,
 * **editSettings**: allows to change index settings.

Example of API Key creation:
```ruby
# Creates a new global API key that can only perform search actions
res = Algolia.add_user_key(["search"])
puts res['key']
# Creates a new API key that can only perform search action on this index
res = index.add_user_key(["search"])
puts res['key']
```
You can also create a temporary API key that will be valid only for a specific period of time (in seconds):
```ruby
# Creates a new global API key that is valid for 300 seconds
res = Algolia.add_user_key(["search"], 300)
puts res['key']
# Creates a new index specific API key valid for 300 seconds
res = index.add_user_key(["search"], 300)
puts res['key']
```

Get the rights of a given key:
```ruby
# Gets the rights of a global key
Algolia.get_user_key("f420238212c54dcfad07ea0aa6d5c45f")
# Gets the rights of an index specific key
index.get_user_key("71671c38001bf3ac857bc82052485107")
```

Delete an existing key:
```ruby
# Deletes a global key
Algolia.delete_user_key("f420238212c54dcfad07ea0aa6d5c45f")
# Deletes an index specific key
index.delete_user_key("71671c38001bf3ac857bc82052485107")
```
