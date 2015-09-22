Node.js Tips
======

Writing code is easy. Writing clear, concise and coherent code is not,
especially in Javascript. Here are some tips that can help developers to stay sane.

Writing legible and tidy code help towards having readable code that can greatly improve
how you refactor code later on. Avoid having too much dependencies within each of your modules
(think about testing when doing this)

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

## The Language

Javascript is an extremely "annoying" (yes, in quotes) language. It is a very
small language that can be picked up quickly. However to master it, there are
a lot to learn.

### Interview Question:

Event Loop:

```js
for (var i = 0; i < 10; i++) {
	setImmediate(function() {
		console.log(i);
	});
}
```

Function Binding:

```js
var test = {};

function a() {
	console.log(this);
}

a.bind(test);
```

`apply` vs `call`

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

### Add a `return` even for asynchronous functions

This allows clear signals that the function has been terminated, and avoid
invoking the callback function multiple times. It even reduces the
use of `else` in many case.

```js
function async(fn) {
	return http.get('http://example.com', function() {
		fn();
	});

	console.log('this will not be printed');
}
```

### Be explicit when using callbacks

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

### Understanding variables and functions

```js
var a = function() {};

// vs

function a() {
}
```

The former is assigning an anonymous function to a variable `a`, while latter is defining a function named a. The biggest difference between the two relies on `variable hoisting`

### Naming functions and closures

Whilst this might seems unnecessary, it definitely helps when you want to see the better stack trace.

```js
var a = function a() {
	throw new Error('e');
};

try {
	a();
} catch(e) {
	console.log(e);
}

```

### Avoiding exporting functions

This doesn't mean that it's _wrong_ to export a function. It really just means that a function isn't ideal to be exported, if it doesn't do anything computational.

node.js, by default, caches the output of the `module.exports` but when it exports a function, that function is being cached, not the output.

```js
module.exports = function() {
	return {
		a: 'b'
	};
};

// vs.

module.exports = {
	a: 'b'
};
```

### Using Promises in a smart way.

When you deal with promises, you just need to return a promise.

```js
var Promise = require('bluebird');

function test() {
	return Promise.resolve();
}

// The return of this function will be another promise too. You do not need to wrap a() within a `new promise`
function a() {
	return test().then(function() {
		return Promise.resolve();
	});
}
```

## Testing

Writing test should always be done. It will greatly help you in the long run.
When writing unit test, I usually use BDD (Behavioural) with mocha, and have istanbul for code coverage.

```js
describe('as a user', function() {
  it('should be able to retrieve expected data', function() {
    // can pass an optional `done` for asynchronous tests
  });
});
```

Think about testing when you code. It can help towards having a good coverage as well as
concise functions.

## Further reading

I would highly suggest reading the [style guide](https://github.com/felixge/node-style-guide) by [@felixge](https://github.com/felixge).

## Contributions

Should you feel that there are other pet peeves you'd like to share,
feel free to fork it, commit and create a PR.
