# RxJS

Reactive Programming is programming with asynchronous data streams. Imagine an event bus or a typical click event which works with asynchronous event streams, on which you can observe and do some side effects. **Reactive is that idea on steroids.** We can create data streams of anything and not just click and hover events.

A library for componsing asynchronous and event-based programs by using observable sequences. Core features of the library are:

1. *Observables* - a collection of future values or events
2. *Observer* - collection of callbacks that knows how to listen to values from Observables
3. *Schedulers* - centralized dispatchers to control concurrency allowing us to coordinate when computations happen on eg setTimeout or requestAnimationFrame
4. *Subjects* - equivalent to an EventEmitter, and the only way of multicasting a value or event to multiple Observers
5. *Operators* - pure funtions to deal with values emitted by Observables

ReactiveX combines the Observer pattern with the Iterator pattern and functional programming with collections to manage sequence of events.

## Observables

They are lazy **push** collections of multiple values over time.

Functions in JavaScript are of **pull** in nature. They are invoked by the consumer of information whenever it requires a value. Functions are not aware when they are going to produce values. Observables, on the other hand, are **push** in nature. They decide when values are produced over time and the observers simply wait to consume that information.

An analogy for functions could be a restaurant. A chef doesn't prepare the dishes by his own. He/she waits for the consumer to ask for food which is then prepared.

An analogy for observables could be a subscription to a magazine. The producer decides when a new magazine is published and distributed to customers. They simply wait for the publisher to send out a new magazine.

Important point to remember is that the Observable(magazine publisher) wouldn't produce a value until there is atleast one subscriber. It's a loss of money otherwise for the magazine publisher, isn't it?

```javascript
class Observable {
  _subscribe;

  constructor(subscribe) {
    this._subscribe = subscribe;
  }

  subscribe(next, error?, complete?) {
    let observer;
    if (typeof next === 'function') {
      observer = {
        next,
        error: error || function () { console.log('DEFAULT ERROR') },
        complete: complete || function () { console.log('DEFAULT COMPLETE') },
      }
    } else {
      observer = next;
    }
    this._subscribe(observer);
  }
}

const observable$ = new Observable((observer) => {
  observer.next(10);
  observer.next(10);
  observer.error('ERR!!');
  observer.complete();
});

/* First way to subscribe */
observable$.subscribe((value) => {
  console.log('SUBSCRIBED OLD', value);
});

/* Second way to subscribe */
observable$.subscribe({
  next: (val) => { console.log('SUBSCRIBED NEXT', val) },
  error: (err) => { console.log('SUBSCRIBED ERROR', err) },
  complete: () => { console.log('SUBSCRIBED COMPLETE') },
});
```

## Subject

A Subject is a special type of Observable that allows values to be multicasted to many observers. While plain Observables are unicast i.e each subscribed observer owns an independent execution of the Observable, subjects are multicast.

```javascript
let count1 = 0;
let count2 = 0;

const observable = new Observable(subscribe => {
  subscribe.next(10);
  count1++;
  subscribe.next(20);
  count1++;
});

observable.subscribe((val) => {
  console.log('OBSERVER1', val);
});

observable.subscribe((val) => {
  console.log('OBSERVER2', val);
});

console.log('COUNT1 is', count1);

const subject = new Subject<number>();

subject.subscribe((val) => {
  console.log('SUBSCRIBE1', val);
});

subject.subscribe((val) => {
  console.log('SUBSCRIBE2', val);
});

subject.next(10);
count2++;
subject.next(20);
count2++;

console.log('COUNT2 is', count2);
```

**Every Subject is an Observable** i.e one can subscribe to it

```javascript
const subject = new Subject<number>();

subject.subscribe((val) => {
  console.log('SUBSCRIBE1', val);
});
```

**Every Subject is an Observer** i.e it is an object with `next`, `error` and `complete` methods to feed a new value to the Subject which will be multicasted to all observers.

```javascript
subject.next(10);
subject.complete();
```

### BehaviorSubject

A variant of Subject which stores the latest value emitted to its consumers and whenever a new Observer subscribes, it immediately receives the "current value" from BehaviorSubject.

```javascript
const subject = new BehaviorSubject(0); // 0 is the initial value
 
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});
 
subject.next(1);
subject.next(2);
 
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`)
});

// Logs:
// observerA: 0
// observerA: 1
// observerA: 2
// observerB: 2
```

### ReplaySubject

Similar to BehaviorSubject except it can also record a part of the Observable execution. It records multiple values from Observable execution and replays them to new subscribers.

```javascript
const subject = new ReplaySubject(3); // buffer 3 values for new subscribers
 
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});
 
subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);
 
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`)
});
 
subject.next(5);
 
// Logs:
// observerA: 1
// observerA: 2
// observerA: 3
// observerA: 4
// observerB: 2
// observerB: 3
// observerB: 4
```

### AsyncSubject

The AsyncSubject is a variant where only the last value of the Observable execution is sent to its observers, and only when the execution completes.

```javascript
const subject = new AsyncSubject();
 
subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`)
});
 
subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);
 
subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`)
});
 
subject.next(5);
subject.complete();
 
// Logs:
// observerA: 5
// observerB: 5
```

## Operators

RxJS is mostly useful for its operators even though without the Observable there is no stream of values. Operators allow complex async code to be easily composed in a declarative manner.

Different [category](https://github.com/ReactiveX/rxjs/blob/master/doc/operators.md#choose-an-operator) of operators are:

1. Creation Operators
2. Transformation Operators
3. Filtering Operators
4. Combination Operators
5. Multicasting Operators
6. Error Handling Operators
7. Utility Operators
8. Conditional Operators
9. Mathematical Operators


## References

- https://gist.github.com/staltz/868e7e9bc2a7b8c1f754
- https://rxjs-dev.firebaseapp.com/guide/overview
- https://github.com/ReactiveX/rxjs/tree/master/doc
