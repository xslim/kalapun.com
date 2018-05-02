---
title: "Mongoid custom relation between embedded documents"
tags: [ruby,rails,mongoid,embedded]
---

Imagine we have a Mongoid document, where we have two embedded documents. And now we want to make some relation between them. What you can think of at first is to use

``` ruby
has_many :acts, :foreign_key => 'event_id'
```


But that won't work. Mongoid is not capable at this moment to support this between embedded documents :(

But we can try to implement our own, custom relationship.

WARNING: This post is only my attempt to make it happen. It can contain bugs, or there can be other ways how to make it work easier. If you found some - let me know.



## Overview

So let's say we have a main document called `Event`.

``` ruby
class Event
  include Mongoid::Document
  field :name, type: String
  embeds_many :acts
  embeds_many :areas
end
```


As you see, it embeds documents `Act` and `Area`.

``` ruby
class Act
  include Mongoid::Document
  include Mongoid::MultiParameterAttributes

  field :title, type: String
  field :start_at, type: DateTime
  field :end_at, type: DateTime

  default_scope asc(:start_at)

  embedded_in :event
```


and

``` ruby
class Area
  include Mongoid::Document

  field :name, type: String

  embedded_in :event
```


## Ralationship

Let's add some custom relationship:

In `Act` we want to relate to one `Area`, so we need an `field :area_id, type: String` to track that. In `Area` document we will need to track array of `acts`, so we'll add `field :act_ids, type: Array`

Now, let's add some custom relationship management to `Act` document

``` ruby
  def area
    self.event.areas.find(self.area_id) rescue nil
  end

  def area_name
    self.area.name rescue nil
  end

  def area_id=(value)

    act_id = self.id.to_s

    area = self.area
    if area.nil?
      area = find_area_by_id(value)
    end

    return super(value) if area.nil?

    if value.blank?
      area.pull(:act_ids, act_id) # remove old one
    else
      area.pull(:act_ids, act_id)
      area = find_area_by_id(value)
      area.add_to_set(:act_ids, act_id)
    end

    super(value)
  end

  def find_area_by_id(value)
    self.event.areas.find(value) rescue nil
  end

  #around_destroy
  before_destroy do |document|
    #Handle callback here.
    area = self.area
    area.pull(:act_ids, self.id.to_s) if area
  end
```


And for `Area`

``` ruby
  def act_ids=(values)
    values.delete('')

    old_values = self.act_ids || []
    old_values = old_values - values
    old_values.each do |act_id|
      act = find_act_with_id(act_id)
      act.unset(:area_id) if act
    end

    new_values = (values - old_values) || []

    new_values.each do |act_id|
      act = find_act_with_id(act_id)
      if act
        if !act.area_id.blank?
          self.event.areas.find(act.area_id).pull(:act_ids, act_id)
        end
        act.set(:area_id, self.id.to_s)
      end
    end
    super(values)
  end

  def find_act_with_id(value)
    self.event.acts.find(value) rescue nil
  end

  def acts
    self.event.acts.find(self.act_ids || []) rescue []
  end

  def acts=(value)
    self.act_ids = value.map(&:id)
  end

  before_destroy do |document|
    #Handle callback here.
    self.act_ids.each do |act_id|
      act = find_act_with_id(act_id)
      act.unset(:area_id) if act
    end
  end
```


Again, this post is just my attempt to make it work. If you have any comments how to improve this - let me know
