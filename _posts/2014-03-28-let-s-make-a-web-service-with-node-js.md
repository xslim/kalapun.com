---
title: "Let's make a Web-service with Node.js"
tags:
  - nodejs
  - javascript
  - coffeescript
  - heroku
  - proxy
---

I'll make a short walk thru how I've built a simple Web-service (proxy) with Node.js that might be usefull for developers who are started playing with it.

The source code for the app is available on [xslim/img.mrt.io](https://github.com/xslim/img.mrt.io)

## Hello world? No, dump Readme

We'll use [Express.js](http://expressjs.com) for our routes. Check that your `package.json` contains `express` as a dependency in `dependencies` section and install them with `npm install`. Create `README.md` file with some content. Next,  create an `app.js` and fill with usefull code:

``` js
var express = require('express'),
    app = express(),
    http = require('http'),
    fs = require('fs');

app.get("/", function(req, res, next) {
  fs.readFile('README.md', function(err, data) {
    res.writeHead(200, {"Content-Type": "text/plain"});
    res.end(data);
  });
});

app.listen(process.env.PORT || 3000);
```

Now after running `node app.js` and visiting the `http://localhost:3000/` will give you a dump of your `README.md` file ([Real example](http://img.mrt.io/)). Cool, huh?



## Coffeescript

I prefer having a simpler and cleaner code. Thus, I use [Coffee script](http://coffeescript.org). To use it, add `coffee-script` `npm` dependency. The filenames should be changed from `.js` to `.coffee`, and the code should be changed as well. You can use [js2coffee.org](http://js2coffee.org) for converting the code. So our `app.coffee` will be changed to this:

``` coffee
express = require('express')
app = express()
http = require('http')
fs = require('fs')

app.get "/", (req, res, next) ->
  fs.readFile 'README.md', (err, data) ->
    res.writeHead 200, "Content-Type": "text/plain"
    res.end data

app.listen process.env.PORT or 3000
```

To run it, use `coffee app.coffee`.

## Watch it!

It might be quite boring to restart the Node every time you make a change to a file. We can automate this using [Grunt](http://gruntjs.com) and [nodemon](http://nodemon.io).

Create a section `devDependencies` in `package.json` and add `grunt`, `grunt-contrib-watch`, `load-grunt-tasks` and `grunt-nodemon`. Create `Gruntfile.js` with contents:

``` js
module.exports = function (grunt) {
    require('load-grunt-tasks')(grunt);
    grunt.initConfig({
        nodemon: {
          dev: {
            script: 'app.coffee'
          }
        }
      });
};
```

Now the only thing you need to run is `grunt nodemon` and enjoy. For a more crazy automation, ther's an awesome blog post [Continuous Development in Node.js](http://blog.ponyfoo.com/2013/09/26/continuous-development-in-nodejs).

## Let's make a proxy

Say we want to show our avatar calling `/avatar.jpg` by streaming the actual data from Gravatar. Can be done!

First, create a `proxy` method

``` coffee
proxyTo = (url, response) ->
  try
    http.get url, (res) ->
      res.pipe response

  catch _error
    response.writeHead 500, "Content-Type": "text/html"
    response.end url, _error.message
```

And the endpoint:

``` coffee
app.get "/avatar.jpg", (req, res) ->
  proxyTo "http://gravatar.com/avatar/4374a44a5a6642a24ac2975b9aa2dfe7", res
```

[Real example](http://img.mrt.io/slim.jpg)

## More advanced proxy

How do we deal with a variables? Let's make a proxy for google static maps. As opposed to Google API which is not that clean, we want a simple clean one. For example, we want `/map/{lat},{lon},{zoom}/{size}`:

``` coffee
app.get "/map/:lat,:lon,:zoom/:size", (req, res) ->
  lat = req.param("lat")
  lon = req.param("lon")
  zoom = req.param("zoom")

  size = req.param("size")
  size = size.split("x")
  w = size[0]
  h = size[1]

  url = "http://maps.googleapis.com"
  url += "/maps/api/staticmap?center=" + lat + "," + lon + "&zoom=" + zoom + "&size=" + w + "x" + h + "&sensor=false"

  proxyTo url, res
```

[Real example](http://img.mrt.io/map/52.70468296296834,5.300731658935547,13/640x200)


## Refactor & Extract

Going with a best practice of not having all the code in one file, let's refactor the map url generation in a separate "module". Create `maps.coffee` that `exports` the function:

``` coffee
exports.static_link = (lat, lon, zoom, size, provider) ->
  size = size.split("x")
  url = ""
  w = size[0]
  h = size[1]
  provider = "google" unless provider
  switch provider
    when "google"
      key = process.env.KEY_GOOGLE
      url = "http://maps.googleapis.com"
      url += "/maps/api/staticmap?center=" + lat + "," + lon + "&zoom=" + zoom + "&size=" + w + "x" + h + "&sensor=false"
      if (key.length > 0)
        url += "&key="+key
    when "here"
      url = "http://m.nok.it"
      url += "?w=" + w + "&h=" + h + "&ml=eng&nord&nodot&pip&c=" + lat + "," + lon + "&z=" + zoom + "&f=0"
  url
```

Use it in `app.coffee`:

``` coffee
maps = require('./maps')

app.get "/map/:lat,:lon,:zoom/:size", (req, res) ->
  url = maps.static_link(req.param("lat"), req.param("lon"), req.param("zoom"), req.param("size"), req.query.t)
  proxyTo url, res  
```

## Deploy to Heroku

Well, Heroku have a nice [write-up](https://devcenter.heroku.com/articles/getting-started-with-nodejs) on how to do this. The only thing to know is that we'r using Coffee script, so we need to change `Procfile` to this:

```
web: coffee app.coffee
```
