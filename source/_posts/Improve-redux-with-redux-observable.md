title: Improve Redux with redux-observable
permalink: improve-redux-with-redux-observable
date: 2016-09-06 00:00:00
---
Redux is a great way to manage your state in any React application. To learn the basics of Redux, visit the [Redux website](http://redux.js.org/) . 

A very simple Redux application looks something like this
<pre><code>Reducers ──────────────────────────▶ React  (View)
 ▲                                    │
 │                                    │
 │                                    │
 │                                    │
 │                                    ▼
 └─────────────────────────────── Actions
                           (onClick, onSumbit, etc.)
</code></pre>

This setup works for really simple applications, but fails to provide the ability to make API calls or use websockets, timers, etc. Where are you suppose to put all that? Luckily, Redux has a feature called middleware. Redux middleware is a function that has the following signature.

<pre><code class="javascript">store => next => action => undefined
</code></pre>

In ES5 it would a function like this
<pre><code class="javascript">function (store) {
    return function(next) {
        return function(action) {
          // Do something with each action
        };
    }
}
</code></pre>

It sits between the actions and the reducers in the diagram.

<pre><code>Reducers ──────────────────────────▶ React  (View)
 ▲                                    │
 │                                    │
 │                                    │
 │                                    │
 │                                    ▼
middleware ◀─────────────────────── Actions
                           (onClick, onSumbit, etc.)
</code></pre>

In the function definition, the `store` is the instance of the redux store. The `next` is the next middleware in the chain and the `action` is the action sent by the action creater. [Get more info here](http://redux.js.org/docs/advanced/Middleware.html)

Middleware allows you to do some really powerful things. For instance, you can return a `Promise` as an action and then create middleware to wait for the Promise to resolve and send the data as an action. That is how the [redux-promise](https://github.com/acdlite/redux-promise/blob/master/src/index.js) middleware works. Another middleware example is [redux-thunk](https://github.com/gaearon/redux-thunk/blob/master/src/index.js), which is similar to redux-promise, but uses callbacks instead of Promises.

Both of those libraries are great and a lot of redux developers use them, but there are still some problems. How do you cancel actions, or make sure actions are in order? In an autocomplete the user may type multiple keys quickly and API responses can come back in the incorrect order.

Or what about timers? Maybe you want to show a message and have it disappear after 10 seconds, but the user is able to close the message box before the 10 seconds finish. redux-promise and redux-thunk don't work well for those use cases. Some developers write the timers in their React views, but that has issues as well.

There is a better way! [redux-observable](https://github.com/redux-observable/redux-observable) is the better way. redux-observable is a middleware, just like the ones I described earlier, except it uses [RxJS Observables](https://github.com/ReactiveX/rxjs) instead of Promises/callbacks. If you are not familiar with Observables, that is OK. You can see some of the features in this article and then get more details later. Observables are incredibly powerful. They allow you to use all the operations you use on arrays (map, filter, reduce, etc.) on events over time, or in our case, Redux actions over time. When React creates Redux actions, we can change the actions and create new actions.

This is the diagram with redux-observable.

<pre><code>Reducers ──────────────────────────▶ React  (View)
 ▲                                    │
 │                                    │
 │                                    │
 │                                    │
 │                                    ▼
Observables ◀────────────────────── Actions
 ▲      │                    (onClick, onSumbit, etc.)         
 │      ▼                         
  ◀─────
</code></pre>

There is a loop on Observables because Observables can trigger other Observables in a loop. With redux-observable, your actions are just plain javascript objects. Just like the original diagram. They are not functions or Promises like with redux-promise/redux-thunk. You create Observables that interact with the Javascript objects (actions) and do all the heavy lifting of ajax, timers, etc.

Here is what the code for a simple redux-observable `epic` looks like. An `epic` is a function that takes an Observable of Redux actions and returns a new Observable. Let's take
a look at an example.

<pre><code class="javascript">const mapEpic = (action$) =>
    action$.ofType('SOME_ACTION').map((action) => {
        return {
            type: 'ANOTHER_ACTION',
            payload: action.payload
        };
    });
</code></pre>

In the above `epic`, we filter for actions with type `'SOME_ACTION'` using the `.ofType` and then use `.map` (exactly like with arrays) to change the action to `'ANOTHER_ACTION'`. `ofType` and `map` are RxJS operators. A full list can be found on the [RxJS website](A full list is available on the [RxJS website](http://reactivex.io/rxjs/manual/overview.html#operators).

Just like the reducers in Redux, every `epic` will receive all the actions. That is why the `ofType` is necessary. The dollar sign at the end of `action$` is just a naming convention for Observables. It is like the convention for jquery elements `$el`.

The `'ANOTHER_ACTION'` will get passed to the reducers after the original `'SOME_ACTION'`.

Now for something a little bit more powerful. Let's make the autocomplete from earlier work correctly by only rendering the most current ajax call.

<pre><code class="javascript">import { ajax } 'rxjs/observable/dom/ajax';

const apiEpic = (action$) =>
    action$.ofType('AJAX_CALL').switchMap((action) =>
        ajax({url: '/some-data'}).map((result) => {
            const data = JSON.parse(result.response);
            return {
                type: 'AJAX_CALL_RESPONSE',
                payload: data
            };
        })
    )
</code></pre>

The above code uses an `ajax` function from RxJS. It is very similar to the `fetch` API except it returns an Observable instead of a Promise. The `switchMap` function is the important piece. `switchMap` will discard the old request if a new `'AJAX_CALL'` action comes in before the API responds. `switchMap` is another RxJS operator. 

Now lets see how to do the message timer with cancel.

<pre><code class="javascript">import { of } from 'rxjs/observable/of';

const messageEpic = (action$) =>  {
    const cancelMessages$ = action$.ofType('MESSAGE_END');

    return action$.ofType('MESSAGE').mergeMap((action) =>
        of({ type: 'MESSAGE_END' })
            .delay(10000)
            .takeUntil(cancelMessages$);
};

</code></pre>

This `epic` is a little more complicated. We listen for type `'MESSAGE'`. For each one, we create an Observable `of` type `'MESSAGE_END'` and delay it 10000 millis (10 seconds). `takeUntil` means ignore the actions/events if another Observable emits a value. In this case, don't send a `'MESSAGE_END'` if the user sends a `'MESSAGE_END'` before the 10 seconds are done. `mergeMap` is necessary instead of `map` because `of` creates an Observable and we want to merge (or flatten) the Observable. Similar to [flatMap](https://lodash.com/docs/4.15.0#flatMap) with arrays.

Hopefully I have peaked your interest a little. redux-observable has made my UI development a lot easier. To learn more about redux-observable, head over to the [redux-observable github page](https://github.com/redux-observable/redux-observable).