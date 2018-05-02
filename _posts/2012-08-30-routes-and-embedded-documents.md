---
title: "Rails routes and mongoid embedded documents"
tags: [ruby,rails,mongoid,embedded]
---

Following my previous post [Mongoid custom relation between embedded documents](http://kalapun.com/posts/mongoid-custom-relation-between-embedded-documents/)

Interesting how to set-up Routes and MVC for embedded documents? I'll try to tell you how I did it.

As I mentioned in previous post, my main document `Event` looks like

``` ruby
class Event
  # cutted
  embeds_many :acts
  embeds_many :areas
end
```


## Routing

In `routes.rb`

``` ruby
  resources :events do
    resources :acts
    resources :areas
  end
```


Now we can access the `Acts` and `Areas` with urls like `/events/:event_id/acts/`,  `/events/:event_id/acts/:id`, and same for `areas`

To see the all the routes you can use command `rake routes` in terminal

## Controllers

In all controllers for embedded documents, we need to have access to parent document. We can achieve this by adding `before_filter` to our controllers.

``` ruby
class ActsController < ApplicationController
  before_filter :check_event!

    def check_event!
    @event = Event.find(params[:event_id]) rescue nil

    if !@event
      redirect_to root_path, :alert => "Event not found!"
    end
  end
```


And also we need to change the actions to use our parent document

``` ruby
  def index
    @acts = @event.acts.all
    # Respond code
  end

  def show
    @act = @event.acts.find(params[:id])
    # Respond code
  end
```


Also, we need to change the creation method, so the child document will be added to the parent

``` ruby
  def create
    @act = Act.new(params[:act])

    respond_to do |format|
      if @event.acts << @act
        @event.save
        format.html { redirect_to [@event, @act], notice: 'Act was successfully created.' }
        # Other format code
      else
        # Else code
      end
    end
  end
```


Notice the adding code `@event.acts << @act` and  the redirect code `redirect_to [@event, @act]`

## Views

For every `link_to` in your view, you need to add relation to `Event` document, and also some paths needs changing.

``` ruby
= link_to 'Show', [@event, @act]
= link_to 'Edit', edit_event_act_path(@event, act)
= link_to 'New Act', new_event_act_path
```


For forms, I'm using `simple_form` gem, but basicaly the only thing you need to change is to put the `@event` there

``` ruby
= simple_form_for([@event, @act], html:{class:'form-horizontal'}) do |f|
  # inputs go here
```


I hope this short post will help you getting started...
