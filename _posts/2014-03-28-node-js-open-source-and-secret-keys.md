---
title: "Node.js, Open source and Secret Keys"
tags:
  - nodejs
  - config
  - env
---

So you want to use of 3-rd party web services that require API keys, but also make your app available to opensource? Can be done. For example, if you'r using Heroku, you can put your secret keys as Environmental variables:

``` sh
$ heroku config:set KEY_MAPBOX=xxx.yyy
$ heroku config
=== my-app Config Vars
KEY_MAPBOX:    xxx.yyy
```

BTW, if you want to connect your folder to the app do this:

``` sh
heroku git:remote -a project
```

Now, you can use it in Node.js app with `key = process.env.KEY_MAPBOX`. But what about time when you develop your app? Setting it via bash's `export` is not fun, so to help with this, ther's a small npm module `node-env-file`. 

``` sh
npm install node-env-file --save
```

It uses `.env` file to pull the variables if they are not set:

``` js
var fs = require('fs'),
    env = require('node-env-file')
    
if (fs.existsSync(__dirname + '/.env' )) {
  env(__dirname + '/.env')
}

console.log("Secret: " + process.env.KEY_MAPBOX);
```



And the `.env` file looks like this:

``` sh
KEY_MAPBOX=xxx.yyy
```

That's better. Now how about synchronizing the `.env` with Heroku? Use  [ddollar/heroku-config](https://github.com/ddollar/heroku-config) to pull / push from `.env`:

``` sh
$ heroku plugins:install git://github.com/ddollar/heroku-config.git
$ heroku config:pull
```
