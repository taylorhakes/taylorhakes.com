title: Testing Promise Code
permalink: testing-promise-code
id: 5
updated: '2015-08-20 19:16:59'
date: 2015-08-18 23:40:53
tags:
---

![thumb image-post mask](/content/images/2015/08/promise.png)
If you have ever tried to write tests for Promise based code, you may have realized it's not exactly straight forward. It has definitely become easier over the last couple years. Newer versions of mocha let you return a Promise from a test and it automatically waits for the result. This is what code looks like normally.


<pre><code class="javascript">// Some Promise code
function promiseFunc() {
  return Promise.resolve('result');
}

// The test
it('Promise Test', function(done) {
  promiseFunc().then(function(result) {
    expect(result).toBe('result');
  }).then(done, done);
});
</code></pre>

And with new Mocha versions
<pre><code class="javascript">it('Promise Test', function() {
  return promiseFunc().then(function(result) {
    expect(result).toBe('result');
  });
});
</code></pre>

This removes the need for the `done()` function. It figures out the function returns a Promise and waits correctly. This is pretty good, but there are a few issues for me. Depending on your Promise implementation, async tests can add additional time to your unit tests, especially if you use a lot of Promises. Another issue is stack traces. In async code, you lose some of your stack trace and it can be difficult to see what caused an error. And lastly, I would prefer not to nest my expects. It makes the code harder to follow (the following code in upcoming ES7 will solve the nesting issue)

<pre><code class="javascript">it('Promise Test', async function() {
  const result = await promiseFunc();
  expect(result).toBe('result');
});
</code></pre>

Ideally it would be nice if we could write tests like this.
<pre><code class="javascript">it('Promise Test', function() {
  var result = promiseFunc();
  expect(result).toBe('result');
});
</code></pre>
It would be nice if we could make Promises be synchronous. If you know the Promise spec, you probably are shaking your head. "Promises are async by default!" That is true, but what if we could make them act pseudo async (patent pending).

Here is how it could work
<pre><code class="javascript">it('Promise Test', function() {
	var result;
	promiseFunc().then(function(promResult) {
		result = promResult;
	});

	// Need to make this work
	Promise.runAll();

	expect(result).toBe('result');
});
</code></pre>

In the `runAll` function above, we will execute all the async callbacks synchronously. To the code being tested, it will appear async. So we shouldn't have to worry about a `.then` being called at the wrong time. To the test, it will be completely sychronous.

In order to do this we first need to mock all Promises used by the code. Basically, replace the Promise implementation with a mock. The second step is to hook into the async portion of the code and make it be synchronous.

I happen to have created a Promise library a few months back: [promise-polyfill](https://github.com/taylorhakes/promise-polyfill). I already have a way to change the async callback implementation as part of the API. It was used for switching between `setImmediate`, `process.nextTick`, `setTimeout`, etc. for executing async callbacks.

All I have to do is use that hook to queue the async callbacks and execute them in the future. The code looks like this.

<pre><code class="javascript">// Queue of waiting callbacks
var waitingCallbacks = [];

// Update the immediate function to push to queue
PromiseMock._setImmediateFn(function mockImmediateFn(fn) {
	waitingCallbacks.push(fn);
}
</code></pre>
Then our `runAll` function can just take each function in the queue and execute them
<pre><code class="javascript">// Execute all pending Promise callbacks
PromiseMock.runAll = function runAll() {
	while(PromiseMock.waiting.length > 0) {
		waitingCallbacks.pop()();
	}
};
</code></pre>

Finally, we can remove the boilerplate of retrieving the value from `.then` and calling `runAll()` by creating this function.
<pre><code class="javascript">PromiseMock.getResult = function result(promise) {
	var result, error;
	promise.then(function(promResult) {
		result = promResult;
	}, function(promError) {
		error = promError;
	});
	PromiseMock.runAll();
	if (error) {
		throw error;
	}
	return result;
};
</code></pre>
And it can be used like this.
<pre><code class="javascript">it('Promise Test', function() {
  var result = PromiseMock.getResult(promiseFunc());
  expect(result).toBe('result');
});
</code></pre>

The full code with the [promise-mock](https://github.com/taylorhakes/promise-mock) would be:
<pre><code class="javascript">var PromiseMock = require('promise-mock');

// Some Promise code
function promiseFunc() {
  return Promise.resolve('result');
}

describe('Promise Group', function() {
  beforeEach(function() {
    PromiseMock.install();
  });
  beforeEach(function() {
    PromiseMock.uninstall();
  });
  it('Promise Test', function() {
    var result = PromiseMock.getResult(promiseFunc());
    expect(result).toBe('result');
  });
});

</code></pre>

The library currently only works with `Promise` in the global scope. The library also assumes that all Promise code returns immediately. If you have some sort of setTimeout, you will have remedy that as well.

<pre><code class="javascript">it('Promise Test', function() {
  var prom = promiseFunc();
  
  // Remedy setTimeout synchronously, etc
  jasmine.clock().tick(101);

  var result = PromiseMock.getResult(prom);
  expect(result).toBe('result');
});
</code></pre>

For more information, check out [promise-mock](https://github.com/taylorhakes/promise-mock)
