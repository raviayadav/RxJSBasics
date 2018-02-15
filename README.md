# RxJS Basics

## What and Why?
* Basic Js click event
```js
const button = document.querySelector('button');
button.addEventListener('click', (event) => {
  console.log(event);
})
```
* Creating an Observable from Event
```js
Rx.Observable.fromEvent(button, 'click')
  .subscribe(event => console.log(event));
```
* observable makes the handling of such events more powerful through its operators.
* If we want to only react to events once a second, we can code like
```js
Rx.Observable.fromEvent(button, 'click')
  .throttleTime(1000)
  .subscribe(event => console.log(event));
```
* If we want to only subscribe on some part of the data, say I would only want clientY(y-axis) of the button always
* Only eventY will get to the next chained value as we take all values of eventY in array and then subscribe that Observable
```js
Rx.Observable.fromEvent(button, 'click')
  .throttleTime(1000)
  .map(event => event.clientY) 
  .subscribe(data => console.log(data)); // Output the clientY everytime
```
* Hence RxJS gives us the reactive programming which can ease our work.

## Observables, Observers and Subscriptions

* **Observerables** are basically wrappers around a source of data which is typically a stream of values. 
* Observables are mostly used for Asynchronous data wrapping but can equally be used for wrapping sync source of data.
* So, we have a stream of data of possible multiple async values and we want to do something when a new value occurs. That is the job of an **Observer** . 
* The job of an Observer is to execute a peice of code, whenever a new value occurs or an error is reported or the Observable reports that it is completed.
* So we need to connect the observer to the Observable to be able to do all that stuff. We do that connection by using **Subsciption** which basically means with one method, the subscribe method we tell the Observable, our wrapper around the stream of values that someone is caring about these values, someone is listening on these values(the observer).
* The Observable implements three methods. ` next() error() complete()`
* The next() method will be called whenever a new value is emitted.
* error() will be called whenever theres an error.
* complete() will be called whenever there are no more values left in the stream. Some streams may never end.
* How does the observable know that is should call next error or complete ? This is done by a contract between the Observable and Observer signed through subscription. The Observable knows that it can fire next, error or complete on an Observer and the Observer knows that the Observable will only fire one of these three methods. So we can easily implement them on the Observer and react whenever they are fired.  
