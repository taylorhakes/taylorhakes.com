title: Creating a clock with setInterval
permalink: creating-a-clock-with-setinterval
id: 3
date: 2015-08-03
tags:
---

![thumb image-post mask](/content/images/2015/08/time-1.jpg)
If you have ever created a Javascript clock, you may have noticed some issues with using setInterval to update the time. The clock ticks at irregular intervals and can get farther and farther from the correct time.

Below is a possible first attempt:

<pre><code class="html">&lt;div id="clock"&gt;&lt;/div&gt;
</code></pre>

<pre><code class="javascript">var date = Date.now(),
  second = 1000;

function pad(num) {
  return ('0' + num).slice(-2);
}

function updateClock() {
  var clockEl = document.getElementById('clock'),
    dateObj;
  date += second;
  dateObj = new Date(date);
  clockEl.innerHTML = pad(dateObj.getHours()) + ':' + pad(dateObj.getMinutes()) + ':' + pad(dateObj.getSeconds());
}

setInterval(updateClock, second);
</code></pre>

<iframe width="100%" height="100" src="https://jsfiddle.net/skvg2jvz/embedded/result%2Cjs%2Chtml/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

If you check back on this clock after a few minutes, you will notice that the time gets off. Let's try something else. How about we check the time/date on each tick and render the correct time.

<pre><code class="javascript">var second = 1000;

function pad(num) {
  return ('0' + num).slice(-2);
}

function updateClock() {
  var clockEl = document.getElementById('clock'),
    dateObj = new Date();
  clockEl.innerHTML = pad(dateObj.getHours()) + ':' + pad(dateObj.getMinutes()) + ':' + pad(dateObj.getSeconds());
}

setInterval(updateClock, second);
</code></pre>

<iframe width="100%" height="100" src="https://jsfiddle.net/skvg2jvz/1/embedded/result%2Cjs%2Chtml/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

That works a lot better, the clock is no longer getting off the original time. But, if you watch the clock closely, you will see it occasionally skip a second and it is often not changing at the exact same time as the system clock.

That happens because setInterval does not always call the function every second. It executes at (1 second) + (time Chrome is doing other things). Read my previous blog post on setInterval to get more details <a href="https://taylorhakes.com/2013/11/10/understanding-setinterval/">Understanding setInterval</a> . It also starts at a random time. It may be at the beginning or the end of the second.

Here is an output of the rendering times.
<iframe width="100%" height="300" src="https://jsfiddle.net/bv0bpLqg/embedded/result%2Cjs%2Chtml/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

As you can see the last 3 numbers are increasing each tick. That is not ideal because it means the clock is becoming more and more incorrect. The next step is to take into account the rendering time and try to render on the exact second. This will happen when Date.now() has 000 at the end (because it is in milliseconds). We can use the % operator to calculate the next time. Let's forget setInterval because we need more fine grained control over the next tick time.

<pre><code class="javascript">var second = 1000;

function pad(num) {
  return ('0' + num).slice(-2);
}

function updateClock() {
  var clockEl = document.getElementById('clock'),
    dateObj = new Date();
  clockEl.innerHTML = pad(dateObj.getHours()) + ':' + pad(dateObj.getMinutes()) + ':' + pad(dateObj.getSeconds());
}

function clockInterval(fn) {
    var time = second - (Date.now() % second);

    setTimeout(function() {
        fn();
        clockInterval(fn);
    }, time);
}

clockTimer(updateClock);
</code></pre>

<iframe width="100%" height="300" src="https://jsfiddle.net/qnqLzqtL/embedded/result%2Cjs%2Chtml/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

Now we have a clock that shows the right time and ticks at the correct time. It still isn't perfect, but it's as good as we can do while not blocking the event loop.

If you want to use this code, I created a <a href="https://github.com/taylorhakes/clock-interval" target="_blank">github repo</a> and <a href="https://www.npmjs.com/package/clock-interval" target="_blank">a npm module</a>. If you have ideas to make this better, please post in the comments.