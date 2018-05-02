# Jekyll Plugins on this site

The possible plugins on Github Pages can be seen here: https://pages.github.com/versions/

## Major

### jekyll-github-metadata
https://github.com/jekyll/github-metadata

* Propagates the `site.github` namespace with [repository metadata](https://help.github.com/articles/repository-metadata-on-github-pages/)
* Sets `site.title` as the repository name, if none is set
* Sets `site.description` as the repository tagline if none is set
* Sets `site.url` as the GitHub Pages domain (cname or user domain), if none is set
* Sets `site.baseurl` as the project name for project pages if none is set

### jekyll-default-layout
https://github.com/benbalter/jekyll-default-layout

* `/index.md` - the home layout, the page layout, or the default layout, if they exist, in that order
* A page - the page layout or the default layout, if they exist, in that order
* A post - the post layout or the default layout, if they exist, in that order

### jekyll-optional-front-matter

https://github.com/benbalter/jekyll-optional-front-matter

### jekyll-titles-from-headings
https://github.com/benbalter/jekyll-titles-from-headings

### jekyll-commonmark-ghpages
https://github.com/github/jekyll-commonmark-ghpages

* `markdown: CommonMarkGhPages`

### jekyll-seo-tag
https://github.com/jekyll/jekyll-seo-tag

* Add the following right before `</head>` in your site's template(s): `{  seo  }`

### jekyll-sitemap
https://github.com/jekyll/jekyll-feed

### jekyll-sitemap
https://github.com/jekyll/jekyll-feed


## Optional

### jekyll-readme-index
https://github.com/benbalter/jekyll-readme-index

```
readme_index:
  enabled:          true
  remove_originals: false
```

### jekyll-avatar
https://github.com/benbalter/jekyll-avatar

* `{  avatar [USERNAME]  }` or `{  avatar hubot size=50  }`
