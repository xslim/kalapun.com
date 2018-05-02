---
title: GPX to GeoJSON in MongoDB with Node.js
tags:
  - mongodb
  - nodejs
  - geoJSON
  - mongoose
  - GPX
---

It's quite hard to find information how to work with [GeoJSON](http://geojson.org). And some information provided in the library [documentation](http://docs.mongodb.org/manual/core/2dsphere/) is quite hard to understand. So her's a few code blocks that can help someone combine [GPX](http://en.wikipedia.org/wiki/GPS_eXchange_Format), GeoJSON, MongoDB, Node.js and [Mongoose](http://mongoosejs.com).

_Disclamer: I'm new to Node.js and CoffeeScript, so feel free to comment my mistakes or code improvements._

## GPX to GeoJSON
We will use a Node.JS library [togeojson](https://github.com/mapbox/togeojson) for this.

``` sh
npm install togeojson --save
npm install jsdom --save
```

Imagine we will upload a GPX file and convert it after to GeoJSON

``` coffeescript
togeojson = require 'togeojson'
fs = require 'fs'
jsdom = require('jsdom').jsdom

# Later on

app.post '/routes/upload', (req, res) ->
  file = req.files.file
  if file
    console.log "Uploaded " + file.originalFilename + " to " + file.path

    fs.readFile file.path, (err, data) ->
        gpx = jsdom(data)
        converted = togeojson.gpx(gpx)

        # Send back converted
        res.send converted
  else
    res.send 400
```

## Storing GeoJSON in MongoDB


For connecting with our MongoDB from Node.js we will use `mongoose` library

``` sh
npm install mongoose --save
```

The GeoJSON has a definition of `Feature` and `FeatureCollection` [(Spec)](http://geojson.org/geojson-spec.html). For the simple case, we will store in our DB only the first `Feature` under the `loc`. The `Feature` itself can be a point, multi-line or a polygon. For most use-cases, working with one `Feature` is enought. Storing `FeatureCollection` will be covered in the separate post.

First, create a schema:

``` coffeescript
mongoose = require 'mongoose'
mongoose.connect(process.env.MONGOHQ_URL);

Schema = mongoose.Schema
routeSchema = new Schema
  name: String
  loc: {type: Object, index: '2dsphere'}
```

Now let's add a virtual setter and getter to convert from GeoJSON to our Mongo representation and vise versa:

``` coffeescript
routeSchema.virtual("geoJSON").get ->
  feature =
    type: "Feature"
    geometry: @loc
    properties:
      name: @name
  resp =
    type: "FeatureCollection"
    features: [
      feature
    ]
  return resp

routeSchema.virtual("geoJSON").set (json) ->
  feature = json.features[0]
  @name = feature.properties.name
  @loc = feature.geometry
```

Now let's change our `/upload` route to add saving:

``` coffeescript
app.post '/routes/upload', (req, res) ->
  file = req.files.file
  if file
    fs.readFile file.path, (err, data) ->
      gpx = jsdom(data)
      converted = togeojson.gpx(gpx)

      fs.unlinkSync file.path

      route = new Route()
      route.geoJSON = converted
      route.save (err) ->
        console.log "Error saved route", err if err
        console.log "Saved route", route
        res.send err if err
        res.send route.geoJSON
   else
     res.send 400
```

## Checking the stored data
After uploading, we can check the data with `mongo` console:

``` sh
mongo -u admin -p password lennon.mongohq.com:10031/app12345
> version()
2.4.9
> db.routes.find()
{ "loc" : { "coordinates" : [ [5.40300237,53.177886249], [5.276818363, 53.223566907] ], "type" : "LineString" }, "name" : "Waddenrace", "_id" : ObjectId("5320bc0cae1f27a09faed03b"), "__v" : 0 }
```

So it's there. Now let's try searching with [$near](http://docs.mongodb.org/manual/reference/operator/query/near/):

``` sh
> db.routes.find({loc: {$near : { $geometry : { type: "Point",coordinates: [5.40300237, 53.177886249]}}, $maxDistance : 500 }})
{ "loc" : { "coordinates" : [ [5.40300237,53.177886249], [5.276818363, 53.223566907] ], "type" : "LineString" }, "name" : "Waddenrace", "_id" : ObjectId("5320bc0cae1f27a09faed03b"), "__v" : 0 }
```

In real life, I suggest using [geoNear](http://docs.mongodb.org/manual/reference/command/geoNear/) command instead of `$near`. Will cover this in next article.
