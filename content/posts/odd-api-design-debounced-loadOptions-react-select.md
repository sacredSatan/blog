---
title: "Odd Api Design: loadOptions prop in async React Select"
date: 2022-10-16T22:48:36+05:30
slug: 2022-10-16-odd-api-design-debounced-loadoptions-react-select
type: posts
draft: false
categories: ["programming"]
tags: ["javascript", "api-design", "react-select", "bugs"]
summary: "react-select/async has a loadOptions prop that helps you filter and display options based on some input string. The function you pass to loadOptions has two ways to set the options to display. This is what caused an unexpected (to me) issue. This isn't a sneaky bug since I caught the issue while developing."
---

`react-select/async` has a `loadOptions` prop that helps you filter and display options based on some input string. The function you pass to `loadOptions` has two ways to set the options to display. This is what caused an unexpected (to me) issue. This isn't a sneaky bug since I caught the issue while developing.

`react-select/async` passes two args to the function you pass in `loadOptions`, first arg is the filter string, and second arg is a callback function. Your function can either pass the options to the callback function, or return a promise that resolves to the options that need to be displayed.

I had to use the callback function way since `react-select/async` doesn't play well with debounced promises being returned from `loadOptions`, and the general advice on filed issues on Github was to use callback to set options instead. I didn't want to spend time going into why callbacks worked vs promises didn't.

Here's what my code sort of looked like:

```jsx
  const loadOptions = async (inputValue, callback) => {
    const displayOptions = await asyncCallForOptions(inputValue);
    callback(displayOptions);
  }
  
  return (<AsyncSelect 
    loadOptions={loadOptions}
  />);
```

This caused an issue where `AsyncSelect` would render options at first but it'd change to an empty dropdown almost immediately after.

What I didn't realize was I'm still passing a function that returns a promise to `loadOptions`, which will resolve to `undefined`. This eventually led me to the source code which cleared things up.

Here's what the code to handle `loadOptions` in `react-select/async` looks like:

```js
  // https://github.com/JedWatson/react-select/blob/master/packages/react-select/src/useAsync.ts#L112
  
  const loadOptions = useCallback(
    (
      inputValue: string,
      callback: (options?: OptionsOrGroups<Option, Group>) => void
    ) => {
      // propsLoadOptions is the `loadOptions` passed as a prop
      if (!propsLoadOptions) return callback(); 
      const loader = propsLoadOptions(inputValue, callback);
      if (loader && typeof loader.then === 'function') {
        loader.then(callback, () => callback());
      }
    },
    [propsLoadOptions]
  );
```

This led to the options being set by the `callback(displayOptions)` call, and then replaced by `undefined` as the async function's promise resolves.

```js
  // https://github.com/JedWatson/react-select/blob/master/packages/react-select/src/useAsync.ts#L118

  // causes the `callback(displayOptions)` call, `loader` now has a Promise that resolves to `undefined`
  const loader = propsLoadOptions(inputValue, callback);
  // loader.then exists because `propsLoadOptions` is an async function
  if (loader && typeof loader.then === 'function') {
    // this sets the display options to `undefined`
    loader.then(callback, () => callback());
  }
```

I just changed my function so it wasn't an async function anymore. It was entirely my fault for not following the api.

I think having `loadOptions` only take functions that return/resolve to the options to be displayed, and having another `loadOptionsCallback` prop that strictly does the things the callback way, would make things a lot simpler.
