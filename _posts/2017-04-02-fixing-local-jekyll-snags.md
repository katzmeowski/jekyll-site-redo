---
layout: post
title: "Fixing snags in running your Jekyll GitHub Pages site locally"
date: 2017-04-02
---

Quickly building and setting up a static site has never been easier thanks to [GitHub Pages](https://pages.github.com/) and its support for the powerful [Jekyll framework](https://jekyllrb.com/docs/github-pages/). To make deploying your website fast and convenient, GitHub handles all of the Jekyll processing &mdash; it takes all the work out of managing updates and dependencies to make sure that your site is always available.

The issue is that every time I want to test a new chunk of code I'd have to push a commit (or submit a pull request if I'm not the owner of the repository) so that GitHub can take my source, process it with Jekyll, and show the live results. But it would be easier to develop and test new code if I just ran Jekyll and deployed a copy of the website on my own computer...or so I thought. I tried to deploy a copy of this site locally and honestly the process was fairly simple, but I hit a number of annoying snags along the way. And based on what I found while Googling for the answer it looks like I'm not alone. I thought I'd write down my process both so I can remember it for the future and so I can help others who are having the same issues.

When I was setting up Jekyll on my machine, I followed [GitHub's own guide](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/), which didn't offer much troubleshooting along the way. Some of the configuration details in the GitHub Jekyll guide are important to make sure that your GitHub Pages site will work the same whether you're deploying it on your computer or on GitHub. This post will follow the GitHub guide pretty closely with the snags and fixes added along the way. I should note that I did this on Ubuntu 16.04 LTS and everybody has their own unique development environment quirks, so your milage may vary.

## Let's do this

Note: you'll need administrator privileges on your computer to do a lot of the following steps. I don't generally recommend running things as root, so make sure your user account has sudo permissions before you move on.

All of the software needed is written in Ruby, so make sure you have a recent version of Ruby.

	$ ruby --version

If you don't have version 2.1.0 or later, or you don't have Ruby at all, update or install it.

	$ sudo apt-get install ruby

GitHub uses the Ruby gem Bundler to gather and install all the other Ruby gems (including Jekyll) needed to run your website. Install it like this:

	$ gem install bundler

### Step 1

The GitHub assumes that you haven't already created a repo for your site, and so Step 1 of their guide walks through initializing a new repo. I had already built a repo for my site and cloned it to my computer, so I skipped that part. If you haven't initialized a repo for your site yet, follow Step 1 of the GitHub guide and come back here for Step 2.

### Step 2

Bundler installs all the gems for you, but you need to tell it where to find them by making a Gemfile. Depending on how you initialized your repo, you might already have a Gemfile inside the repo's root directory. If not, create a new file named `Gemfile`. Add the following lines to the Gemfile:

	source 'https://rubygems.org'
	gem 'github-pages', group: :jekyll_plugins

This Gemfile is telling Bundler to grab the `github-pages` gem, and in the process Bundler will also grab all of that gem's dependencies &mdash; Jekyll, Faraday, etc. When you run Bundler, all of those gems should get installed. Make sure your terminal is looking at the root directory of your website and run Bundler.

	$ bundle install

Bundler requires admin permission to do it's thing, so type in your user password when it asks. (Note: it's tempting to run `$ sudo bundle install` so you can skip typing your password, but don't do it! Running Bundler as root can screw things up sometimes.) If everything goes smoothly for you, skip ahead to Step 3. If not, read on; we'll get through this together.

#### Here's where things get tricky

When I did this install, Bundler was chugging along grabbing gem packages until one of those gems required the `JSON v 1.8.1` gem. The error messages said that Bundler failed to build gem native extension. A [StackOverflow question](http://stackoverflow.com/questions/21095098/why-wont-bundler-install-json-gem) about this issue showed that the issue is in the dependencies needed by the gem &mdash; in fact, most of the issues I ran into had to do with mismanaged dependencies. It seems that Bundler is great for managing when gems depend on other gems, but not so great at managing when gems depend on software packages running on the host machine. In this case, I needed the development versions of a few packages in order to make the dependencies resolve. Those development versions contain development header files that the gems were looking for. Install the following dev package to get those header files and the JSON gem issue should be fixed.

	$ sudo apt-get install libgmp-dev

Run `$ bundle install` again, and see if any more errors happen. If you're following along and your install magically comes back error-free, skip ahead to Step 3. But more than likely Bundler will throw another error at you. This time, I ran into similar trouble with installing the Nokogiri gem. This time, Nokogiri's [installation tutorial](http://www.nokogiri.org/tutorials/installing_nokogiri.html) had the answer: more missing development header files. Run the following two lines to make sure you have all the packages you need to fix Nokogiri's installation.

	$ sudo apt-get install build-essential patch
	$ sudo apt-get install ruby-dev zlib1g-dev liblzma-dev

When that's done, try running `$ bundle install` again. At this point Bundler was able to install all the gems needed on my machine, so I was ready to move on.

### Step 3

If you're still following along with the official GitHub guide, Step 3 assumes that you haven't built any actual content in your site's repo yet &mdash; you've only made a Gemfile and installed all the relevant gems. I already had actual content in my site's repo and had it cloned to my local machine, so I skipped Step 3. If you haven't done that, follow the GitHub guide and come back here for Step 4.

### Step 4

The end is so close. We've come so far. All that's left to do is run Jekyll and make it serve up the site locally. Make sure you're in the root directory of your site's repo and execute Jekyll.

	$ bundle exec jekyll serve

I did a dumb thing the first time I tried this. I got really excited that everything finally installed properly that I didn't notice I wasn't in the root of my repo. If you run Bundler outside of the root directory, it will yell at you that it can't find the Gemfile. If that happens to you, don't panic! Just `cd` your way back to the root and try again.

So I'm back at the root directory and I try again and...Jekyll can't find a JavaScript runtime to use. [There are lots](https://github.com/rails/execjs) of different JS runtimes that Ruby programs like Jekyll can use to process your site, but none of them were installed on my machine yet. You might already have one installed on your machine and Jekyll might work just fine, but if not you'll need to install one. Luckily, Node.js is compatible with Jekyll and is popular enough to have good documentation, so that's the runtime I chose to use. Install it like this:

	$ sudo apt-get install nodejs

When Node.js finishes installing, try to execute Jekyll again and...finally! If it worked, the terminal should include a line that reads `Server running... press ctrl-c to stop.` or something similar. That terminal output should also include a line that reads `Server address: http://127.0.0.1:4000/`. Go to that address or `http://localhost:4000` to view and test your site locally.

Wow, what an adventure. It took me the better part of an afternoon to research, test, and troubleshoot my local Jekyll install, but it was worth it to be able to play with new features and functionality locally before committing them to the live, public version of my site. I hope you can learn from my mistakes and struggles so you can spend less time setting up the environment and more time implementing cool new things.