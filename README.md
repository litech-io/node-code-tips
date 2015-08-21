Node.js Tips
======

Writing code is easy. Writing clear, concise and coherent code is not,
especially in Javascript. Here are some tips that can help developers to stay sane.

## Linting

When collaborating with other members (organization/open source/etc.),
always use some form of linting before committing code.
Whether it's eslint, jshint etc, doesn't matter. Just use one.

## Style

- 80 characters length
- tabspace: 2 (With js's callback natures, you don't want to start scrolling horizontally)

## Structure

Always have a root file (index.js/app.js/server.js). It gives clear guidance to
readers as to where to begin.

`tests`: always have tests for your code, and put them within the tests directory

`lib`: holds all library (middlewares, and helpers)

## Case Studies

### Always use _either_ synchronous or asynchronous in functions. Not both.

```js
function terrible(a, b, cb) {
	// Don't do this!
	if (a === 10) {
		cb(null, a + b);
	} else {
		someAsyncFunc(a, b, cb);
	}
}

function good(a, b, cb) {
	if (a === 10) {
		process.nextTick(function() {
			cb(null, a + b);
		});
	} else {
		someAsyncFunc(a, b, cb);
	}
}
```

### Consistency in functions returns

Synchronous functions should always use `return` and avoid `callbacks` to be
called upon completion.

```js
function sync(a, b) {
	return a + b;
}
```

Asynchronous should rely on either `Promise` or `callback`.

```js
function async(a, b, callback) {
	process.nextTick(function() {
		callback(null, a, b);
	});
}

function promisified(a, b) {
	return Promise.resolve(a + b);
}
```

### When using callback, first argument is _always_ an error object

```js
function test(cb) {
	cb(err, args);
}
```

### Bad usage of `async` library

`async` library is only meant to use with functions that are truly asynchronous.
Do not use it with synchronous ones!
(Reference: [#75](https://github.com/caolan/async/issues/75))

```js
async.map(arr, function(item, callback) {
	// Don't do this!
	callback(null, item);
}, function() {
	console.log('sad');
});

// Good
async.map(arr, function(item, callback) {
	process.nextTick(function() {
		callback(null, item);
	});
}, function() {
	console.log('happy now')
});
```

### Use `lodash` for object manipulations

[`lodash`](https://lodash.com/) is a utility library that can be used for
`object` and `array`. Do not use the corresponding [`async`](https://github.com/caolan/async)
(map/every/etc.), as it's meant for something totally different.

```js
var _ = require('lodash');

_.each(['a','b'], function(item) {
	console.log(item);
});
```

### Return earlier on the function to make code readable

`return`-ing earlier allows a better coding style with clear and concise logical sequence.

```js
function funcA() {
	if (condition) {
		return;
	}

	// a lot of code....
}
```

### Be explict when using callbacks

Always be explicit in function calls, so that everyone would be clear as to
which variables are to be passed around.

```js
// Avoid this!
function(fn) {
	fs.readFile('test.txt', fn);
}

// This would be better
function(fn) {
	fs.readFile('test.txt', function(err, data) {
		fn(err, data);
	});
}
```

## Further reading

I would highly suggest reading the [style guide](https://github.com/felixge/node-style-guide) by [@felixge](https://github.com/felixge).

## Contributions

Should you feel that there are other pet peeves you'd like to share,
feel free to fork it, commit and create a PR.
