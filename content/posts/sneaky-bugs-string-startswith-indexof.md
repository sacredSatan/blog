---
title: "Sneaky bugs: startsWith and indexOf with empty string"
date: 2022-08-15T10:22:26+05:30
slug: 2022-08-15-sneaky-bugs-string-startswith-indexof
type: posts
draft: false
categories: ["programming"]
tags: ["javascript", "bugs", "string"]
summary: "Found another sneaky bug while using `String.startsWith()`."
---

`String.startsWith()` returns a boolean after testing whether the string begins with the search string passed to it, and has a second argument to specify the starting location for the search.

All straightforward until you try to pass an empty search string, in which case it always returns `true`. 

```js
  const str = "Artyom";
  str.startsWith("Art"); // expected: true, actual: true
  str.startsWith("yom"); // expected: false, actual: false
  str.startsWith(""); // expected: false?, actual: true
  str.startsWith("", Infinity); // expected: false?, actual: true
```

I didn't expect that to happen, and checking on MDN didn't directly help since the page for [startsWith](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/startsWith) doesn't mention this behaviour. 

However, the MDN page links to the [specification](https://tc39.es/ecma262/multipage/text-processing.html#sec-string.prototype.startswith) which describes the steps performed while this method is called. It has a step that returns `true` if the search string's length is `0`.

```
  [...]
  9. Let searchLength be the length of searchStr.
  10. If searchLength = 0, return true.
  [...]
```

`String.indexOf()` has a lot of gotchas when it comes to empty strings but I expected this newer method to be better in this regard. At least the behaviour of `indexOf` is well defined in the [MDN page](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/indexOf#return_value).

Since I expected the call with empty search string to return `false`, I introduced a bug in the code which sneaked past me even after reading through the code as I wasn't aware of this behaviour.
