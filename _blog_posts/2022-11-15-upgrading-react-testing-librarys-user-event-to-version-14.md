---
layout: post
title: Upgrading Testing Library's user-event to version 14
date: 2022-11-15 07:21:00 -0800
tags: react
---

{% if page.path contains "#excerpt" %}
[<img src="../assets/images/react-testing-library.png" alt="React Testing Library octopus logo, React logo, and Jest logo">]({{site.url}}/collection/2022-11-15-upgrading-react-testing-librarys-user-event-to-version-14)
{% else %}
![React Testing Library octopus logo, React logo, and Jest logo](../assets/images/react-testing-library.png)
{% endif %}

In this article I'll go over how to upgrade Testing Library's [user-event](https://testing-library.com/docs/user-event/intro) package to version 14. I'll give a brief refresher on why we use `user-event` in the first place, take a look at what's new in version 14, and finally go over the details of how I did the upgrade. I'll include a couple examples of tests that broke when upgrading that will hopefully help make your transition smoother.

<!-- more -->

# What is `user-event`?

As explained in the `user-event` docs:

<blockquote>"user-event is a companion library for Testing Library that simulates user interactions by dispatching the events that would happen if the interaction took place in a browser."</blockquote>

To sum up, this library helps to closely mimic how the user interacts with the browser and makes for better tests.

# Why should we use it?

Previously, [fireEvent](https://testing-library.com/docs/dom-testing-library/api-events/#fireevent) was the API for testing user interactions in the browser. However, `user-event` does it one better by trying to simulate _all_ the interactions that happen when a user takes an action in the browser. Kent C. Dodds compares `fireEvent.change` to `userEvent.type` in a [blog post](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library#not-using-testing-libraryuser-event):

<blockquote>"`fireEvent.change` will simply trigger a single change event on the input. However the type call will trigger `keyDown`, `keyPress`, and `keyUp` events for each character as well. It's much closer to the user's actual interactions."</blockquote>

Some of these interactions will be missed with `fireEvent`, and that could be problematic for your tests. As mentioned in the docs, there are still a few rare cases where `fireEvent` may be needed, but in most cases `user-event` should be used.

# What's new in `user-event` 14?

You can see most of the changes that come with version 14 on the [releases page](https://github.com/testing-library/user-event/releases/tag/v14.0.0). Besides lots of other smaller features and bug fixes, the biggest change is the [setup](https://testing-library.com/docs/user-event/setup) API, which allows you to configure options for each instance that the `user-event` library is used. More on this below.

Before you attempt an upgrade, be sure to take a good look at the "Breaking Changes" section of the releases page mentioned above to see how you might be affected. I'll provide an example of a breaking change after looking at how to start the upgrade.

# How to upgrade

Upgrade the package by running:

{% highlight shell %}
yarn upgrade @testing-library/user-event
{% endhighlight %}

or:

{% highlight shell %}
npm update @testing-library/user-event
{% endhighlight %}

The first thing I'd do after upgrading is to run your tests to get an idea of how much work you have ahead of you:

{% highlight shell %}
yarn test
{% endhighlight %}

or:

{% highlight shell %}
npm test
{% endhighlight %}

In my case, after upgrading I had about 18% of tests failing. Most tests should pass because calling `userEvent` directly calls the new `setup` API under the hood. This call to `setup` behind the scenes was included specifically to help make the transition to version 14 easier. I'll show you how we should use `setup` going forward.

To start however, it might be helpful to look at how we did things previously. Your tests might've looked like the following, where we import and call `userEvent` directly:

{% highlight javascript %}
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

test("clicking checkbox", () => {
  render(<ExampleComponent />);

  userEvent.click(screen.getByRole("checkbox"));

  expect(screen.getByTestId("example-id")).toBeInTheDocument();
});
{% endhighlight %}

Now that we have a reference of what we were doing before, let's get to how we should do things going forward. As I mentioned above, the `setup` API is likely what will cause the most amount of code changes when you upgrade. You'll want to create a test utility like the following code I put into a file I named `setup.ts`:

{% highlight ts %}
import { render } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { Options } from "@testing-library/user-event/dist/types/options";
import { ReactElement } from "react";

export const setup = (ui: ReactElement, options?: Options) => ({
  user: userEvent.setup(options),
  ...render(ui),
});
{% endhighlight %}

This utility accepts a React component and some options, instantiates a `user` with `userEvent.setup` and passes `options` as an argument to `setup`, and then uses `render` to render the component. With this test utility created, you can then import it and use it in your tests like so:

{% highlight javascript %}
import { screen } from "@testing-library/react";
// make sure to correct the import path for your app
import { setup } from "@/util/testUtils/setup";

test("clicking checkbox", async () => {
  const { user } = setup(<ExampleComponent />);

  await user.click(screen.getByRole("checkbox"));

  expect(screen.getByTestId("example-id")).toBeInTheDocument();
});
{% endhighlight %}

Notice a few differences from how we did things previously? Before we would import and call `userEvent` directly and call `render` with the component. Now we destructure `user` from `setup`, and call any `user-event` actions on `user`. Also notice we had to add an `async` in the test setup options, and are adding an `await` before `user.click`. Previously when calling `userEvent.click`, it would automatically wrap an `await` around the action. Not anymore. Now you have to be explicit about calling `await`. In my experience upgrading, although there may be a few rare cases where an `await` isn't needed, you can pretty much always assume you'll need it and default to using it.

# Configuring `setup` options

There are many `setup` [options](https://testing-library.com/docs/user-event/options) available for use. Let's take a look at an example of the options and how it affects our tests.

Again let's start with what we were doing before. Let's say you wanted to type into two different inputs, but skip the click event that would usually happen by default:

{% highlight javascript %}
render(<SomeComponent />);

userEvent.type(
  screen.getByRole("textbox", { name: "someTextbox" }),
  "something",
  { skipClick: true }
);

userEvent.type(
  screen.getByRole("textbox", { name: "anotherTextbox" }),
  "anything",
  { skipClick: true }
);
{% endhighlight %}

Now, using `setup` we'd do the following:

{% highlight javascript %}
const { user } = setup(<SomeComponent />, { skipClick: true });

await user.type(
  screen.getByRole("textbox", { name: "someTextbox" }),
  "something"
);

await user.type(
  screen.getByRole("textbox", { name: "anotherTextbox" }),
  "anything"
);
{% endhighlight %}

Instead of passing a `skipClick` option for every single event, `{ skipClick: true }` is applied to the entire instance of `setup`. This can be handy for tricky situations like needing to change the [keyboard layout](https://testing-library.com/docs/user-event/options#keyboardmap) for an entire test.

# How I fixed a test using a `setup` option

Remember how I said to take a close look at the "Breaking Changes" section on the releases page? I did not and struggled to figure out why one of our existing tests wasn't working.

I had a test where [upload](https://testing-library.com/docs/user-event/utility/#upload) was used to upload a CSV to an input, and we wanted to test that an error would occur when trying to upload a JSON file instead of a CSV file:

{% highlight javascript %}
const mockFile = new File([blob], "test.json", {
  type: "application/json",
});

const { user } = setup(<ImportQuoteRequests {...props} />);

await user.upload(screen.getByTestId("#CSVReader"), mockFile);

await waitFor(() => {
  expect(props.setErrors).toHaveBeenCalledWith([
    "The file you've chosen couldn't be recognized.",
  ]);
});
{% endhighlight %}

Before the upgrade this test was working, and the only thing that really had changed was the usage of `setup` ...so what gives? It turns out that one of the breaking changes listed was that the default for `upload` had changed from allowing any file type regardless of the input's `accept` property, to only allowing what was in the `accept` property by default. This actually more closely resembles the user interaction, because I tried uploading a different file type myself in the UI and wasn't able to do it. But I still wanted to test this to get more coverage, and surely if there's a will to upload the wrong file type there's a way.

The answer was to use the [applyAccept](https://testing-library.com/docs/user-event/options#applyaccept) `setup` option and set it to `false`:

{% highlight javascript %}
const { user } = setup(<ImportQuoteRequests {...props} />, {
  applyAccept: false,
});
{% endhighlight %}

That was the only thing that needed to change. So beware of the power of `setup` configurations.

# How I fixed a test using `keyboard`

Let's take a look at another test that broke after upgrading and how I went about solving it. I had a test where I wanted to make sure someone couldn't submit an invalid date via an input. Previously I was using `userEvent.type` to input an invalid date:

{% highlight javascript %}
userEvent.type(dateInput, "asdf");
{% endhighlight %}

This one was a bit tricky because in the UI, when you click the input, a datepicker modal pops up and you'd have to either hit the escape key or tab your way to the input to get the modal to go away while still being focused on the input. I tried going the escape key route, and [keyboard](https://testing-library.com/docs/user-event/keyboard) came to the rescue:

{% highlight javascript %}
await user.click(dateInput);
await user.keyboard("{Escape}asdf");
{% endhighlight %}

This simulates the user hitting the escape key and then 'asdf', which helped me test invalid data for this particular input. This again more closely resembles the actual user interaction compared to `userEvent.type`, and therefore is providing better quality test coverage.

# Refactor opportunities

I mostly just focused on fixing the breaking tests after upgrading, but since I was refactoring a lot of these tests to work with the new library version anyways, I looked for any easy refactors I could make along the way. If you haven't stumbled across this yet, bookmark this blog post by Kent C. Dodds: ["Common mistakes with React Testing Library"](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library).

A couple favorites of mine were to use [screen](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library#not-using-screen) instead of destructuring, and to use [find* queries instead of waitFor](https://kentcdodds.com/blog/common-mistakes-with-react-testing-library#using-waitfor-to-wait-for-elements-that-can-be-queried-with-find). Again, not necessary, but perhaps a good opportunity to do some cleanup and use queries that return better error messages, saving you time when debugging your tests.

# Conclusion

To sum up, `user-event` version 14 comes with lots of great improvements and increases your testing quality. While you will likely face some breaking changes, the upgrade is well worth it. Hope this helps you in your upgrade!
