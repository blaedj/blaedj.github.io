---
layout: post
title: "Failing Tests Considered Helpful"
categories: programming
---

  _Background: I work in web development, and am trying to learn and apply the concepts of TDD._

  I have been working on an MVC web application, and was recently creating a new model. This model inherits from an existing ORM-generated class (a questionable practice, but important to our story). As I needed a method to get all the data, let's call it 'getAllDataPoints', with one parameter. Pretty standard naming convention right?

  Anyhow, I stubbed out the basic logic for the method, a rather simple database query. I wrote a unit test for the method, which expected a return value of not null. Run tests.... awesome, green! I figured I should make sure we are getting many data points, so I modified the test to check that a reasonable number of records were returned. The test passed on the first run again, great!

  A nagging thought told me that I should probably make the test fail, everything was seeming too easy. I commented out the return on the method, changed it to return only the first 3 records, and ran the tests again. All green. Swee-... Wait, that should've failed! I commented out the method, ran the tests, still all green. Hmmm, maybe my naming was _too_ standard... I checked the base class, and sure enough, there was a method with the same name, taking no parameter, and in my tests I was calling that method. I changed the method call, ran the tests, and corrected the situation.

## Moral of the story: ##

  __Start with a failing test!__ Testing doesn't help much if you are testing the wrong thing! Taking shortcuts, like stubbing out the method first, aren't necessarily time savers. Sure, in this case I only wasted about 10 minutes max of my time, but it could easily have been an hour or two in a more complicated situation. This is a lesson I have read in several places, but learning from mistakes seems to make things sink in better...

PS If you wondered why I was re-creating an existing method that seemingly did what I needed, this happened in the course of creating a method that manipulated the data when it was finished.
