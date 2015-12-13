title: A Few Javascript Performance Tips
permalink: a-few-javascript-performance-tips
id: 4
updated: '2015-08-20 21:43:21'
date: 2015-08-18 23:27:03
tags:
---

![thumb image-post mask](/content/images/2015/08/speed.jpg)
Recently, I've discovered some simple tricks to get my Javascript code to execute faster.  This is not a full list, just some that I use fairly often. If anyone has others, please post them in the comments. It would be great to compile a large list.

Before I begin, I would like to emphasize a phrase that every developer should know. "Premature optimization is the root all evil." Optimization can only get you so far. Good programming practices and code structure are more important. With that, let's get started.
<h3>Use for loops instead of Array.forEach, for.. in, jQuery.each etc.</h3>
<img alt="" src="http://assets.tumblr.com/assets/scripts/vendor/tiny_mce_3_5_10/themes/advanced/img/trans.gif" />
<pre><code class="javascript">var arr = Array(10000).join('x').split('');
for (var i = 0, len = arr.length; i &lt; len; i++) {
   ..some code on the array
}</code></pre>
instead of
<pre><code class="javascript">arr.forEach(function (item, i) {
    ..some code on the array
});</code></pre>
or
<pre><code class="javascript">for (var i in arr) {
    if(arr.hasOwnProperty(i)) {
        ..some code on the array
    }
}</code></pre>
In general, a for loop will execute 10x faster than Array.forEach or for..in. Here is the <a href="http://jsperf.com/array-for-vs-foreach/3" target="_blank">JSPerf comparison</a>
<h3>Don't create functions in loops or commonly used code</h3>
Each function carries memory overhead. Define functions that are commonly used. Only use anonymous function when necessary.
<pre><code class="javascript">for( var i = 0; i &lt; 100; i++) {
    setTimeout(function() { 
        .. do something 
    }, 10);
}</code></pre>
instead do this
<pre><code class="javascript">function repeatedFn() {
    .. do something 
}
for( var i = 0; i &lt; 100; i++) {
    setTimeout(repeatedFn, 10);
}</code></pre>
<h3>Avoid unnecessary functions calls / closures</h3>
Even if you aren't using anonymous functions. All functions still have overhead. Don't create unnecessary functions.

For instance, don't make an add function
<pre><code class="javascript">function add(a, b) {
    return a + b;
}</code></pre>
Check out this <a href="http://jsperf.com/built-in-add-vs-function-add" target="_blank">JSPerf comparison</a>
<h3>Use vanilla javascript when possible</h3>
Libraries/Frameworks can be great for code organization, cross-browser compatibility, etc. Vanilla javascript is much faster in certain circumstances. For instance, I often see people use jQuery to call the click handler on a DOM element.
<pre><code class="javascript">$testDiv.trigger('click') or $testDiv.triggerHandler('click');</code></pre>
the following is cleaner and significantly faster
<pre><code class="javascript">onClickTest();</code></pre>
Check out this <a href="http://jsperf.com/jquery-trigger-vs-function-call" target="_blank">JSPerf comparison</a>