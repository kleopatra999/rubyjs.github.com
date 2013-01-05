---
layout: posts
title: "javascript: when to use typeof"
---

It was ingrained in my brain to never use typeof. "JavaScript the good parts" even lists it in the section awful parts. So let's first look at what typeof really does and why it's considered evil.

## What does typeof do?

typeof returns a string with the type (not the class) of an obj.

{% highlight javascript %}

typeof 1         // => 'number'
typeof true      // => 'boolean'
typeof 'str'     // => 'string'
typeof undefined // => 'undefined'
typeof function () {} // => 'function'
typeof null      // => 'object'
typeof {}        // => 'object'
typeof []        // => 'object'
typeof /\w+/     // => 'object' or 'function'

{% endhighlight %}

The reason typeof is considered awful is that for Number/String objects typeof returns 'object' and not as we would expect 'number'.

{% highlight javascript %}

typeof new Number(1)    // => 'object'
typeof new String('f')  // => 'object'
typeof /a/              // => 'object' or 'function'

{% endhighlight %}

This makes typeof unsuitable for checking the type of an object. Instead many libraries agree upon one way of doing things. The typical implementation of isNumber() looks like that.

{% highlight javascript %}

function isNumber(obj) {
  return Object.prototype.toString.call(obj) == '[object Number]'
}

{% endhighlight %}

The beauty of this implementation is how clean and reliable it is, furthermore the same implementation can be used for all other types as well.

The reason this works is that `call( 1 )` will coerce the given JavaScript primitive `1` to a Number type `new Number(1)`. The following code snippet illustrates that process.

{% highlight javascript %}

function foo() { console.log(typeof this) }
foo.call( 1 )
// 'object' (not 'number' as you might assume)

{% endhighlight %}

Unfortunately this process is also terribly slow.

## Use typeof for performance

Performance is where typeof shines. Raw uncomprised speed. Let's improve the ubiquitious isNumber method and make it run 50 times faster.

The main idea is to only call the expensive part `toString.call` if we really need to and exit early when we know it is a number primitive.

{% highlight javascript %}

function isNumberFast(obj) {
  if (typeof obj == 'number') return true;
  return Object.prototype.toString.call(obj) == '[object Number]'
}

{% endhighlight %}

`isNumberFast` is now blazingly fast when you pass it a number primitive. However it is still slow for other primitives such as string, boolean, undefined. We can improve on that by simply adding an additional check that returns quickly for any other primitive.

{% highlight javascript %}

function isNumberFaster(obj) {
  if (typeof obj == 'number') return true;
  if (typeof obj != 'object') return false;
  return Object.prototype.toString.call(obj) == '[object Number]'
}

{% endhighlight %}

See the benchmarks here: [http://jsperf.com/isnumber-with-typeof](http://jsperf.com/isnumber-with-typeof)

## Use typeof before checking for properties

One more case where typeof can have a significant impact is when checking for properties.

{% highlight javascript %}

function wrap(obj) {
  if (obj && obj._wrapped) return obj;
  return {_wrapped: obj};
}

{% endhighlight %}

The wrap method checks if the obj has a property _wrapped and if so returns early. The if clause first checks if obj is falsey and if it contains a \_wrapped property. When passing a primitive it has to be typecasted to its corresponding class first to check for the property. This costly typecast can easily be avoided.

{% highlight javascript %}

function wrap(obj) {
  if (obj && typeof obj == 'object' && obj._wrapped) return obj;
  return {_wrapped: obj};
}

{% endhighlight %}

Another low-hanging fruit to improve performance by a magnitude. Check benchmarks:
[http://jsperf.com/typeof-prefix-for-property-checks](http://jsperf.com/typeof-prefix-for-property-checks)

## Do not overuse these techniques

Do not use these techniques everywhere. Restrict yourself to methods that are heavily used internally. There is a tradeoff between code readability and performance. RubyJS uses typeof checks for the R() method and to typecast method arguments (or rather to avoid unnecessary typecast).

### Further reading

- [javascriptweblog.wordpress.com/2010/09/27/the-secret-life-of-javascript-primitives/](http://javascriptweblog.wordpress.com/2010/09/27/the-secret-life-of-javascript-primitives/)
- [developer.mozilla.org/en-US/docs/JavaScript/Reference/Operators/typeof]()





