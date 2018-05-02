---
title: Liquid tag management for Jekyll
redirect_from: "/blog/2014/01/27/liquid-tag-management-for-jekyll/"
tags:
  - jekyll
  - html
  - liquid
---

Hosting a Jekyll site on GitHub Pages is cool but a bit limited. You can't use ruby code. But you can use [Liquid](https://github.com/Shopify/liquid/wiki/Liquid-for-Designers) template tags.

So her's a few hacks how to make a tag management for your blog:

## Tag cloud
A solution to show a tag cloud that can be ound in the net, and modified to suit my needs:

``` html
{% raw %}
  <ul class="tags">
{% for tag in site.tags %}
    <li style="font-size: {{ tag | last | size | times: 100 | divided_by: site.tags.size | plus: 70  }}%">
        <a href="#{{ tag | first | slugize }}">
            {{ tag | first }}
        </a>
    </li>
{% endfor %}
  </ul>
{% endraw %}
```



And you might add some style to it:

``` css
  .tags {
    list-style: none;
    padding: 0;
    text-align: justify;
    font-size: 20px;
  }
  .tags li {
    display: inline-block;
    margin: 0 25px 25px 0;
  }
```

## Tag links

I'v created a file `post_tags.html` in `_includes`:

``` html
{% raw %}
<span class="categories">
  {% if post %}
    {% assign tags = post.tags %}
  {% else %}
    {% assign tags = page.tags %}
  {% endif %}
  {% for tag in tags %}
  <a href="/tags/#{{tag|slugize}}">{{tag}}</a>{% unless forloop.last %},{% endunless %}
  {% endfor %}
</span>
{% endraw %}
```

and you can include it at the end (or beginning) of your post:

``` html
{% raw %}
{% include post_tags.html %}
{% endraw %}
```

## Tag page

On the tag page we have a tag cloud and a list of articles:

``` html
{% raw %}
---
layout: page
title: Tags
footer: false
permalink: /tags/
---

<--
PUT THE TAG CLOUD HERE
-->

<div id="blog-archives">
{% for tag in site.tags %}
  {% capture tag_name %}{{ tag | first }}{% endcapture %}
  <h2 id="#{{ tag_name | slugize }}">{{ tag_name }}</h2>
  <a name="{{ tag_name | slugize }}"></a>
  {% for post in site.tags[tag_name] %}
  <article>
    <h3><a href="{{ root_url }}{{ post.url }}">{{post.title}}</a></h3>
  </article>
  {% endfor %}
{% endfor %}
</div>
{% endraw %}
```

The result you can check here: [http://kalapun.com/tags/](http://kalapun.com/tags/)
