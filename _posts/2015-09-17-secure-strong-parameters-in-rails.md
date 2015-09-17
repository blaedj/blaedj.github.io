---
layout: post
title:  "Which Strong parameters can I securely permit in rails?"
date:   2015-09-17
tags: ruby-on-rails security software-development programming
---

Assume the following setup in a typical rails application: 

{% highlight ruby linenos%}
# work_order.rb
class WorkOrder <
ActiveRecord::Base
  has_many :line_items

  accepts_nested_attributes_for :line_items

end

#line_item.rb
class LineItem < ActiveRecord::Base
  belongs_to :work_order
end


# work_orders_controller.rb
class WorkOrdersController < ApplicationController
  # .... standard code here .....

  private
  def work_order_params
  params.require(:work_order)
    .permit(date, #.. various Work Order properties
            line_items_attributes: [
            # various attrs here, i.e. qty, cost
            :work_order_id
            ])
  end
end
{% endhighlight %}

Note the inclusion of `work_order_id` in the line_item_attributes array. All
a potential attacker has to do is insert a hidden html input tag into the edit form of a
work_order similar to this: 

    <input type="hidden" 
		name="work_order[labor_items_attributes][0][work_order_id]" value="509">

When they submit the form, the labor item will then be assigned to the work
order with id '509'. Whoops. 

The kicker to all this is that you don't actually need to include the 'work_order_id'
for everything to work correctly, Rails knows[^1] that the line item you are creating
is for 'this' work order. 

## So What?
This may not seem like a big deal in this context, and in truth it's probably
not. However, substitute a `User` for `WorkOrder`, and `AuthorizedFriend` for
`LineItem`, and imagine that an `AuthorizedFriend` has special privileges
with `User`s content, where that content is pictures, payment methods
etc. See the problem?

## Moral of the story
 Be careful what parameters you are allowing! Even though rails 'does' a lot
 of security for you, that doesn't mean you can just forget about security
 altogether when designing and developing software applications.


[^1]: How does rails know this? Good question! stay tuned for a blog post on this very subject...
