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
  obs.complete();
  obs.error('This wont run as complete above');
})
  .subscribe(observer);
```
* In the above code, we are not listening to a click right now, We see the obs.next values in the console immediately because we are subscribed to an observable which takes a function and that function takes an observer(the const observer we defined) which we pass to subscribe `.subscribe(observer)` That takes it to the next method of the const observer we defined and executes it. After an error or complete in the Observable function (obs.error() or obs.complete()) no more code will be run as it will terminate the observable there.
* An async example 
```js
Rx.Observable.create(function(obs){
  obs.next('Some value');
  setTimeout(function() {
      obs.complete();
  },2000);
  obs.next('Good times');
})
  .subscribe(observer); // output -> Some value, Good times..after 2 seconds, completed
```
* Recreating the fromEvent from create 
```js
Rx.Observable.create(function(obs){
  button.onclick = function(event) {
    obs.next(event.clientX);
  }
})
  .subscribe(observer);
```
* Observables like these on which we never call complete are actually dangerous as they can cause memory leaks. Always unsubscibe such observables in the following way. Take your observable in a variable and unsubscribe it at the end.
```js
const subscription = Rx.Observable.create(function(obs){
  button.onclick = function(event) {
    obs.next(event.clientX);
  }
})
  .subscribe(observer);

setTimeout(() => {
  subscription.unsubscribe();
}, 5000);
```

## map() And throtteTime()

* Create an Observable using interval method. This will emit numbers after an interval of milliseconds that is passed by the user.
```js
const Observable = Rx.Observable.interval(1000);
// As no error or complete on ths one cos infinite stream
const Observer = {
  next: function(value) {
    console.log(value);
  }
}
// Map will modify each stream value of Observable
// Throttletime will let only one stream value pass every x milliseconds
// In this case as we are already using interval of 1, we get 3 difference
const Example = Observable
  .map(value => 'Number: ' + value*2)
  .throttleTime(2000)
  .subscribe(Observer);

setTimeout(() => {
  Example.unsubscribe();
}, 20000);
```

## Subject

* Onservables are kind of passive, we may wrap an event or http request but we can't trigger the emission of a new value manually.
* At some point, we might want to emit a value manually from an Observable, like an event emitter. 
* We can do this using a Subject, it inherits from the Observer but here we can call the next method manually to force it to emit a new value.
* Therefore we can have a forcefull or active approach with Subjects.
* A subject is generally used as an event emitter and is subscribed at multiple places.
```js
const subject = new Rx.Subject();

subject.subscribe({
  next: (value) => {
    console.log(value);
  },
  error: (error) => {
    console.log(error);
  },
  complete: () => {
    console.log('completed');
  }
});

subject.subscribe({
  next: value => console.log(value)
});

subject.next('Some peice of data');
// subject.error(error);
subject.complete();
```
## Filter
* Return true or false in a fliter
```js
// Want to emit a number every second in ascending order
const observable = Rx.Observable.interval(1000);
// Subscribe on it and define an observer 
// It is never going to complete
// Use filter to return true or false
observable
  .filter(value => value % 2 === 0)
  .subscribe({
  next : value => console.log(value),
  error: error => console.log('error', error),
});
```
## DebounceTime and distinctUntilChanged

* We use debounce time on say an input to reduce the number of http calls made on each keystroke.
* This is a bit different from throtteTime as the debounceTime only fires after the event is not fired again for the given amount of time. throttle time will skip events which are filtered during the time lapse whereas debounceTime will always run after the end.
* The use case is as follows
* (Debounce) You want to make sure that a user has finished performing an action before carrying out a task e.g wait for a user to finish typing into an input field before sending an ajax request to query the db.
* (Throttle) You want to control the rate at which an event is being fired so as to reduce the amount of time the corresponding task would run e.g you have an autcomplete field that queries your db and you want to trottle rate at which the input event is fired,(to avoid too many request to be sent in a short time).
```js
const input = document.querySelector('input');
const observable = Rx.Observable.fromEvent(input, 'input');
// The event will fire again after a delay of 2 seconds. Taking (not skipping like throttle) all values
observable
  .debounceTime(2000)
  .subscribe({
  next: (event) => console.log(event.target.value)
});

```
* Now if some on types Ravi in input and then backspaces and types Ravi again within the delay of debounceTime, we must name send the same call again. We can acheive this by using distinctValue. If the values are distinct the event will be prevented from firing.
* Always be careful about the ordering of chain methods, distinctUntilChanged if used before deboubcedTime will not work cos there will be no time to check for changes.
```js
const input = document.querySelector('input');

const observable = Rx.Observable.fromEvent(input, 'input');
// The order of chaining is important here, we must check for distinct values only after debounceTime 
observable
  .map(event => event.target.value)
  .debounceTime(2000)
  .distinctUntilChanged()
  .subscribe({
  next: (value) => console.log(value)
});

```

## Scan vs Reduce

* Reduce works same as array.reduce(). It takes in acc, cur value and an initial value and gives the reduced output.
* Scan although almost same as reduce, returns the value of the acc at every step. 
* Reduce to be used only on stream which we know will end at some point.
* Scan can be used on infinite streams as well apart from finite streams.
* Redce at work
```js
const observable = Rx.Observable.of(1, 2, 3, 4, 5);
observable
  .reduce((acc, curr) => {
  return acc + curr;
}, 0)
  .subscribe({
  next: value => console.log(value)
});
```
* The output of the above code will be 15 in the console. It will be a SINGLE value at the end.
* Scan at work
```js
const observable = Rx.Observable.of(1, 2, 3, 4, 5);
observable
  .scan((acc, curr) => {
  return acc + curr;
}, 0)
  .subscribe({
  next: value => console.log(value)
});
```
* The output of the above code will be 1, 3, 6, 10, 15. The logging the value at every change. This is good to monitor events and can be used on infinite streams. Scan will use next at every value.

## Pluck
* Works similar to map but works only for objects.
* Can get property or properties using pluck
```js
const Observable = Rx.Observable.fromEvent(document.querySelector('input'), 'input');
Observable
  .debounceTime(500)
//   .map(event => event.target.value)
  .pluck('target', 'value')
  .distinctUntilChanged()
  .subscribe({
  next : value => console.log(value)
});
```
* **Always** remember to use pluck/map/filter after debounceTime to save resources

## MergeMap (FlatMap)
* [concatMap vs MergeMap](https://blog.angularindepth.com/practical-rxjs-in-the-wild-requests-with-concatmap-vs-mergemap-vs-forkjoin-11e5b2efe293)
* It is used to merge two observables to get combined output.
* Let us say we have two inputs whose combined values we need, we will use mergreMap there
```js
const input1 = document.querySelector('#input1');
const input2 = document.querySelector('#input2');
const span = document.querySelector('span');
Rx.Observable.fromEvent(input1, 'input')
	.subscribe({
  	next: event => span.textContent = event.target.value
  });
Rx.Observable.fromEvent(input2, 'input')
	.subscribe({
  	next: event => span.textContent = event.target.value
  });
```
* Now in the above code, the text in the span will get replaced by either of the two inputs we have. We must use mergeMap to get a combined value
* MergeMap will take an outer observable and then will merge an inner observable into it. And when the inner observable emits a value, it will merge the outer observables value into it to give us a combined value.
* We have to return an observable from the mergeMap function. We returned a combined value of two observables. Nothing happens when we write in the 1st input box as it is the outer observable. As soon as we type in the 2nd input box we get the output as it is the inner observable and mergeMap fires only when the inner observables value is merged with the outer observable. We used map on the inner observable to make sure we get the desired value. it will apply to both events.

```js
const input1 = document.querySelector('#input1');
const input2 = document.querySelector('#input2');
const span = document.querySelector('span');
const obs1 = Rx.Observable.fromEvent(input1, 'input');
const obs2 = Rx.Observable.fromEvent(input2, 'input');

obs1.mergeMap(
  event1 => {
    return obs2.map(event2 => event1.target.value + ' ' + event2.target.value)
  }
)
  .subscribe(
  value => span.textContent = value
);
```

## SwitchMap

* We use switchMap to cancel the last request and serve the new call.
* For example, if we use the below code, a new subscription will take place on every click. That is, every time user clicks on the click me button, the console log will start from 0 to --- value. Multiple clicks will mean multiple simultaneous countings.
* We can stop that by using mergeMap will cancel the last subscription and start a new one every time the user clicks the click button.
* In a switch map we return a function where was pass the value of the outside observable but insdie of the function body we return the second observable. Switchmap will now react to the values emitted from the outer observable and it will then trigger the inner observable, basically switch the values. We wont recieve the click events but the values of the inner observables hence the name switcMap. The key is that it will cancels all old subscriptions if the outer observable is triggered(click button clicked).
```js
// Multiple clicks on click me will trigger multiple subscriptions which will run simultaneously
const button = document.querySelector('button');
const obs1 = Rx.Observable.fromEvent(button, 'click');
const obs2 = Rx.Observable.interval(1999);
obs1.subscribe(event => obs2.subscribe
               (value => console.log(value)));
```
```js
// switchMap cancels all the old subscriptions and starts the new one only
const button = document.querySelector('button');
const obs1 = Rx.Observable.fromEvent(button, 'click');
const obs2 = Rx.Observable.interval(1999);
obs1.switchMap(event => obs2).subscribe(value => console.log(value));
```

## Behavior Subject

* We use it as a variable which when changed will let all the other parts of the project know that it has changed. We subscribe on it and make the functionality run over when the value changes.
* It is similar to Subject. But Subjects do not have a default value. eg.
```js
const SubjectEmitted = new Rx.Subject();
const button = document.querySelector('button');
const div = document.querySelector('div');
button.addEventListener('click', (e) => SubjectEmitted.next('clicked!!'));
SubjectEmitted.subscribe(data => div.textContent = data);
// SubjectEmitted.next('Not Clicked'); // This is the extra code we need to write manually to set the default code. Technically this is still incorrect as it will set it as the default value only when this line is reached in the code.
```
* Using behavior subject
```js
const SubjectEmitted = new Rx.BehaviorSubject('Default Value');
const button = document.querySelector('button');
const div = document.querySelector('div');
button.addEventListener('click', (e) => SubjectEmitted.next('clicked!!'));
SubjectEmitted.subscribe(data => div.textContent = data);
```
## ForkJoin

* Many times, we need to load data from more than one source, and we need to delay the post-loading logic until all the data has loaded.
* ReactiveX Observables provide a method called forkJoin() to wrap multiple Observables. Its subscribe() method sets the handlers on the entire set of Observables.
```js
  getBooksAndMovies() {
    Observable.forkJoin(
        this.http.get('/app/books.json').map((res:Response) => res.json()),
        this.http.get('/app/movies.json').map((res:Response) => res.json())
    ).subscribe(
      data => {
        this.books = data[0]
        this.movies = data[1]
      },
      err => console.error(err)
    );
  }
```
* Notice that forkJoin() takes multiple arguments of type Observable. These can be Http.get() calls or any other asynchronous operation which implements the Observable pattern. We don't subscribe to each of these Observables individually. Instead, we subscribe to the "container" Observable object created by forkJoin().
* When using Http.get() and Observable.forkJoin() together, the onNext handler will execute only once, and only after all HTTP requests complete successfully. It will receive an array containing the combined response data from all requests. In this case, our books data will be stored in data[0] and our movies data will be stored in data[1].
* The onError handler here will run if either of the HTTP requests returns an error code.
