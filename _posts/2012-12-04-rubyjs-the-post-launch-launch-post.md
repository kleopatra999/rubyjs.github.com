---
layout: posts
title: RubyJS - The post-launch launch post
---

Last week I launched RubyJS at [jscamp.asia](jscamp.asia).

### RubyJS is a port of the Ruby core-lib to JavaScript

RubyJS includes classes like String, Array, Numbers and almost all of the methods are implemented and are compliant (not just inspired) to Ruby.

### It is not about writing Ruby in JavaScript

It's all about the methods and having a proper standard library. There is no intent to mimic the object/class model or metaprogramming features of Ruby. Also it is not about the syntax.

You are still writing JavaScript but now have a set of convenience methods at your disposal. RubyJS does not save you from learning JavaScript, but it saves you from using and learning various 3rd party libraries.

For completeness sake everything is ported 1 to 1, this does however not mean you have to use everything, e.g. use native numbers for  arithmetic (+,-,/,*) operations rather than the RubyJS methods.

### The JS way. Similar to what you're already using

RubyJS is actually not that much different from what you're already doing.

{% highlight javascript %}
// underscorejs
_.chain([]).sortBy(fn).map(fn).value()
// RubyJS:
R([]).sortBy(fn).map(fn).toNative()
{% endhighlight %}

RubyJS objects are lightweight wrappers around native objects. You can easily get the wrapped natives back. In most cases you don't even have to call toNative(). JavaScript coerces correctly out of the box:

{% highlight javascript %}

var str = "#"+ R('foo').rjust(5);
// => '#  foo'
R(1.2345).round(2) + 1
// => 2.23

{% endhighlight %}

Symbol#to_proc is an example of a pragmatic decision RubyJS took. Symbols just dont make sense in JS, but writing [1.23,2.3].map(&:round) is more convenient. So instead of a Symbol class, RubyJS instead implements a `proc` method, that gives the same benefits, plus more.

{% highlight javascript %}

R([1.234, 3.232], true).map(R.proc('round'))  // => [1, 3]
// Assign it to a global var for better looks:
var proc = R.proc;
// R.proc accepts arguments that are passed to the block:
R([1.234, 3.232], true).map(proc('round', 2)) // => [1.23, 3.23]

{% endhighlight %}


### Iterators, Block and Functions

A Ruby block, lambda, Proc maps to a JavaScript function. Hard-core rubyists might scream in pain over this simplification. However in 99% of cases the small differences really don't matter. Block arguments work the same in RubyJS.

{% highlight javascript %}

R([1,2,3]).map(function (a) { return a * a} )
// => [1,4,9]
R([[1,-4], [3,4]]).map(function (a  ) {return a[0] * a[1];})
R([[1,-4], [3,4]]).map(function (a,b) {return a * b;})
// => [-4, 12]

{% endhighlight %}

Iterators are chainable through Enumerator objects. It allows for flexible and consice iterating.

{% highlight javascript %}

R(1).upto(5, function (i) { R.puts("# "+i) })
R(1).upto(5).inject(0, function (mem, i) { return mem + i; })

R.w('foo bar baz').include('bar'); // => true
R([1,15,8]).sort();   // => [1,8,15]
R([1,15,8]).minmax(); // => [1,15]
R([1,-15,8], true).min_by(R.proc('abs')); // => 1
R([1,'foo',2]).sort();
// ArgumentError, does not try to sort different types.

R.w('fizz buzz').eachWithIndex().map(function(e,i) {
  return i + ": " + e;
}).toNative(true) // ['0 fizz', '1 buzz']

// Breaking out of loops with breaker objects.
R.catch_break(function (breaker) {
  R([1,2,3]).each(function (i) { if (i == 2) breaker.break(i); })
})
// => 2



{% endhighlight %}

An overview of [Enumerable methods](/doc/classes/RubyJS/Enumerable.html) and [Array](/doc/classes/RubyJS/Array.html).



### What is the advantage over using x,y,z?

There are many libraries out there that implement missing functionality of JavaScript. Underscorejs, stringjs, underscore.string, momentjs. Each with an own API, documentation.

{% highlight javascript %}
// An example using underscore and stringjs
arr = ['foo', 'bar']
str = _.map(arr, function (w) { return S(w).capitalize().s })
       .join(',');
S(str).pad(50).s
{% endhighlight %}

- RubyJS is chainable across types which leads to clean code
- One API, one way of doing things
- The Ruby core-lib is battle-hardened, small, concise yet powerful

{% highlight javascript %}
// Rewritten in RubyJS
R(arr, true).map(function (w) { return w.capitalize() })
  .join(',')
  .center(50)
  .toNative();
// In most cases calling toNative() is actually not necessary.
{% endhighlight %}



### Lightweight, fast, works across browsers (and nodejs)

Not just the file-size (20kb minzipped), but also the implementation using wrapper objects. Features that have little advantage but produce a lot of extra code or felt unnatural in JavaScript were skipped. There are numerous performance optimizations to make it fast, and all major browsers are supported.


### <del>What about that price tag</del>

<del>It started out as a silly experiment and eventually became an obsession. There is a lot of grunt work involved, porting specs, code, docs. Yet still I'm passionate about working on RubyJS. Charging money for RubyJS would allow me to continue working on RubyJS full-time, and give me the incentive to also do the boring parts. Not many people can pull off that stunt. At least I want to give it a try.</del>

<del>So I decided to charge 190$ for a commercial license. It's alpha but it'll be beta soon, and you're getting upgrades for 1 year after the beta launch. Support is given on a best-effort basis.</del>

<del>How did I came up with 190$? On a recent small JS project I've spent at least half a day just for adding missing functionality to JS. I would have gotten my money back within the first week. How valuable is your time?</del>

After one week of intense talks I decided to make it free and MIT.

Go check the [getting started guide](/gettings-started.html), or the [api docs](/doc/index.html). Try it in your Firebug/webdev console or get the files on [github.com/rubyjs/core-lib](http://github.com/rubyjs/core-lib).
