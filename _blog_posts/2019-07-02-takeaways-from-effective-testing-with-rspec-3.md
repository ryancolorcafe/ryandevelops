---
layout: post
title:  Takeaways from "Effective Testing with RSpec 3"
date:   2019-07-02 05:35:00 -0800
tags:   ruby
---

{% if page.path contains "#excerpt" %}
  [<img src="../assets/images/effective-testing-with-rspec-3.jpg" alt="Walt Disney Music Hall">]({{site.url}}/collection/2019-07-02-takeaways-from-effective-testing-with-rspec-3)
{% else %}
  ![Walt Disney Music Hall](../assets/images/effective-testing-with-rspec-3.jpg)
{% endif %}


If you are looking to step up your RSpec knowledge, I definitely recommend ["Effective Testing with RSpec 3"](https://pragprog.com/book/rspec3/effective-testing-with-rspec-3) by Myron Marston and Ian Dees. In this post I'll sum up my main takeaways from the book, covering the motivations for testing, what makes for a good test, and describing the main types of specs you should focus on.

<!-- more -->

# Motivations for testing

The incentives for testing are made clear from the beginning. Consider this quote from Tom Stuart in the foreward:

<blockquote>"Using RSpec isn’t just a testing framework. It’s a tool for learning how to think critically, patiently, and systematically about the design of your code, and how to make software in a methodical way so you have confidence that it’s well organized, clear, and correct."</blockquote>

Marston and Dees elaborate on all the reasons why you should test. Good tests can make you more confident in your code, such as ensuring your happy path behaves as expected and that your code handles errors the correct way. They can help eliminate the fear of wondering if that new feature you want to add will accidentally break something in another part of the app.

Refactoring can also be easier with tests, since they will let you know if you've broken anything in the process. Writing tests can help guide the design of your code and encourages you to break things into small steps. The sustainability of your app grows as well, since it is much easier to add new features in a well-tested app rather than wrestling with code you're unsure about. As an added bonus, tests can help to serve as documentation for your code's behavior.

# What makes for a good test?

### Your tests aren't too brittle
A key factor of a good test according to the book is that they aren't too strict. We've all been burned by tests that seem to break so easily while only making a small change to the code. The authors provide an example:

<blockquote>"Rather than asserting that an error message exactly matches a particular string ('Could not find user with ID 123'), consider using substrings to match just the key parts ('Could not find user')."</blockquote>

Even small modifications like that can make for a much more pleasant coding experience, the authors claim. They also warn not to couple your tests too specifically to the elements in your UI, as the UI tends to change frequently and can cause more headaches down the road.

### The test is necessary
A good test is also one that is truly necessary. Over-testing can increase the chance of breaking other tests you thought were unconnected and make test suites take a really long time to run. They quote Kent Beck, who states:

<blockquote>"I get paid for code that works, not for tests, so my philosophy is to test as little as possible to reach a given level of confidence…"</blockquote>

Or as the authors themselves put it, they ask the following question before deciding whether or not to write a test:

<blockquote>"Does this test pay for the cost of writing and running it?"</blockquote>

### They are speedy and share code
Keeping tests snappy means you get quicker feedback, which keeps you in the zone longer and increases your focus. Good test suites also make use of shared code whenever possible, avoiding bloat and creating less setup headaches for specs that are similar. These usually come in the form of let definitions, hooks, and helper methods.

# Main spec types

Marston and Dees recommend focusing on just a few types of specs, specifically the ones below as described by Steve Freeman and Nat Pryce in their book ["Growing Object-Oriented Software, Guided by Tests"](https://www.amazon.com/Growing-Object-Oriented-Software-Guided-Tests/dp/0321503627):

* • Acceptance: does the whole system work?
* • Unit: Do our objects do the right thing, are they convenient to work with?
* • Integration: Does our code work against code we can’t change?

### Acceptance specs
Acceptance specs exercise a feature from beginning to end, which is great for testing that all the different parts involved are working together as they should. They're also often harder to write, are more brittle, and slower to run.

### Unit specs
Unit specs focus strictly on the unit of code you are testing. These tend to be very fast tests since they often don't need to interact with the rest of your app. They're also very useful in that they can help guide the design of your code. However, they will not be as valuable as when looking to do a larger refactor of your code, since they won't tell you much except for the very specific code it is testing.

### Integration specs
Integration specs are for external services, such as databases or third-party APIs. They can be tricky in that if you're writing something to a database, for example, you may need to rollback any changes made so as not to affect other tests. Since they are allowed to call third party code (even if indirect), they are often much slower to run.

### Preferred spec types
Unit specs are generally preferred as they are the fastest to run, easiest to write, and can be instrumental in designing your code. However, as mentioned above, they aren't as valuable in testing features end-to-end like an acceptance test. For acceptance tests, the authors recommend focusing on the happy path and keeping them spare otherwise. To reduce writing integration tests, their advice is:

<blockquote>"to keep [your] interfaces to external resources small and well defined. That way, [you] only have to write a few integration specs for these interfaces."</blockquote>

# What else is covered in the book?
You'll go through the exercise of creating a small app to get practice with creating each type of spec, learn about how metadata and RSpec's configurations can be very useful, be introduced to the endless amount of matchers and ways to set expectations, as well as deep diving into RSpec Mocks. Give the book a spin and let me know if you found it to be as helpful as I did!

<br/>
