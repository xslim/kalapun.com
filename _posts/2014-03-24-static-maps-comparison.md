---
title: Static Maps comparison
tags: 
  - maps
published: true
---



Her's my personal comparison on different static maps apis. As Example, we want to show a road map image for Enkhuizen (lat: 52.70468296296834, lon: 5.300731658935547), with a zoom 13, using a free account and having nice map style.

**UPDATE:** Her's a nice website for this: http://staticmapmaker.com

## Google Maps

![Google Maps](http://maps.googleapis.com/maps/api/staticmap?center=52.70468296296834,5.300731658935547&zoom=13&size=640x200&sensor=false)

- **Pros:** Google-style maps
- **Cons:** Max size limit (640x640)
- **Limitations:** 25 000 views per day

## Mapbox

![Mapbox Map](http://api.tiles.mapbox.com/v3/xslim.hgm2p8g2/5.300731658935547,52.70468296296834,13/640x200.png)

Uses nicely-styled OpenStreet maps

- **Pros:** Very clean API
- **Cons:** Reversed lat & lon in request
- **Limitations:** 3000 map views per month (???)

## Nokia Here Maps

![Nokia Here Map](http://m.nok.it/?w=640&h=200&ml=eng&nord&nodot&pip&c=52.70468296296834,5.300731658935547&z=13&f=0)

~~Supports 2 endpoints - short Nokia `m.nok.it` (probably deprecated), and a more modern Here `image.maps.api.here.com/mia/1.6/mapview` which require api keys.~~

- **Pros:** Has a "Map-in-map view"
- **Cons:** Not common for people
- **Limitations:** 2500 views per day

Any better solutions?
