---
published: false
title: "Hubot, a personal assistance"
comments: true
description: null
tags: 
  - automation
  - hubot
---

## Installation
``` sh
npm install -g coffee-script
npm install -g hubot
```

Now create your assistance providing his name. Here, we'll refer to `hubot`

``` sh
hubot -c hubot
```

``` sh
cd hubot
HUBOT_AUTH_ADMIN="" REDISTOGO_URL="redis://pub-redis-XXX.com:12771" bin/hubot
```

Deploy to heroku:

``` sh
git init
git add --all
git commit -am "Initial commit"
heroku git:remote -a my-hubot-app
heroku config:set REDISTOGO_URL="redis://pub-redis-xxx.example.com:12771"
heroku config:set HEROKU_URL="http://my-hubot-app.herokuapp.com"
```

