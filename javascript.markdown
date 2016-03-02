# Character Codes from Keyboard Listeners 

If I create the following keyboard event listeners for `keydown`,
`keypress`, and `keyup`:

```javascript
window.addEventListener('keydown', function(e) { console.log("Keydown: " + e.charCode + ", " + e.keyCode); });
window.addEventListener('keypress', function(e) { console.log("Keypress: " + e.charCode + ", " + e.keyCode); });
window.addEventListener('keyup', function(e) { console.log("Keyup: " + e.charCode + ", " + e.keyCode); });
```

and then I press `A`, my browser console will read the following:

```
Keydown: 0, 65
Keypress: 65, 65
Keyup: 0, 65
```

and if I then press `a`, my browser console will read:

```
Keydown: 0, 65
Keypress: 97, 97
Keyup: 0, 65
```

The `keypress` event seems to be the way to go. Regardless, there seems to
be quite a bit of [incompatibility and lack of
support](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent#Browser_compatibility)
across browsers for various aspects of the keyboard events.

# Create An Array Containing 1 To N 

Some languages, such as Ruby, have a built in range constraint that makes it
easy to construct an array of values from 1 to N. JavaScript is not one of
those languages. Nevertheless, if you don't mind the aesthetics, you can get
away with something like this:

```javascript
> Array.apply(null, {length: 10}).map(Number.call, Number);
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

That gives us `0` through `9`. To get `1` through `10`, we can tweak it
slightly:

```javascript
> Array.apply(null, {length: 10}).map(Number.call, function(n) {
    return Number(n) + 1;
  });
=> [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

To generalize this, we can replace `10` with `N` and then just expect that
`N` will be defined somewhere:

```javascript
> var N = 10;
=> undefined
> Array.apply(null, {length: N}).map(Number.call, function(n) {
    return Number(n) + 1;
  });
=> [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

[Source](http://stackoverflow.com/a/20066663/535590)

# Enable ES7 Transforms With react-rails 

The [`react-rails`](https://github.com/reactjs/react-rails) gem adds JSX and
ES6 transforms to the asset pipeline.  By using `.js.jsx` and `.es6.jsx`
extensions with relevant files, the asset pipeline will know to make the
appropriate transformation when compiling application assets. ES7 transforms
are not enabled by default, but can be configured. Add the following to the
`config/application.js` file to allow ES7's *class properties* syntax:

```ruby
config.react.jsx_transform_options = {
  optional: ["es7.classProperties"]
}
```

h/t Mike Chau

# Random Cannot Be Seeded 

In JavaScript, you can use `Math.random()` to get a *sorta* random value.

```javascript
> Math.random()
0.5130641541909426
```

Most programming languages give you a method of seeding the random number
generator for determinism. Unfortunately, JavaScript provides no way for
choosing or resetting the seed to `Math.random()`.

[source](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random)

# Splat Arguments To A Function 

Often times you have a function that takes a certain set of arguments. Like
the following `adder` function:

```javascript
var adder = function(a,b,c) {
  return a + b + c;
};
```

But you are left trying to pass in arguments as an array (e.g. `[1,2,3]`).
You want to be able to *splat* the array of arguments so that it matches the
function declaration. This can be done by using `apply`.

```javascript
> adder.apply(undefined, [1,2,3])
6
```

# Throttling A Function Call 

Imagine you have a JavaScript function that makes a request to your server.
Perhaps it is sending user input from a `textarea` to be processed by the
server. You may want to wrap this function in a keyboard event listener so
that you are sure to react immediately to any user input. However, as the
user starts typing away into this text area you may find that way to many
requests are being fired off to the server. The request needs to be
*throttled*.

You can roll your own approach to sufficiently intermittent server calls.
Though, it turns out that [underscore.js](http://underscorejs.org/) comes
with two functions out of the box for this kind of behavior.

- [`throttle`](http://underscorejs.org/#throttle) will give you a function
  that wraps your function in a way that essentially rate-limits it to being
  called at most once every `N` milliseconds.

- [`debounce`](http://underscorejs.org/#debounce), on the other hand, will
  give you a function that only calls your function once `N` milliseconds
  has passed since it was last called.

These are two subtly different approaches to making sure a function gets
called, just not too often.

h/t [Jake Worth](https://twitter.com/jwworth)

# Transforming ES6 and JSX With Babel 6 

With Babel 5, transforming ES6 and JSX into ES5 code was accomplished by
including the `babel-loader`. This would be configured in
`webpack.config.js` with something like the following:

```javascript
module: {
  loaders: [
    {
      test: /\.jsx?$/,
      exclude: /node_modules/,
      loader: 'babel-loader',
    }
  ],
},
```

Now, with Babel 6, the different parts of the loader have been broken out
into separate plugins. These plugins need to be installed

```
$ npm install babel-preset-es2015 babel-preset-react --save-dev
```

and then included as presets

```javascript
module: {
  loaders: [
    {
      test: /\.jsx?$/,
      exclude: /node_modules/,
      loader: 'babel-loader',
      query: {
        presets: ['es2015', 'react']
      },
    }
  ],
},
```

Alternatively, the presets can be specified in the project's `.babelrc` file.

[Source](http://jamesknelson.com/the-six-things-you-need-to-know-about-babel-6/)

# Truthiness of Integer Arrays 

We can consider the truthiness of `[1]` as follows:

```javascript
> [1] == true
=> true
> Boolean(true)
=> true
> Boolean([1])
=> true
```

We can consider the truthiness of `[0]` as follows:

```javascript
> [0] == false
=> true
> Boolean(false)
=> false
> Boolean([0])
=> true
```

The truthiness of `[0]` does not seem to be consistent.

See this [JavaScript Equality Table](https://dorey.github.io/JavaScript-Equality-Table/)
for more details.
