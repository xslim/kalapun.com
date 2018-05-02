---
title: "Generating API - teaser"
published: false
---

![](/images/2013-03-01/seq_scruffy.png)

Imagine you'r defining how Web-service API should look like in a simple YAML file



``` yaml
baseURL: "http://localhost"
fixtureDirectory: ""

defaults:
  request:
    method: "GET"
  response:
    responseCode: 200
    headers:
      Content-Type: application/json

mocks:
  -
    request:
      url: "/api/v1/catalog/products"
    response:
      fixture: "catalog_products.json"
      timeout: 60.0
  -
    request:
      url: "/api/v1/pickups"
    response:
      fixture: "pickups.json"
```


Now imagine you make a stubs from it, in runtime, on iOS ans Android.. and Ruby...

Or you generate API documentation...

Or just a few diagrams, to show your boss how it works...

``` sh
 ./parser.rb api.yml > seq.diag
seqdiag -Tsvg seq.diag > seq.svg
./svg2png.sh seq.svg
```


And you get a Sequence diagram!

![](/images/2013-03-01/seq_def.png)

And after a few additional tricks...

``` sh
 ./svg2scruffy.rb seq.svg > seq_scruffy.svg
```


You can get a fun "hand-drawn" diagram

![](/images/2013-03-01/seq_scruffy.png)

Stay tuned...
