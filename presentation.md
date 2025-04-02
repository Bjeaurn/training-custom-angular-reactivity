---
title: "Angular Reactivity+ (for Kadaster)"
---

# Angular Reactivity+

---

## ToC
- Reactivity "refresh", intro into "Why reactivity"
- Modern Signal examples
- Comparison to RxJS "alternatives"
- ? Discussion about simplicity, expressiveness, readability ?
- RxJS interop
- Angular specific examples

---

<div style="">
  <img src="./assets/bjorn.jpg" width="100" style="border-radius:100%; display: inline-flex;">
  <h1 style="font-size: 0.9em;">Bjorn Schijff</h1>
  <small style="display: inline-flex;">Sr. Frontend Engineer / Architect</small>
  <div>
    <img src="./assets/codestar.svg" height="30" style="border: 0; background-color: transparent;">
  </div>
  <small>@Bjeaurn</small>
  <br />
  <small>bjorn.schijff@soprasteria.com</small>
</div>

Note: Introduction, but also: Who are you? Introduce yourself in 30 seconds/1m?

---

What is Reactivity?

Note: What is reactivity? What is reactive programming? [DISCUSS]

----

> Reactive programming is a **declarative** programming paradigm concerned with **data streams** and the **propagation of change**.

<small>https://en.wikipedia.org/wiki/Reactive_programming</small>

Note: Declarative, data streams, propagation of change.

----

```js
var count = 0;

function render() {
    document.getElementById("counter").innerHTML = count;
}

document.getElementById("counterPlus")
        .addEventListener('click', () => {
    count = count + 1;
    render(); // <--- This here...
})
```
 

```html
<div>
    <span class="counter"></span>
    
    <button class="counterPlus">+</button>
</div>
```

Note: Regular variable being updated on click. This is not reactive. (Imperative!). Angular does this for you.

---

```ts
var count = signal(0);

document.getElementById("counter")
        .addEventListener('click', () => {
            count.update(value => value + 1);
        })

// Pseudo-code
computed(() => {
    document.getElementyById("counter").innerHTML = count();
});
```

Note: Pseudo-code for a more Reactive example. Splitting the render logic away from the "business" logic of updating the count. This probably won't work or be different depending on your Signal API.

---

## Imperative example for comparison

Note: "reactive-werken-in-frontend.md", depending on how quickly or well the discussions are going: Maybe use 5-15 minutes to go through the examples.

---

# Reactive

Declarative

Data Streams

Propagation of Changes

Note: So after the example and discussion, quick recap; what is reactive programming.

----

**Declarative**

```js
// Imperative - This is how NOT to do it.
let count = 0;

function render() {
    document.getElementById("counter").innerHTML = count;
}

document.getElementById("counterPlus")
        .addEventListener('click', () => {
    count = count + 1;
    render(); // <--- This here...
})

document.getElementById("counterMin")
        .addEventListener('click', () => {
            count = count - 1;
            render() // <-- This again. Oh no, I forgot it?!
        })

// How do I see how the count is influenced over time? 
// By different parts of the application?
```
 
Note: Imagine we add another button that influences how the counter changes. The minus example. Now we have two functions, they both influence the same state and have to call the same functions.

----

**Declarative**

```ts
// Angular 17.1+
class ListComponent {
  list = input([]);
  emptyHeading = input('');

  count = computed(() => this.list().length);

  heading = computed(() => {
    if (this.count() > 0) {
      const noun = this.count() > 1 ? 'items' : 'item';
      return this.count() + ' ' + noun;
    }
    return this.emptyHeading();
  });
}
```

Note: A declarative example. We write the "recipe" and every definition has its own easy to follow set of rules.

----

**Data Streams**

You could consider a Signal to be a current state of values that change over time (as a stream).

```ts
count = signal(0)

// User clicks a button
count.update(value => value + 1)
// Count is now 1.

// User clicks again (count: 2)
// User may wait for a bit... (count: 2)
// User clicks *again* (count: 3)
```

Note: This is a basic visualisation on how a value might change over time, representing a stream of data. We'll come back to this in the RxJS part.

----

**Propagation of Change**

When a Signal changes, it notifies all interested. You don't have to manually keep track or update the state of another variable. This keeps the code clean and makes the flow of data more "Push" instead of "Pull".

---

### Any modern framework takes care of this problem for you. 

Note: So you can focus on the business logic, not the two-way binding. But the same benefits stay! And because it's Framework native, they can integrate it with their change logic etc., so it's win-win!

----

### Angular example

```ts
@Component({
    ...
    template: `<div>{{ counter() }}</div>
    <button (click)="addCount()">+</button>`
}) export class Component {
    counter = signal(0);

    addCount() {
        counter.update(count => count + 1);
    }
}
```

Note: Angular in this example, takes care of subscribing to your counter signal and binding your click to the function. Because of the two-way binding also takes care of updating the value.

----

### React example

```tsx
const Component = () => {
  const [counter, setCounter] = useState(0);
  return (
    <>
      <div>{counter}</div>
      <button onClick={() => setCounter(counter + 1)}>
          +
        </button>
    </>
  );
};
```

Note: React uses hooks to connect the state value updates to the UI and update it automatically when the state changes.

---

## Pros & Cons

Of Signals

----

## Pros

- Always up to date state of a value
- No need to notify or be aware of other parts of the application
- More declarative approach is possible

----

## Cons

- No real "value over time", only a current state.
- Slightly more complicated API for changing values.
- Not every Signal is the same; some are writeable, some are only calculated.

Note: Point 3: WriteableSignal vs. ComputedSignal.
 
---

### But what does a Signal do then?

- Holds a (initial) state value<!-- .element: class="fragment" -->
- Receives and stores state changes<!-- .element: class="fragment" -->
- Returns current state on demand<!-- .element: class="fragment" -->
- Informs all listeners on state changes<!-- .element: class="fragment" -->

---

# Angular's Signals

* Available since v16 (experimental)<!-- .element: class="fragment" -->
* Stable since v17<!-- .element: class="fragment" -->
* Signal input/output since v19<!-- .element: class="fragment" -->
* LinkedSignal since v19<!-- .element: class="fragment" -->
* v20 coming in May!<!-- .element: class="fragment" -->

Note: What version are you on? How is upgrading going?

----

https://www.angular.courses/caniuse

https://angular.dev/reference/releases#actively-supported-versions

---

# Exercises

Exercises 1-3

Note: Exercises 1 to 3. Get some coffe, take 10-15m? Then we'll discuss and go over the exercises.

---

## Comparison with existing solutions

# RxJS<!-- .element: class="fragment" -->

----

Signals: Synchronous, current state.

RxJS: Asynchronous streams, events.

Note: If you're using RxJS just to keep track of a current state, switching that over to a Signal might be a good idea!

---

# RxJS

- Implementation of the Observable pattern in Javascript
- Used heavily by `Angular`
- Lots of adoption in libraries like
  - `Redux-observable`
  - `VueRx`
  - `NgRx`
- Java/Scala, C#, also have their implementation

---

# The RxJS Contract

----

### Wait, what's a contract?

- A set of rules agreed upon <!-- .element: class="fragment" -->
- Using the same language throughout <!-- .element: class="fragment" -->
- Ensures we're all using the same things for the same reasons. <!-- .element: class="fragment" -->

----

### So, the RxJS Contract

- Observable<!-- .element: class="fragment" -->
- Subscribers<!-- .element: class="fragment" -->
- Subscription<!-- .element: class="fragment" -->
- Operators<!-- .element: class="fragment" -->
- Subject<!-- .element: class="fragment" -->

Note: Check if we know all of them, ask around.

---



#### Observable

- Datasource\* <!-- .element: class="small" -->
- You can `subscribe` to its contents, much like a newsletter.
- Unlike a newsletter, it won't send anything until at least someone has a `subscription`.
- You can use `operators` on it, to get the parts you are interested in.

---

#### Subscribers

```js
const obs = from([1, 2, 3]);

obs.subscribe({
  next: (next) => {
    console.log(next);
  },
  error: () => {},
  complete: () => {
    console.log("I'm done!");
  },
});

// What does this do?
```

- A subscriber activates an Observable. <!-- .element: class="fragment" -->
- Within a subscriber you handle every value, an error, or when it's done. <!-- .element: class="fragment" -->
- Without error handling, the subscription will end immediately <!-- .element: class="fragment" -->

---

#### Subscription

```js
const obs = from([1, 2, 3]).pipe(delay(1));

const subscription = obs.subscribe({
  next: (next) => {
    console.log(next);
  },
  error: () => {},
  complete: () => {
    console.log("I'm done!");
  },
});

subscription.unsubscribe();

// What do you think this does?
```

```js
// and what happens when we remove the delay()?
```

<!-- .element: class="fragment" -->

- A subscription let's you unsubscribe from the ~~newsletter~~ `Observable`.

---

#### Operators

- Operators are small operations you perform on top of your Observable. <!-- .element: class="fragment" -->
- You can use multiple operators to transform the data to your liking. <!-- .element: class="fragment" -->
- This is done using the pipe() method, introduced in RxJS v6 <!-- .element: class="fragment" -->
- There's multiple categories of operators.<br />More on that later! 🔜<!-- .element: class="fragment" -->

---

#### Creation operators

- Generates an Observable based on your input
  `from() of()`<br /> <!-- .element: class="fragment" -->
  `interval() EMPTY`<br /> <!-- .element: class="fragment" -->
  `merge() concat()`<br /> <!-- .element: class="fragment" -->
  `zip()`<br /> <!-- .element: class="fragment" -->
  - and more! (On that later!)<!-- .element: class="fragment" -->

Note: from() is the catchall for a lot of different variables, including Promises. fromPromise has been deprecated a while ago. Corneel en Tobias doen hier leuke analogie??

---

```js
const obs = from([1, 2, 3]).pipe(
  map((x) => x * 2),
  filter((x) => x < 4)
);

obs.subscribe({
  next: (next) => {
    console.log(next);
  }, // 2, 4, I'm done!
  error: () => {},
  complete: () => {
    console.log("I'm done!");
  },
});
```

<!-- .element: class="fragment" -->

```
// What happens when we switch the map() and filter() around?
```

<!-- .element: class="fragment" -->

---

#### But wait, there's more! 🙀

- Subject <!-- .element: class="fragment" -->
  - BehaviorSubject
  - ReplaySubject
- Observers <!-- .element: class="fragment" -->

Note: We might get into this, as they compare nicely to what Signals make very easy for us.

---

# Exercises

Exercises 4-6

Note: Short break, 10-15m? // TODO Vervangen voor subject vs. signal voorbeelden?
// Daarna door naar meer inhoudelijke RxJS voorbeelden?
// Wel baseren op RxJS 101...

---

# Common mistakes

- By André Staltz

----

- Using Subjects too much. Use Observable.create!<!-- .element: class="fragment" -->
- Using Observable.create too much. Use Creation operators<!-- .element: class="fragment" -->
- Subscribing too much and unsubscribing too much<!-- .element: class="fragment" -->
- Subject.next inside subscribe<!-- .element: class="fragment" -->
- Subscribe inside subscribe<!-- .element: class="fragment" -->

----

// TODO 
Expand with more Angular specific examples, especially in comparison with modern Signal solutions?

---

## Cold versus Hot

- A cold ❄️ Observable does not emit events when there are no subscribers.<!-- .element: class="fragment" -->
- A hot 🔥 Observable does emit events, even if there's no subscriber.<!-- .element: class="fragment" -->
- A cold ❄️ Observable creates a new stream for each subscriber.<!-- .element: class="fragment" -->
- A hot 🔥 Observable adds a new subscriber to the existing Observable.<!-- .element: class="fragment" -->

Note: Check if the class knows the difference before explaining?

----

```ts
const obs = interval(1000).pipe(
  // share(), <--- This makes it hot! 🔥
  // Share secretly is a `multicast()` with a `refCount()`
  take(5)
);

obs.subscribe((a) => console.log("A", a));

setTimeout(() => {
  obs.subscribe((b) => console.log("B", b));
}, 1001); // We start it after the first emit.
```

```js
// 🔥 Hot: A0, A1, B1, A2, B2, etc.
// ❄️ Cold: A0, A1, B0, A2, B1, A3, B2, etc.
```

---

Final Exercise

---

# Recap

// Todo write recap.

---

<div style="">
  <img src="./assets/bjorn.jpg" width="100" style="border-radius:100%; display: inline-flex;">
  <h1 style="font-size: 0.9em;">Bjorn Schijff</h1>
  <small style="display: inline-flex;">Sr. Frontend Engineer / Architect</small>
  <div>
    <img src="./assets/codestar.svg" height="30" style="border: 0; background-color: transparent;">
  </div>
  <small>@Bjeaurn</small>
  <br />
  <small>bjorn.schijff@soprasteria.com</small>
</div>