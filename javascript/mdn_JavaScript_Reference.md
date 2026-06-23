https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference

- [1. JavaScript execution model](#1-javascript-execution-model)
- [2. Strict mode](#2-strict-mode)

under misc -

# 1. JavaScript execution model

...

**Job queue and event loop**

Every time, the agent pulls a job from the queue and executes it. A job is considered completed when the stack is empty; then, the next job is pulled from the queue. HTML event loops split jobs into two categories: tasks and microtasks. 

> [My Note](https://developer.mozilla.org/en-US/docs/Web/API/HTML_DOM_API/Microtask_guide) A microtask is a short function which is executed after the function or program which created it exits and only if the JavaScript execution stack is empty, but before returning control to the event loop. JavaScript **promises** and the **Mutation Observer API** both use the microtask queue to run their callbacks. 
> 
>  tasks get added to the task queue when:
> -  A new JavaScript program or subprogram is executed directly.
> - The user clicks an element. A task is then created and executes all event callbacks.
> - A timeout or interval created with `setTimeout()` or `setInterval()` is reached, causing the corresponding callback to be added to the task queue.
> 
> The oldest runnable task in the task queue will be executed during a single iteration of the event loop. After that, microtasks will be executed until the microtask queue is empty, and then the browser may choose to update rendering. Then the browser moves on to the next iteration of event loop.
> 
> If a microtask adds more microtasks to the queue by calling `queueMicrotask()`, those newly-added microtasks execute before the next task is run.
>
> While there have been tricks available that made it possible to enqueue microtasks in the past (such as by creating a promise that resolves immediately), the addition of the `queueMicrotask()` method adds a standard way.
>
> **When to use microtasks**
>
> Ensuring ordering on conditional use of promises - when promises are used in one clause of an `if...else` statement but not in the other clause
>
> ...
>
> Batching operations - You can also use microtasks to collect multiple requests from various sources into a single batch
> ```js
> let sendMessage = (message) => {
>   messageQueue.push(message);
> 
>   if (messageQueue.length === 1)
>     queueMicrotask(() => {
>       const json = JSON.stringify(messageQueue);
>       messageQueue.length = 0;
>       fetch("url-of-receiver", json);
>     });
> };
> ```



# 2. Strict mode

Eliminates some JavaScript silent errors by changing them to throw errors.

```js
"use strict";
function fun() {return this;}
console.assert(fun() === undefined);
```
