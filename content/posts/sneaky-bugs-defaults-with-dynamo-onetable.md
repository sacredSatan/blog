---
title: "Sneaky Bugs: using defaults in schema with dynamodb-onetable"
date: 2024-12-31T19:09:08+05:30
slug: 2024-12-31-sneaky-bugs-defaults-with-dynamodb-onetable
type: posts
draft: false
categories: ["programming"]
tags: ["bugs", "dynamodb", "aws", "dynamodb-onetable"]
summary: "dynamodb-onetable is basically mongoose but for dynamodb. The schema def allows you to define fields with their types and an optional default value, with a behaviour that I didn't expect, and caused a bug to sneak in."
---

[`dynamodb-onetable`](https://doc.onetable.io) is basically mongoose but for dynamodb. It allows you to model data, and takes care of validation etc. based on your schema. It also does a bunch of other things, including simplification of dynamodb requests, as they can get a bit complex. 

The schema def allows you to define fields with their types and an optional default value, with a behaviour that I didn't expect, and caused a bug to sneak in.

I was not going to write about it since it didn't happen to me, but I wanted to get one post in for 2024, and I'm out of ideas and time. 

The schema def looks something like this:

```js
const schema = {
  version: "0.0.1",
  indexes: {
    primary: { hash: "pk", sort: "sk" },
  },
  models: {
    TestModel: {
      pk: { type: String, required: true, value: "test" },
      sk: { type: String, required: true, value: "${_type}" },

      withoutDefault: { type: Boolean },
      withDefault: { type: Boolean, default: false },
    },
  },
};
```
The model `TestModel` has two fields other than the primary keys. In the current state, the pk/sk fields will always equal to `test/TestModel`, which would be an issue if you were trying to write multiple records, but not in this case. 

The usecase involved upserting records, and `dynamodb-onetable` updates fields with default values if they're missing from the `upsert` call args. This was a bit unexpected as you'd only expect the `default` value to come into play when the field is previously unset. 

### create

```js
  // without args, only populates the default field
  await TestModel.create({});

  // expected and actual is same
  // {
  //   withDefault: false,
  // }
```

```js
  // with both fields, so populates them both
  await TestModel.create({
    withDefault: true,
    withoutDefault: true,
  });

  // expected and actual is same
  // {
  //   withDefault: true,
  //   withoutDefault: true,
  // }
```

All good so far.

### upsert vs update

```js
  // current item
  // {
  //   withoutDefault: true,
  //   withDefault: true,
  // }

  // update works as expected
  await TestModel.update({
    // ...pk/sk
    withoutDefault: false,
  });

  // expected and actual is same
  // {
  //   withoutDefault: false,
  //   withDefault: true,
  // }

  // upsert replaces all the missing fields with their default values if it exists
  await TestModel.upsert({
    // ...pk/sk
    withoutDefault: true,
  });

  // expected
  // {
  //   withoutDefault: true,
  //   withDefault: true,
  // }

  // actual
  // {
  //   withoutDefault: false, // replaced this
  //   withDefault: true,
  // }
```

Since the record got upserted multiple times with different set of args, not always containing the field with default, the actual value got replaced with the default value.
