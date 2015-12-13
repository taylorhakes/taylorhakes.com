title: Understanding setInterval
permalink: understanding-setinterval
id: 2
updated: '2015-08-20 21:40:09'
date: 2015-08-18 22:29:16
tags:
---

![thumb image-post mask](/content/images/2015/08/timer.jpg)
If you write a lot of Javascript, inevitably you find the need to use the functions setTimeout or setInterval. setTimeout executes a function after a given number of milliseconds and setInterval executes a function forever at a given interval. You can read more about them here (<a href="https://developer.mozilla.org/en-US/docs/Web/API/Window.setTimeout">setTimeout</a> and <a href="https://developer.mozilla.org/en-US/docs/Web/API/Window.setInterval">setInterval</a>).

Most developers use them, but don't really understand what is going on behind the scenes. An interesting explanation came from <a href="http://ejohn.org/blog/how-javascript-timers-work/">John Resig's blog</a>. I originally took the information as fact without doing my own investigation. About a year ago AppNexus decided to give an interview question on setInterval. My colleague, Sam Mati, and I decided to do some tests on setInterval. Interestingly, we found John's model of setInterval to be wrong.

Here is the important diagram from John's blog post. Notice that setInterval's function is always placed on the event stack at 10ms intervals from the original execution time. If another event takes too much time, only one setInterval function will remain in the queue at a time.

<a href="http://ejohn.org/blog/how-javascript-timers-work/"><img class="alignnone size-full wp-image-35" alt="timers" src="http://i2.wp.com/ejohn.org/files/Timers.png" width="427" height="320" /></a>

We decided to create a simple test. The code executes a setInterval at a 1 second interval and then has code that runs for 1.5 seconds.
<a href="http://jsfiddle.net/2VeYC/">http://jsfiddle.net/2VeYC/</a>
<pre>
<code class="javascript">var start = new Date();

// Mimics a long running call for a number of milliseconds 
function waitMilliseconds(milliseconds) {
  var start = +(new Date());
  while(start + milliseconds > +(new Date())) {}
}

// First setInterval call. console.log the time 
setInterval(function() {
  var current = new Date();
  console.log('setInterval executing: ' + (current - start));
}, 1000);

// Some code that takes a while to run
var current = new Date();
console.log('start long running code: ' + (current - start)); 
waitMilliseconds(1500);
current = new Date();
console.log('end long running code: ' + (current - start));</code>
</pre>
If John's explanation was correct, we should see something like this. The first setInterval would take 1.5 seconds (because of the slow code), the second would take .5 seconds to get back in sync (with the original call) and then after it would continue at 1s intervals.
<pre>
<code class="javascript">start long running code: 0 
end long running code: 1500
setInterval executing: 1501 
setInterval executing: 2000 
setInterval executing: 3000
</code></pre>
The actual results
<pre>
<code class="javascript">start long running code: 0
end long running code: 1503
setInterval executing: 1505
setInterval executing: 2505
setInterval executing: 3507</code>
</pre>
It appears that setInterval executes at the interval based on the last time it was executed. Essentially it behaves equivalent to this code. <a href="http://jsfiddle.net/u8SgN/">http://jsfiddle.net/u8SgN/</a>
<pre>
<code class="javascript">// Recreation of setInterval
function pseudoSetInterval(fn, time /* [, param1, param2, ...]*/) {
    var args = Array.prototype.slice.call(arguments,2);
    setTimeout(function interval() {  
        setTimeout(interval, time);
        fn.apply(window, args);
    }, time);
}</code></pre>
To make sure that was correct, we created another test with only setInterval. We added long running code of 1.5s to setInterval. <a href="http://jsfiddle.net/FQFGM/">http://jsfiddle.net/FQFGM/</a>
<pre>
<code class="javascript">// Get the start of code execution
var start = new Date();

// Mimics a long running call for a number of milliseconds 
function waitMilliseconds(milliseconds) {
  var start = +(new Date());
  while(start + milliseconds > +(new Date())) {}
}

var first = true;
setInterval(function() {
    var current = new Date();
    console.log('setInterval executing: ' + (current - start));

    // Make the first execution take 1.5 seconds
    if (first) {
        waitMilliseconds(1500);
        first = false;
    }
}, 1000);
</code></pre>
Here are the results. It has the same result.
<pre>
<code class="javascript">setInterval executing: 1001
setInterval executing: 2504
setInterval executing: 3505
setInterval executing: 4506
</code></pre>
This result will not impact most javascript developers, but for games, animations, etc. it is important to know exactly how javascript's events work.