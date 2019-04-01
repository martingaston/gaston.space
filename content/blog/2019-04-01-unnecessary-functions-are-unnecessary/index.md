---
title: Unnecessary Function Wrapping Is Unnecessary
date: "2019-04-01T19:52:51.169Z"
description: Hey, it's the little mistake we're all bad for!
---

Unnecessary function wrapping is a bad habit I think everybody has done and will almost certainly do again - in the interests of transparency, reader, I honestly do it **all the time**.

In my flat of code, it's one of the first things I try and spruce up, fluffing down the pillows of unnecessary function wrapping before I have guests round to code review.

Or, more realistically, I often realise I've done it five seconds after I've sent something off for review. It's so frustrating because it adds verbosity and, worse, increases the cost of maintaining.

Consider this code:

```javascript
class Shop {
  constructor(items = []) {
    this.items = items;
  }

  updateShopPricesAtEndOfDay() {
    this.items = this.items.map(item => updateItem(item));
    return this.items;
  }
}
```

I'm not here to rag on classes, my particular bone of contention is within the call to `map`. Because this:

```javascript
this.items.map(item => updateItem(item));
```

...is just a more verbose way of writing:

```javascript
this.items.map(updateItem);
```

So, what exactly is going on here?

1. The `map` method is called on the array `this.items`.
2. `map` takes a callback function as its argument, which is invoked once for every element in the array - returning a new array based on the the results of the callback applied to each item.
3. The callback function is invoked with the `currentValue` argument assigned to the variable `item`.
4. The callback function, when invoked, calls `updateItem` with the `item` variable.

In the second version, `updateItem` is simply passed as the callback function - because functions in JavaScript are first class - which is called by `map` with the `currentValue` argument.

A good pattern to spot is `.method(some_variable => function(some_variable))`, which can almost always be shortened to `.method(function)`.

More info can be found in *[Professor Frisbee's Mostly Adequate Guide to Functional Programming](https://github.com/MostlyAdequate/mostly-adequate-guide/blob/master/ch02.md)*.
