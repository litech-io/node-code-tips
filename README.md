Node.js Tips
======

Some tips to writing _coherent_ and _concise_ Javascript that keeps everyone sane!

> Always use _either_ synchronous or asynchronous in functions. Not both.

```js
// Bad
function wtf(a, b, cb) {
	cb(null, a + b);
}


// Good
function sync(a, b) {
  return a + b;
}

function async(a, b, cb) {
  process.nextTick(function() {
  	cb(null, a + b);
  });
}
```

> When using callback, first argument is _always_ an error object

```js
function test(cb) {
	cb(err, args); // good
	cb(args);  // bad
}
```

> Bad usage of `async` library

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
