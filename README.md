SimpleWorker
============
>Inline web workers + promises

SimpleWorker.js is a utility for defining web worker tasks inline. It lets you write a function which is executed in the context of a new Web Worker. It uses a Promise ([powered by Q](https://github.com/kriskowal/q)) to represent the result of the asynchronous background computation.

A simple example is:

    var add = SimpleWorker(function (a, b) {
      return a + b;
    });
    
    var promise = add(1, 2);
    
    promise.then(function (result) {
      alert(result); // 3
    });
    
SimpleWorkers embraces promises throughout. If the result within your web worker is also asynchronous, you can return a promise-like object from your worker and it will automatically post back the result to the main thread. 

    var add = SimpleWorker(function (a, b) {
      var deferred = Q.defer();

      setTimeout(function () {
        deferred.resolve(a + b);
      }, 3000);

      return deferred.promise;
    }, ['https://raw.github.com/kriskowal/q/master/q.js']);
    
This particular example loads Q.js to provide promises from within the worker. In general, you can use this to take advantage of other third party libraries that use promises. This always isn't realistic - if you need to return a result asynchronously, you can do so by calling the `postResult(result)` function from within your worker.

## The Details ##
SimpleWorker tasks are executed within the context of a new WebWorker. This has implications and you should use the following guidelines to decide whether you should use it:

* Your task function cannot access any resources from your main thread. Any references to variables from the containing scope will be unresolved and throw an error.
* There is an overhead cost associated with spinning up a new WebWorker. Use them only when you have a task which is computationally expensive, has the potential to block the UI thread, and which you are able to isolate.
* You can pass arguments into your worker and get back a result. However, due to the limitation of the "Structured Cloning" algorithm which implements serialization in message passing between Web Workers and the main thread, you cannot pass in or return back Error objects, DOM elements, or functions. [Read more](http://www.whatwg.org/specs/web-apps/current-work/multipage/common-dom-interfaces.html#safe-passing-of-structured-data).
