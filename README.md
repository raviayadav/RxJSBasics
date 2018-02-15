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
* The Observer implements three methods. ` next() error() complete()`
* The next() method will be called by the Observable whenever a new value is emitted.
* error() will be called whenever theres an error.
* complete() will be called whenever there are no more values left in the stream. Some streams may never end.
* How does the observable know that is should call next error or complete ? This is done by a contract between the Observable and Observer signed through subscription. The Observable knows that it can fire next, error or complete on an Observer and the Observer knows that the Observable will only fire one of these three methods. So we can easily implement them on the Observer and react whenever they are fired.  
* The following Observable wraps an event. It is never ending as a user can click any time. This Observable has an infinite stream of values(a new value everytime user clicks the button) amd then in the subscribe method we pass an observer. 
```js
Rx.Observable.fromEvent(button, 'click')
  .subscribe(value => console.log(value.clientX);
```
* Subscribe can take either all the three methods or an object which implements all these three methods. Eg.
```js
const button = document.querySelector('button');
const observer = {
  next: function(value) {
        console.log(value.clientX); // mouseevent.clientX
        },
  error: function(error) {
         console.log(error);
        },
  complete: function() {
        console.log(completed);
        }
}
Rx.Observable.fromEvent(button, 'click').subscribe(observer); // will produce the same results as above code
```
* To create an Observable from scratch using create (there are multiple methods to create Observables)
```js
// create has a function that takes an observable
const button = document.querySelector('button');
// button.addEventListener('click', event => {
//   console.log(event);
// });

// Rx.Observable.fromEvent(button, 'click')
//   .subscribe(event => console.log(event));

const observer = {
  next: function(value) {
    console.log(value);
  },
  error: function(error) {
    console.log(error);
  },
  complete: function() {
    console.log('completed');
  }
}

// Rx.Observable.fromEvent(button, 'click')
Rx.Observable.create(function(obs){
  obs.next('Some value');
  obs.next('Good times');
//   obs.error('Error occured');
  obs.next('This code is unreachable as error above');
  obs.complete('If no error, complete will run');
  obs.error('This wont run as complete above');
})
  .subscribe(observer);
```
* In the above code, we are not listening to a click right now, We see the obs.next values in the console immediately because we are subscribed to an observable which takes a function and that function takes an observer(the const observer we defined) which we pass to subscribe `.subscribe(observer)` That takes it to the next method of the const observer we defined and executes it. After an error or complete in the Observable function (obs.error() or obs.complete()) no more code will be run as it will terminate the observable there.
