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
    bag = bag.concat(callback(a));
    a++;
  }
  return bag;
};

let repeat = (value) => cycle((a) => [{ a, b: a + 1, value }]);

let cat = (...values) =>
  cycle((a) => {
    let value = values[a % values.length];
    return [{ a, b: a + 1, value }];
  });
```

With this, we could implement `repeat` and `cat` much more concise!

### Patterns as Functions of Space

So far, I've talked about `a` and `b` as a _time_ span, but that's just a metaphor, because tidal is used for music.
But it's actually just a numerical range, so nothing stops us from interpreting Patterns as functions of space:

![cat](./img/cat.png)

In this other kind of cat picture, we see each whole number cycle represented as a line, and the value of each `Hap` is interpreted as a color.

From now on, I'll use this type of visualization, as it makes more sense in a written guide.

### End of Chapter 1

This is the end of the first chapter!
I've decided to create a separate HTML file with the code for each chapter including a minimal REPL for you to play around with.

Open [chapter1.html](./learn/chapter1.html) to play with `cat` and `repeat`. You can evaluate the code with `ctrl+enter` to see the visualization in action!

## To be continued
