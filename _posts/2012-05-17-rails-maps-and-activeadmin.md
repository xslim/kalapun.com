---
title: "Rails, Maps and ActiveAdmin"
redirect_from: "/blog/2012/05/17/rails-maps-and-activeadmin/"
tags: [ruby, rails, activeadmin, maps, gmap]
---

In this short post I'll tell how I made [Google-Maps-for-Rails](https://github.com/apneadiving/Google-Maps-for-Rails) be able to work with [ActiveAdmin](http://activeadmin.info).

Let's asume you have a working configured Rails 3.2, MongoDB and ActiveAdmin.

First you need to install gem - `gem 'gmaps4rails'`, then run `rails g gmaps4rails:install`



Let's add maps for our model object `Shop`.

``` ruby
class Shop
  include Mongoid::Document
  include Mongoid::Timestamps
  include Mongoid::Spacial::Document

  field :location, :type => Array, :spacial => {:lat => :lat, :lng => :lng, :return_array => true }
  index [[:location, Mongo::GEO2D]], :min => -180, :max => 180, :bits => 24, :background => true
end
```


We need to change it a bit and add this things:

``` ruby
  include Gmaps4rails::ActsAsGmappable
  acts_as_gmappable :process_geocoding => false

  def latitude
    location[1]
  end

  def longitude
    location[0]
  end
```


Now let's edit our `app/admin/shops.rb`

``` ruby
    show :title => proc{ "#{shop.business.name if shop.business} / #{shop.name}" } do
    attributes_table do
      row :name
      row(:location) {|o| gmaps("markers" => {data: o.to_gmaps4rails}, "map_options" =>  { auto_zoom: false, zoom: 15 }) }
      row :address
      row :description
      row :created_at
      row :updated_at
    end
  end
```


If we will try to test it - we won't see a map, because ActiveAdmin is not auto loading our scripts and styles. So let's try fix that.

First, we need to add gmaps4rails css & javascript files for loading.

`app/assets/stylesheets/active_admin.css.scss`

``` css
@import "gmaps4rails";
```


`app/assets/javascripts/active_admin.js`

``` javascript
//= require gmaps4rails/gmaps4rails.base
//= require gmaps4rails/gmaps4rails.googlemaps
```


Now, we need to make custom view for out maps, that will also load google-map javascript. Create file `app/views/gmaps4rails/_gmaps4rails.html.erb` With contents:

``` html
<script type="text/javascript" src='http://maps.google.com/maps/api/js?sensor=true'></script>
<script type="text/javascript" src='http://google-maps-utility-library-v3.googlecode.com/svn/tags/markerclusterer/1.0/src/markerclusterer_compiled.js'></script>

 <% case dom.map_provider %>
<% when "mapquest" %>
<div id="<%= dom.map_id %>" style="width:750px; height:475px;">

</div>
<% when "bing" %>
  <div id="<%= dom.map_id %>" class="<%= dom.map_class %>"></div>
<% else %>
<div class="<%= dom.container_class %>">
  <div id="<%= dom.map_id %>" class="<%= dom.map_class %>"></div>
</div>
<% end %>


<script type="text/javascript" charset="utf-8">
 <%=raw options.to_gmaps4rails %>
</script>
```


Now if we test showing our model in ActiveAdmin, we will see a nice map. But let's add few more things. Edit your `Shop` model again and add this code:

``` ruby
  def gmaps4rails_title
    self.name
  end

  def gmaps4rails_infowindow
    "<b>#{self.name}</b><br /><i>#{self.description}</i><br /><br />#{self.address}<br /><i>#{self.location.join(', ')}</i>"
  end
```


Now our markers will be clickable and will show nice info window.
