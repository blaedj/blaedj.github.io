---
layout: post
title:  "ActiveRecord validations as a developer safety net"
date:   2022-12-06
tags: ruby-on-rails 
---


Here is an idea I had - using rails ActiveRecord validations to ensure that the
proper methods are used to update state.

Part of the allure of active record callbacks (`after_update_commit`,
`before_destroy` etc) is that you don't need to worry about remembering to run
them.

For example, if updating a `Comment` should kick off a job to notify the
`Author` of a `BlogPost` via email, well, just use something like:

```ruby 
class Comment < ApplicationRecord

  # ... 

 def notify_blog.author_of_new_verified_comment
    return unless poster.verified?

    NotifyAuthorOfCommentJob.perform_async(self.id, blog.author.id)
 end
 after_save :notify_author_of_new_verified_comment
end
```

Then in any controller, we don't need to worry about making sure we remember to
enqueue the notify job, but only if the comment poster is a verified account
etc.

This becomes a more comfortable crutch to lean upon as the number of
contributers (or just the sheer codebase size) increases. A proliferation of
callbacks can (not always, but can) lead to pain - you can spend a lot of time
trying to figure out complex happy-paths that end up being reliant on several
callbacks chained together.

I've long liked the idea of using specific objects and/or methods to encapsulate
behavior that needs to happen when changing some state. Things like a
`BlogPost#publish!` method that not only updates the post's `status` to
`published`, but also increases the authors `post_count`, and kicks off a job to
rebuild some post cache somewhere. Maybe it even sends off some emails.

This is often easier to test, makes it clear where to add future behavior that
may be needed, and is conceptually clear - if I want a set of things to happen
when a post is published, that is the responsibility of the 'Publisher'.

BUT... how do we make sure that the BlogPublisher is used consistently? Maybe we
add a shortcut action somewhere, and instead of remembering to call the
BlogPublisher, we only call `@post.update(status: :published)`. If we had an
ActiveRecord callback, we wouldn't need to worry about that, right?


Well, here is a crazy idea (It may also be terrible, I haven't used this pattern
in anger): Let's use AR validations to raise an error if something other than
the `Post.publish!` method is used to set the `status` column to `:published`

Here is an example: 


```ruby 
class Post < ApplicationRecord
  attr_reader :in_publish_flow
  class ImproperUpdateError < StandardError; end

  # ...

  validate do 
    unless @in_publish_flow || !will_save_change_to_attribute?(:status, :published)
      raise ImproperUpdateError, "use Post#publish! to publish a blog post" 
    end
  end
  
  def publish!
    @in_publish_flow = true
    update(status: :published)
    author.increment_post_count! 
    RebuildBlogPostCacheJob.perform_later(author.id)
  end
end
```

Notice that we're raising an error, instead of the usual 

`errors.add(:status, "...")` seen in validations. This is intentional, because
we want this to be caught in _development_, this isn't something we want to
bubble up to the users of our app[^1].

Also, we're using an attr_reader for in_publish_flow, and that value is only set
in the `publish!` method - this attr being set to `true` is the guard against us
just calling `update(..)` instead of using the proper method

In the end, if we really want/need to, we can still just call
`post.update_attribute(status: "published")` (this is ruby, after all, there is 
always a workaround), but at least at that point it becomes a conscious choice,
and we aren't doing this by accident.  


[^1]: I fully acknowledge that this in itself might be a smell
