Node.js Tips
======

Some tips to writing _coherent_ and _concise_ Javascript that keeps everyone sane!

### Always use _either_ synchronous or asynchronous in functions. Not both.

```js
// Bad
function wtf(a, b, cb) {
	if (a === 10) {
		cb(null, a + b);
	} else {
		someAsyncFunc(a, b, cb);
	}
}

// Good
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

Synchronous functions should always use `return` and avoid `callbacks` to be called upon completion.

```js
function sync(a, b) {
	return a + b;
}
```

Asynchronous should rely on either `Promise` or `callbacks`.

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

`async` library is only meant to use with functions that are truly asynchronous. Do not use it with synchronous ones!
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

### Return earlier on the function to make code readable

```js
function funcA() {
	if (condition) {
		return;
	}

	// a lot of code....
}
```
