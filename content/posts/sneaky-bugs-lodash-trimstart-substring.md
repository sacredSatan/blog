---
title: "Sneaky Bugs: using lodash/trimStart to trim off a leading substring"
date: 2021-11-28T16:07:54+05:30
slug: 2021-11-28-sneaky-bugs-lodash-trimstart-substring
type: posts
draft: false
categories: ["programming"]
tags: ["javascript", "bugs", "lodash"]
summary: "Sneaky bugs are bugs that escape your first glance through the code. I've had this recently happen to me with `lodash/trimStart`."
---

Sneaky bugs are bugs that escape your first glance through the code. I've had this recently happen to me with `lodash/trimStart`. 

`trimStart` removes leading whitespace or passed in characters until it hits a character that isn't one of the characters to trim. In this case it was used to remove a substring from the beginning of the input string.

```js
  trimStart('Mr. Mario', 'Mr. ') // expected: 'Mario', actual: 'ario'
  trimStart('+919999999999', '+91') // expected: '9999999999', actual: ''
```

You're better off using `replace` with `startsWith` to do the job. 

```js
  const inputStr = 'Mr. Mario'
  const subStr = 'Mr. '
  startsWith(inputStr, subStr) && replace(inputStr, subStr, '') // expected: 'Mario', actual: 'Mario'
```

This wouldn't necessarily happen to someone who's familiar with `lodash/trimStart` but it certainly sneaked past me.
