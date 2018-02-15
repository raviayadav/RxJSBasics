# RxJS Basics

## Creation of an observable
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
