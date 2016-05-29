title: Introducing Painless Testing Library
date: 2016-02-16
permalink: introducing-painless-testing-library
tags:
---
[Painless](https://github.com/taylorhakes/painless) is a new javascript testing library. It improves upon other libraries such as Mocha, Jasmine, and Tape. Painless comes bundled with the ability to run your tests, mock any dependencies and assert that your code is working. Here are some of the main issues trying to be solved by painless.
#### Modern Javascript Support
In 2016, most javascript developers are writing some ES6/ES2015 code. Promises, generators, async/await, observables are all very powerful, but can only be harnessed if the testing library supports them. Painless has first class support for these features.
#### Figuring out what is wrong
If you write tests, inevitably some will eventually fail. It is important for the test library to tell you what is wrong as efficiently as possible. Other libraries give you way too much noise. You will often see a stack trace that includes test library code or you will see a badly formatted diff, making it really hard to see what is wrong. Painless tries to only show you the information that is relevant to why your test is failing. 
##### Jasmine output
![test error](/img/jasmine.png)
##### Painless output
![test error](/img/painless.png) 
In addition, painless is just a standard Node process. If you need to use the debugger, you can use all the same tools you are familiar with.
#### Really fast
Time waiting for your tests to finish is time you should be writing code. Painless is very fast out of the box and also has the option to run your tests async, which speeds up network and IO tests. AVA test library has a similar feature, but AVA runs your tests in subprocesses, which makes code harder to debug. Painless uses Node's async nature to run tests while waiting for IO or the network, all in a single thread.
#### Easy to use
Some libraries make you install a bunch of extra dependencies before you can start testing. For instance, Mocha needs an assertion library, a mocking library and a couple other libaries for most use cases. It can be annoying to install 4+ libraries to get started. Mocha is modular, but the trade off makes the library harder to use.
#### Run tests in Node and browsers
Painless supports running tests in Node and browsers. With no code changes you can run all your tests in Node, Chrome, PhantomJS or any other browser.
#### No globals
Painless tests don't depend upon `describe`, `it`, `expect`, etc. to be in the global scope. This allows painless to run as a simple node process `node test.js`.
#### Conclusion
I have been working hard to make [painless](https://github.com/taylorhakes/painless) the best test library. If any of the above features are interesting, check out [painless](https://github.com/taylorhakes/painless). If you would like to see comparisons to other test libraries, [go here](https://github.com/taylorhakes/painless#compared-to-other-libraries-). Even if you don't use painless, keep writing tests.
