---
published: true
title: "Hosting your Web App for free - Overview"
---

There are numerous cloud platforms where you can host your Web App for free or almost free. Here I will give an overview of some hosting solutions: Heroku, Amazon AWS, OpenShift, AppFog, DigitalOcean.

## Platforms
- [Heroku](https://www.heroku.com/pricing) - 1 free instance per app. Probably unlimited apps. Instance is limited to one port
- [OpenShift](https://www.openshift.com/products/pricing/plan-comparison) - 3 free small instances per account. Instance is limited but not as much as Heroku.
- [AppFog](https://www.appfog.com/pricing/) - No free plans, plans start from 20$/mo
- [Amazon AWS](http://aws.amazon.com/ec2/pricing/) - Free for 1 year, Instance is a VM
- [DigitalOcean](https://www.digitalocean.com/pricing/) - Virtual machines

## Additional Instance pricing
- 1x / 2x is amount of CPU
- 512MB is amount of RAM
- If not noted, price is per hour

| Platform | 1x512MB | 2x512MB | 1x1GB |2x1GB  | 4x2GB |
|----------|---------|---------|-------|-------|-------|
|Heroku    | 0.05$   |  -      |   -   | 0.10$ | -     |
|OpenShift | 0.02$   | 0.025$  |   -   | 0.05$ | 0.10$ |
|AWS       | -       | -       | 0.013$| -     | -     |
|DO /h     | 0.007$  | -       | 0.015$| -     |0.06$  |
|DO /mo    | 5$/mo   | -       | 10$/mo| -     |40$/mo |

Bigger plans:

| Platform | 2x8GB | 4x8GB |
|----------|-------|-------|
|AWS       | 0.14$ | 0.232$|
|DO /h     | -     | 0.119$|
|DO /mo    | -     | 80$/mo| 


## Other
| Platform | Storage | Traffic | SSL |
|----------|---------|---------|-----|
|Heroku    |as addon | ?       |- |
|OpenShift | 1GB     | ?       |+ |
|AWS       |as addon | ?       |+ |
|DO        | 20GB    | 1TB     |+ |

## Deployment
- Heroku - super easy - `git push heroku master`
- OpenShift - more choises, needs configuration
- Amazon AWS - DIY
- DigitalOcean - DIY