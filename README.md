# idlecycles

just another single HTML live coding playground..

## Examples

- [cat](https://felixroos.github.io/idlecycles/#Y2F0KCdjeWFuJyUyQyUyMCdtYWdlbnRhJyUyQyd5ZWxsb3cnKQ==)
- [seq](https://felixroos.github.io/idlecycles/#c2VxKCdjeWFuJyUyQydtYWdlbnRhJyUyQyd5ZWxsb3cnKQ==)
- [seq > seq](https://felixroos.github.io/idlecycles/#c2VxKCdjeWFuJyUyQyUyMHNlcSgnbWFnZW50YSclMkMlMjAneWVsbG93Jykp)
- [3 vs 2](https://felixroos.github.io/idlecycles/#c2VxKHNlcSgnY3lhbiclMkMlMjAnbWFnZW50YSclMkMlMjAneWVsbG93JyklMkMlMjBzZXEoJ2N5YW4nJTJDJTIwJ21hZ2VudGEnKSk=)
- [fast](https://felixroos.github.io/idlecycles/#ZmFzdCgyMCUyQyUyMHNlcSgnY3lhbiclMkMlMjAneWVsbG93Jykp)
- [slow](https://felixroos.github.io/idlecycles/#c2xvdygyJTJDJTIwY2F0KCdjeWFuJyUyQyUyMCdtYWdlbnRhJykp)
- [stack](https://felixroos.github.io/idlecycles/#ZmFzdCg0JTJDJTIwc2VxKCdjeWFuJyUyQyUyMCclMjMwMDAwMDA1MCclMkMlMjBzdGFjaygnY3lhbiclMkMlMjAnJTIzMDAwMDAwNTAnKSkp)
- [patternified arg](https://felixroos.github.io/idlecycles/#c2VxKCdjeWFuJyUyQydtYWdlbnRhJyUyQyd5ZWxsb3cnKSUwQS5mYXN0KGNhdCgxJTJDMiUyQzMlMkM0JTJDNSUyQzYlMkM3JTJDOCkp)

## Tidal from Scratch

I've created this repo to understand the core of tidal at a deeper level by re-implementing its core logic.
This part of the README will document my progress and could be helpful for people trying to understand tidal.
Disclaimer: This is not tidal, so take it with a grain of salt.

## Chapter 1: Patterns as Functions of Time

The core primitive of tidal are Patterns, which are essentially functions of time:

```ts
let silence = (a, b) => [];
silence(0, 1); // []
silence(42, 79); // []
```

The above example is the most simple pattern I can think of: silence. It will always return an empty Array. The inputs `a` and `b` are the timespan we want to know about.

### We love repetition

The above example doesn't really do much, so let's implement another pattern that repeats the same thing over and over:

```js
let repeat = (a, b) => {
  let bag = [];
  while (a < b) {
    bag.push({ a, b: a + 1, value: "X" });
    a++;
  }
  return bag;
};
repeat(0, 1); // [{a: 0, b: 1, value: 'X'}]
repeat(0, 2); // [{a: 0, b: 1, value: 'X'}, {a: 1, b: 2, value: 'X'}]
repeat(1, 3); // [{a: 1, b: 2, value: 'X'}, {a: 2, b: 3, value: 'X'}]
```

This function will give us one object per whole number within the given time span.
The object contains the timespan during it is active + some value.

### Higher Order Functions

Ideally, we'd want to choose the value to repeat, so let's make it a higher order function:

```js
let repeat = (value) => (a, b) => {
  let bag = [];
  while (a < b) {
    bag.push({ a, b: a + 1, value });
    a++;
  }
  return bag;
};
let snakes = repeat("snake");
let dogs = repeat("dog");
snakes(0, 2); // [{a: 0, b: 1, value: 'snake'},{a: 1, b: 2, value: 'snake'}]
dogs(1, 3); // [{a: 1, b: 2, value: 'dog'}, {a: 2, b: 3, value: 'dog'}]
```

Higher order just means we have a function that returns a function. The outer function takes a value and returns the inner function, which is a Pattern.

### It's Hap-pening

Our returned object could be expressed as a typescript type like this:

```ts
declare type Hap<T> = { a: number; b: number; value: T };
```

With these `Hap`'s, we can now define a Pattern like this:

```ts
declare type Pattern<T> = (a: number, b: number) => Hap<T>[];
```

Note that these typings are only a simplified version of what strudel or tidal use, but it's enough to get started thinking about Patterns.

### Conc-_cat_-enation

Our repeat function is still a bit boring.. How about switching between different values?

```js
let cat =
  (...values) =>
  (a, b) => {
    let bag = [];
    while (a < b) {
      const value = a % values.length;
      bag.push({ a, b: a + 1, value });
      a++;
    }
    return bag;
  };
let pat = cat("snake", "dog");
pat(0, 2); // [{a: 0, b: 1, value: 'snake'},{a: 1, b: 2, value: 'dog'}]
pat(1, 3); // [{a: 1, b: 2, value: 'dog'}, {a: 2, b: 3, value: 'snake'}]
```

The above pattern gives us an infinite timeline of snakes and dogs!

The implementation looks very similar to `repeat`, maybe we should generalize this loop:

```js
let cycle = (callback) => (a, b) => {
  let bag = [];
  while (a < b) {
    bag = bag.concat(callback(a, a + 1));
    a++;
  }
  return bag;
};

let repeat = (value) => cycle((a, b) => [{ a, b, value }]);

let cat = (...values) =>
  cycle((a, b) => {
    let value = values[a % values.length];
    return [{ a, b, value }];
  });
```

With this, we could implement `repeat` and `cat` much more concise!

### Patterns as Functions of Space

So far, I've talked about `a` and `b` as a _time_ span, but that's just a metaphor, because tidal is used for music.
But it's actually just a numerical range, so nothing stops us from interpreting Patterns as functions of space:

![cat](./img/cat.png)

In this other kind of cat picture, we see each whole number cycle represented as a line, and the value of each `Hap` is interpreted as a color.

From now on, I'll use this type of visualization, as it makes more sense in a written guide.

### Chapter 1 REPL

This is the end of the first chapter!
I've decided to create a separate HTML file with the code for each chapter including a minimal REPL for you to play around with.

Open [chapter1.html](https://felixroos.github.io/idlecycles/learn/chapter1.html) to play with `cat` and `repeat`. You can evaluate the code with `ctrl+enter` to see the visualization in action!

Here are some examples:

- [cat](https://felixroos.github.io/idlecycles/learn/chapter1.html#Y2F0KCdjeWFuJyUyQyUyMCdtYWdlbnRhJyUyQyUyMCd5ZWxsb3cnKQ==)
- [repeat](https://felixroos.github.io/idlecycles/learn/chapter1.html#cmVwZWF0KCdjeWFuJyk=)

This might still not be very exciting, but we're on our way!

## Chapter 2: adding more functions

In this chapter we will take the previous primitives and create new useful functions.

### slow and fast

One interesting property about Patterns as functions of time (or space) is that we can edit the time span from the outside:

```js
let fast = (factor, pat) => (a, b) =>
  pat(
    a * factor, // scale up timespan
    b * factor
  ).map((hap) => ({
    a: hap.a / factor, // scale down hap timespan
    b: hap.b / factor,
    value: hap.value,
  }));

fast(5, cat("cyan", "magenta", "yellow"));
```

This will have the effect of scaling the time axis of our Haps. This is the visual representation:

![fast](./img/fast.png)

The trick is to first request scale up our timespan and then scale down the time span of each Hap by the given factor.

We can implement the opposite version of the function like this:

```js
let slow = (factor, pat) => fast(1 / factor, pat);
```

Example:

![slow](./img/slow.png)

The fact that there are gray areas is due to the way the visual is implemented. Each Hap gets one rectangle whose width is determined by its timespan duration. In the above case, each colored rectangle is 2 screens wide, so you won't see half of it.
We could theoretically implement a split logic, where rectangles that are longer than one screen would be split and moved to the next line, but i guess this is fine for now..

### seq

Another useful function can be derived by combining `cat` and `fast`:

```js
let seq = (...values) => fast(values.length, cat(...values));
```

This function will squeeze its given values into a single cycle:

![seq](./img/seq.png)

Here we can already see a cool property of functional programming: If we have a set of base functions, we can combine them declaratively to create new functions. You can think of it like crafting new materials from a set of raw materials.

### stack

The `stack` function will place all values in the same timespan:

```js
let stack = (...values) =>
  cycle((a, b) => values.map((value) => ({ a, b, value })));
```

Visually, we're in 2D, so stacked rectangles will end up in the same place.
This will only have a visual effect when the stacked colors are transparent:

![stack](./img/stack.png)

### Patterns of Patterns

Our standard library is growing, but we're still missing an important property of Patterns: the ability to nest Patterns inside of each other. Here's an example:

```js
cat("cyan", seq("magenta", "yellow"));
```

If we do that, we only get cyan, because the second input of `cat` is not a color (and the canvas API will resort to the last working color when drawing the rect of the second input).

We can fix it like this:

```js
let cat = (...values) =>
  cycle((a, b) => {
    let value = values[a % values.length];
    // if we have a function, we'll assume it's a Pattern function
    if (typeof value === "function") {
      return value(a, b);
    }
    // if it's not a function we'll take it as before
    return [{ a, b, value }];
  });
```

Now if a value is a function, we'll call it with the current cycle, assuming it's a Pattern function, which allows us to nest:

![catseq](./img/catseq.png)

And magically, we can nest Patterns as deep as we like:

![nesting](./img/nesting.png)

### Chapter 2 REPL

This is the end of the second chapter!

Open [chapter2.html](https://felixroos.github.io/idlecycles/learn/chapter2.html) to play with all the new functions. You can evaluate the code with `ctrl+enter` to see the visualization in action!

With these new functions, we can already create some non-trivial patterns, but the syntax is still a bit unwieldy, as it's difficult to keep track of all the parens when nesting.. Luckily, there is a way out, which we'll implement in the next chapter!

Examples:

- [fast](https://felixroos.github.io/idlecycles/learn/chapter2.html#ZmFzdCg1JTJDJTIwY2F0KCdjeWFuJyUyQyUyMCdtYWdlbnRhJyUyQyUyMCd5ZWxsb3cnKSk=)
- [slow](https://felixroos.github.io/idlecycles/learn/chapter2.html#c2xvdygyJTJDJTIwY2F0KCdjeWFuJyUyQyUyMCdtYWdlbnRhJyUyQyUyMCd5ZWxsb3cnKSk=)
- [seq](https://felixroos.github.io/idlecycles/learn/chapter2.html#c2VxKCdjeWFuJyUyQyUyMCdtYWdlbnRhJyUyQyUyMCd5ZWxsb3cnKQ==)
- [stack](https://felixroos.github.io/idlecycles/learn/chapter2.html#c3RhY2soJyUyM2ZmZmYwMDUwJyUyQyUyMCclMjMwMGZmZmY1MCcp)
- [cat with nested seq](https://felixroos.github.io/idlecycles/learn/chapter2.html#Y2F0KCdjeWFuJyUyQyUyMHNlcSgnbWFnZW50YSclMkMlMjAneWVsbG93Jykp)
- [deeper nesting](https://felixroos.github.io/idlecycles/learn/chapter2.html#ZmFzdCgzJTJDJTBBJTIwY2F0KCUwQSUyMCUyMHNlcSglMjJjeWFuJTIyJTJDJTIwc2VxKCUyMm1hZ2VudGElMjIlMkMlMjAlMjJ5ZWxsb3clMjIpKSUyQyUwQSUyMCUyMHNlcSglMEElMjAlMjAlMjAlMjAlMjIlMjMwMGZmZmY1MCUyMiUyQyUwQSUyMCUyMCUyMCUyMCUyMiUyM2ZmZmYwMDUwJTIyJTJDJTBBJTIwJTIwJTIwJTIwc3RhY2soJTIyJTIzMDBmZmZmNTAlMjIlMkMlMjIlMjNmZmZmMDA1MCUyMiklMEElMjAlMjApJTBBJTIwKSUwQSk=)

## Chapter 3: Language Improvements

Let's improve the language a bit. In strudel you can write:

```js
cat("cyan", "magenta", "yellow").fast(2);
// is equivalent to:
fast(2, cat("cyan", "magenta", "yellow"));
```

In our little implementation, we can only write the second variant.
To get to the first variant, `cat` needs to return an object, where `fast` is a property.

### Pattern as a class

We can refactor our existing code by introducing the `Pattern` class.

```js
class Pattern {
  constructor(query) {
    this.query = query;
  }
}
// Pattern creation shortcut:
let P = (q) => new Pattern(q);

Pattern.prototype.fast = function (speed) {
  return fast(speed, this);
};

Pattern.prototype.slow = function (speed) {
  return slow(speed, this);
};
```

This still won't work, because `fast` itself needs to return a `Pattern` instance + handle a `Pattern` as its input:

```js
let fast = (factor, pat) =>
  // the P(...) wraps our query function in a Pattern
  // + pat is now a Pattern, which has to queried by calling pat.query
  P((a, b) =>
    pat.query(a * factor, b * factor).map((hap) => ({
      a: hap.a / factor,
      b: hap.b / factor,
      value: hap.value,
    }))
  );
```

We're almost there, now the last refactoring to do is to make sure all the other functions return a `Pattern` instance, so we can call `.fast` and `.slow` on them.
Luckily, our `cycle` function is the only thing we need to touch:

```js
let cycle = (callback) =>
  P((a, b) => {
    // ^ the P(...) is new
    let bag = [];
    while (a < b) {
      bag = bag.concat(callback(a, a + 1));
      a++;
    }
    return bag;
  });
```

Finally, we can do:

![fastmethod](./img/fastmethod.png)

### More Meta Programming

We have added our `fast` and `slow` methods to a `Pattern` by defining a function on `Pattern.prototype`.
Because we want to add many more functions that transform a `Pattern`, we can add a helper function to simplify that:

```js
let register = (name, fn) => {
  Pattern.prototype[name] = function (...args) {
    return fn(args, this);
  };
  return fn;
};

let fast = register("fast", (factor, pat) => /* ... */);
let slow = register("slow", (factor, pat) => fast(1 / factor, pat));
```

This paves the way for a lot more functions to come!

### lastOf / firstOf

Last but not least, let's add another set of functions:

```js
let firstOf = register("firstOf", (n, fn, pat) =>
  cycle((a, b) => (a % n === 0 ? fn(pat) : pat).query(a, b))
);
let lastOf = register("lastOf", (n, fn, pat) =>
  cycle((a, b) => (a % n === n - 1 ? fn(pat) : pat).query(a, b))
);
```

This allows us to conditionally apply a function every n cycles:

![lastOf](./img/lastOf.png)

## To be continued
