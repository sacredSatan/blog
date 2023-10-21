---
title: "Sneaky Bugs: string substitution and string split"
date: 2023-10-21T16:13:24+05:30
slug: 2023-10-21-sneaky-bugs-string-substitution-and-string-split
type: posts
draft: false
categories: ["programming"]
tags: ["javascript", "bugs", "string"]
summary: "This is more silly than sneaky, but it did get past me and made its way to production."
---

This is more silly than sneaky, but it did get past me and made its way to production.

Here's the problem: there's a string full of HTML, but you want to strip away everything other than text and anchor links.

The code runs in browser, and here's how I went about solving it using DOMParser:

```js
  const REPLACEMENT_TEXT = "___replacement_text"
  const parsedDoc = new DOMParser().parseFromString(htmlStr, "text/html");

  // get the list of all anchor tags
  const { links } = parsedDoc;

  // collect info for anchor tags and replace their text with a common replacement term
  const anchorTags = new Map<string, { text: string, href: string }>();
  let index = 0;
  for (const link of links) {
    const replacementKey = `${REPLACEMENT_TEXT}_${index}`;
    anchorTags.set(replacementKey, {
      text: link.innerText,
      href: link.getAttribute("href") ?? "#",
    });
    link.innerText = replacementKey;
    index++;
  }

  // loop over the anchor tags, use the replacement key to find and insert
  // the anchor tag in sanitizedStr.
  let sanitizedStr = "";
  let bodyText = parsedDocument.body.innerText;

  anchorTags.forEach(({ text, href }, index) => {
    const replacementKey = `${REPLACEMENT_TEXT}_${index}`;
    const [ beforeText, afterText ] = bodyText.split(replacementKey);
    sanitizedStr += beforeText;
    sanitizedStr += `<a href="${href}">${text}</a>`;
    bodyText = afterText;
  });
  
  if(bodyText) {
    sanitizedStr += bodyText;
  }

  return sanitizedStr;
```

This seemed to work okay, but it threw a TypeError saying `bodyText` is `undefined` for larger html strings.

This is because I was appending index to the replacement text, and `bodyText.split("abc_1");` not only splits the text on `"abc_1"`, it also splits at `"abc_10"`, `"abc_11"` etc. I was running out of `bodyText` sooner than expected because of that.

The code only "works" if the input string has 10 or fewer links in it.

Fortunately I work with people smarter than me, and someone pointed out that I don't even need an index in the replacement text if I just use an array to store `anchorTags`.

Here's what the version that works as intended looked like:

```js
  // ...

  // collect info for anchor tags and replace their text with a common replacement term
  const anchorTags: { text: string, href: string }[] = [];
  for (const link of links) {
    anchorTags.push({
      text: link.innerText,
      href: link.getAttribute("href") ?? "#",
    });
    link.innerText = REPLACEMENT_TEXT;
  }

  let sanitizedStr = "";
  const bodyText = parsedDocument.body.innerText;
  const splits = bodyText.split(REPLACEMENT_TEXT);

  let counter = 0;
  while (counter < splits.length) {
    sanitizedStr += splits[counter];

    if (anchorTags[counter]) {
      const { href, text } = anchorTags[counter];
      sanitizedStr += `<a href="${href}">${text}</a>`;
    }

    counter++;
  }
  
  // ...

```

This was kind of similar to the `trimStart` bug where I expected it to stop after the "first match".
