---
layout: post
title:  Create your own Jekyll theme gem
date:   2018-09-30 16:40:09 -0700
tags:   jekyll
---

## Why create your own theme gem?

Creating your own Jekyll theme gem is awesome for several reasons:

• Jekyll is an open-source platform where you can learn from others, and your own creation may inspire the next person to take the torch and create something great of their own.  

• You will learn a ton in the process, including how to publish a gem.  

• You can customize and make your theme exactly how you'd like it to be.  

<!-- more -->

• That all goes without saying that it's an excellent way to show off your coding chops.

## Planning your creation

Now that you're sold on creating your own Jekyll theme gem, let's dive into the first consideration: what do you want to do? You don't have to know right away, but it helps to have some idea of what you want to accomplish. Take a look at a lot of other Jekyll themes and see what you like, what you don't like, and maybe you'll have some new ideas that you're not really seeing out in the wild. For instance, I didn't like that you couldn't paginate within a specific group of tags, and I didn't see many examples out there that had this functionality, so I made that a goal of mine. A couple of great places to look at themes are the [awesome-jekyll-themes](https://github.com/planetjekyll/awesome-jekyll-themes) GitHub page, and the [Jekyll Themes](http://jekyllthemes.org/) site.

You should also try downloading several different gem-based themes and notice how easy is it to setup? How easy it to configure? Being able to get up and running quickly with a new theme is an important consideration. Although I am really proud of the [jekyll-theme-collider](https://github.com/ryancolorcafe/jekyll-theme-collider) gem I created, there are quite a few steps to get up and running.

A lot of this is due to the fact that I was using the [jekyll-paginate-v2](https://github.com/sverrirs/jekyll-paginate-v2) plugin, which requires an unconventional folder structure. Other customizations I made contributed to this as well. So just beware that the more you stray from the conventions of Jekyll, the more complicated your setup may will be. Notice how that in most gem-based themes, it is only the `assets`,  `_layouts`, `_includes`, and `_sass` directories that are included in the gem. Any other custom files or folders you add will make the theme harder to setup. If you haven't already, make sure to give this [Jekyll docs page](https://jekyllrb.com/docs/themes/) on themes a good look before you begin.

Now that we've gone over all of these considerations, lets take a look at the nitty gritty of how to create your own theme.

![Dr. Jekyll in the laboratory](../assets/images/dr_jekyll_laboratory.jpg)

## How to create your gem-based theme

To create a new Jekyll scaffold for your gem theme, simply run:  

&nbsp;&nbsp;`jekyll new-theme jekyll-theme-awesome`

This will create many files and folders for you to base your theme on. The `jekyll-theme-awesome.gemspec` file that will be created is important to publishing your gem, so make sure to fill out all the required info there. You will at the very least need to change the `spec.summary` and `spec.homepage` values in order for you to successfully run the `bundle exec jekyll serve --watch` command.

The gemspec file is where all the important information for your gem lives, including the author name, version number, and what dependencies the gem has (such as plugins). The `spec.files` line is where you say what files should be included in the gem. You can edit this, but it definitely goes against the conventions of Jekyll, which is part of why Jekyll is so easy to use, so just be aware of this if you choose to do so. In this line you can see that by default the `assets`, `_layouts`, `_includes`, `_sass`, `LICENSE.txt` and `README.md` files and folders are all included in your gem.

If plugins are required in order for your theme to work, you will need to add each plugin in this file like so:

&nbsp;&nbsp;`spec.add_runtime_dependency "jekyll-paginate-v2", "~> 1.7"`

As a quick refresher, here's what all the other files and folders that were created are for:
- The `_includes` folder is for reusable partials that you may want to include throughout your theme.
- The `_layouts` folder where you keep the main template files.
- The `assets` folder is where you'll keep any images, graphics, etc that you'll want to include in your gem. This is also where you'll import your Sass files. For instance, inside this folder I have a `css` folder with `main.scss` inside. In this file you'd simply include the following:

{% highlight ruby %}
---
---
@import "jekyll-theme-awesome";
{% endhighlight %}

- The `_sass` folder is where your style partials will go. And if you setup your css as above in the `assets` folder, you'd place your `jekyll-theme-awesome.scss` file here, and then import whatever other Sass partials you'd like here. You'll also need to include this in your `config.yml` so that Jekyll knows where to pull in your Sass partials:

{% highlight yaml %}
sass:
    sass_dir: _sass
{% endhighlight %}

- The `Gemfile` will simply point to your `.gemspec` file, and is where Bundler looks to manage your gems.
- The `README.md` is where you'll provide the documentation for your theme, including how to install and configure the gem.
- The `LICENSE.txt` is where you state what kind of license your theme falls under. By default it is set to be under the [MIT](https://opensource.org/licenses/MIT) license.

Without having done anything but edit the `.gemspec` file's `spec.summary` and `spec.homepage` configuration, you could run the `bundle exec jekyll serve --watch` command, head to `localhost:4000` in your browser, and you wouldn't see much but an index of your site directory. You can add an index.html file to your root directory and add something like the following to see something slightly more interesting:

{% highlight ruby %}
---
title: Home
layout: default
---
# Yello?
{% endhighlight %}


So now you can really go crazy and start to make your theme. Give some style to your `_layouts/post.html` template, create some example posts with lots of varied content so you prepare your theme for all the various use cases. Add [pagination](https://jekyllrb.com/docs/pagination/), add [tags](https://jekyllrb.com/docs/plugins/tags/), create an about page. Add lots of styling. Go nuts.

Another thing you'll really want to do is create an example `config.yml` file, because you can make it really quick and easy for someone downloading your gem to get setup. For example, here is part of the `config.yml` file I created for the gem theme I created:

{% highlight yaml %}
title: Rando Dev
email: example@example.com
description: >
  This is the Collider Jekyll Theme.
baseurl: ""
url: "https://jekyll-theme-collider.netlify.com"
hosted_url: ""
github_username: ""
linked_in_profile: ""
full_name: Rando Dev
user_description: Software Engineer
disqus:
  shortname: jekyll-theme-collider-netlify-com
{% endhighlight %}


Instead of someone having to track down in every partial where their name is displayed in the code, they can simply edit the `full_name` line. In the code where you want the person's name to display, you would set up something like so:

{% raw %}
&nbsp;&nbsp;`<p>{{site.full_name}}</p>`
{% endraw %}

Any of the `config.yml` variables are available throughout your site using the {% raw %}`{{site.variable_name}}`{% endraw %} convention.

Once you've done all the cool things you've wanted to do with your theme and feel it is ready to be published, sign up for an account at [RubyGems.org](https://rubygems.org).

Then you'll build the gem using this command:

&nbsp;&nbsp;`gem build jekyll-theme-awesome.gemspec`

After you run this, a file will appear in your root directory that will look like this:

&nbsp;&nbsp;`jekyll-theme-awesome-0.1.0.gem`.

To push up and publish your theme, you'll use this command:

&nbsp;&nbsp;`gem push my-theme-0.1.0.gem`

You will be prompted to sign in with your RubyGems.org credentials, and once you do so your gem will be published!

Way to go. Pat yourself on the back. 🎉

However, you can't just stop there. That would be irresponsible. You'll need to give your gem theme a good testing. Try downloading your gem theme to a new Jekyll project. How easy is it to setup? Does it work without any hitches and minimal setup? Do you run into any snags? This brings me to my next topic (read: rant), documentation.

As an English major who values good communication, I take these things seriously. Why go through all the work of creating an awesome theme while leaving your potential users without a clue how to use it? Really come from the perspective of someone who knows very little about Jekyll or coding at all (ok maybe not that far, but you get the idea). Take nothing for granted. Assume they run into every possible issue while attempting to setup your gem theme and won't magically know where to look for your theme's more hidden features.

So now you've created awesome documentation, good on you 👏. But you want to continue adding to and developing your theme further. Once you've created the next round of changes you want to make and push up to your gem, you'll use the following steps:

In your `.gemspec` file, change the `spec.version` configuration to the desired version number. Use this [guide](https://guides.rubygems.org/patterns/#semantic-versioning) from RubyGems.org to determine what your version number should be.

Then run the `gem build jekyll-theme-awesome.gemspec` command again to build the next version of your gem. To push up your new gem version, run this command as before, but with your new version number at the end, like so:

&nbsp;&nbsp;`gem push my-theme-0.1.1.gem`

That's all there is to it.

Amazing, you've now created your very own Jekyll gem theme!🍾 It's a great feeling to contribute to open-source and have a bundle of code out there that you created and someone can possibly benefit from. Please [contact me](https://ryandevelops.com/contact) if you have any questions about how to create your own gem theme, I'm happy to help!
