---
layout: post
title: Dealing with Travis CI console output
description: "When using RSpec's progress format it's not enough"
modified: 2013-08-19
category: articles
tags: [continuous integration, Travis CI]
comments: true
draft: true
---

When we work with Ruby projects, [Travis CI](https://travis-ci.org/){:target="_blank"} will run [by default](http://about.travis-ci.org/docs/user/languages/ruby/#Default-Test-Script){:target="_blank"} our `rake test` recipe. But what if we need to run many scripts at the same time?

Wait, why would we ever do that? Well, for instance, to get better control over the Travis CI console output. You know... scrolling through pages and pages looking for our failing specs. Generally speaking it could come in handy when using the [RSpec's progress format](https://www.relishapp.com/rspec/rspec-core/v/2-14/docs/command-line/format-option){:target="_blank"} it's not enough or you're running RSpec test suites mixed with other kinds of scripts and tests.

In order to get rid of all that noise, we could create a script like the one below. Let's name it `script/ci`:

{% gist 6099003 %}

Let's make it executable by using the `chmod` command:

{% highlight bash %}
$ chmod +x script/ci
{% endhighlight %}

After that, we need to specify in our `.travis.yml` file that we need to override the default script and run `script/ci` instead:

{% gist 6250191 %}

So now at the end of Travis CI console output we will get the list of our failing test suites:

![travis output without collapsing](/images/travis_without.png)

That's interesting but it's not enough, we still need to scroll through the page to detect the failures.

A step beyond could be to collapse each suite output just like Travis CI already does with all the preliminary steps like `bundle install` and so on.

Digging into the Travis CI [source code](https://github.com/travis-ci/travis-build/blob/7e78698c40f037da9d18579345ae041c6cd43189/lib/travis/build/shell/dsl.rb#L58){:target="_blank"} I found out how they achieve that.

{% highlight ruby %}
def fold(name, &block)
  raw "echo -en 'travis_fold:start:#{name}\\r'"
  result = yield(self)
  raw "echo -en 'travis_fold:end:#{name}\\r'"
  result
end
{% endhighlight %}

So now we can just wrap each test suite and introduce the same solution in our script.

{% gist 6096827 %}

Now that we have each test suite output collapsed we have a nicer output and we could access the test suites that failed to find out easily errors and back traces.

![travis output with collapsing](/images/travis_collapsed.gif)

So, that's all! Please feel free to ping me for suggestions, ideas, improvements, rants, etc.
