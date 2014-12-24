Δ dffptch.js
==========
A micro library for diffing and patching JSON objects using a compact diff format.

Why
---
If your JavaScript client is sending a lot of updates to your backend – as it might
in a collaborative app, a real-time game or a continously saving app – 
transfering the entire changed JSON wastes a lot of bandwitdh. dffptch.js
makes sending only the changes in a compact format very easy.

Example
-------
```javascript
var rabbit = {
  name: 'Thumper',
  color: 'grey',
  age: 2,
  bestFriend: 'bambi',
  foodNotes: {
    grassHay: 'his primary food'
  } 
};
// We make some changes to our rabbit
var updatedRabbit = {
  name: 'Thumper', // Still the same name
  color: 'grey and white', // He is white as well
  age: 3, // He just turned three!
  // we delete `bestFriend` – Thumper has many friends and he likes them equally
  foodNotes: {
    grassHay: 'his primary food', // Grass hay is still solid food for a rabbit
    carrots: 'he likes them a lot' // He also likes carrots
  } 
};
var delta = diff(rabbit, updatedRabbit);
// Delta is now a compact diff representing 1 deletion, 2 modifications and 1
// nested added property. The diff format might look odd, but is actually very
// simple as explained below.
assert.deepEqual(delta,
                 {"d": ["1"],
                  "m": {"0": 3, "2": "grey and white"},
                  "r": {"3": {"a": {"carrots": "he likes them a lot"}}}
                 });
```

Features
--------
* __What you expect__ – Handles all types of changes. Added, modified and
  deleted properties. Nested objects and arrays.
* __Compact one way diffs__ – No needless verbosity. Property names
  are shortened to single characters while still remaining unambiguous.
* __Performant__ – Sane choices of algorithms. For instance, when generating
  the delta the keys of the old and new object are sorted, and then all changes
  are found in a single pass over both objects at the same time.
* __Very small__ – Too many huge libraries claim to be lightweight! This one is
  not among them. By having a tight focus on its targeted use case and a
  carefull implementation dffptch.js is barely 600B minified. Gzipped it's only
  420B.
* __Readable source code__ – Well commented and less than 50 sloc. UglifyJS2
  takes care of almost all golfing.
* __Availability__: Available both as a CommonJS module, a AMD module and a
  plain old global export.
* __Tested__: The test suite covers every feature.

Compared to other diff & patch libraries
----------------------------------------
* It is significantly smaller. Ten times smaller than some alternatives.
* In common cases dffptch.js generates smaller diffs because it only patches one way and thus can shorten property name.
* It doesn't handle complex array changes as well as others. See the section on limitations below.

Install
-------
```
bower install dffptch
```
or
```
npm install dffptch
```

How the diff format works
-------------------------
The diff format is an object with up to four properties. `a` is an
object representing added properties. Each key is the name of a property and
each value is the value assigned to the property. `m` is a similar object but
for modified properties and with shortened keys. `d` is an array with
deleted properties as elements. `r` contains all changes to nested objects
and arrays, it recursively contains the four properties as well for the
nested object.

An example
```javascript
{
  a: {foo: 'bar'}, // One aded property
  m: {'3': 'hello'}, // One modified property
  d: ['5'], // One deleted property
  r: {'3': { ... }} Changes to one nested object
}
```
In `m`, `d` and `r` the property names are shortened to single characters.  The
algorithm works like this: The keys in the original object are sorted, giving
each key a unique number.  The number is converted to a character using
JavaScripts `String.fromCharCode` with an offset so the first key is assigned
to the char '1' (this avoids the characters '/' and '\' that require escaping
in JSON.

So for this object
```javascript
{
  foo: 'bar',
  sample: 'object',
  an: 'example'
}
```
we'd get the sorted keys `['an', 'foo', 'sample']` and thus `an` whould be shortened
to `'1'`, `foo` to `'2'` and `sample` to `3`. There are _a lot_ of uniqode characters
so this approach is safe no matter how many properties your objects have.

Browser support
---------------
dffptch.js is environment independent (neither Node nor a browser is required).
It does however use the two ECMAScript 6 functions
[`Object.keys`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/keys) and
[`Array.prototype.map`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Array/map).
If you require support for < IE9 you should polyfill those functions
(splendid polyfills are included in the MDN links above).

Limitations
-----------
The differences generated by dffptch.js can only take you from _a to b_.
Not from _b to a_. This is by design and is necessary for the compact format.

dffptch.js handles arrays as it handles objects.  Order is not taken into
account. If you're changing elements or append to an array this is not an
issue. However, if you're reordering or inserting elements the diffs will be
suboptimal. Finding the shortest edit distance in an ordered and possibly
nested collection whould complicate dffptch.js significantly with little
benefit. Simply flattening your data before feeding it to dffptch.js avoids the
problem.

License
-------
dffptch.js is made by Simon Friis Vindum. But copyright declarations wastes bandwidth.
thus dffptch.js is public domain or [WTFPL](http://en.wikipedia.org/wiki/WTFPL) or
[CC0](http://en.wikipedia.org/wiki/Creative_Commons_license#Zero_.2F_Public_domain).
Do what you want but please [follow me on Twitter](https://twitter.com/paldepind)
or give a GitHub star if you feel like it.

