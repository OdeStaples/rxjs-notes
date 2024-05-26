# of

- `of` allows us to create an observable out of one or more values.
- `of` emits its values immediately and completes synchronously.
- Useful for creating Observables with static data or test data.
- Not suitable for asynchronous data sources like HTTP requests or user interactions.

- Example:

  ```ts
  import { of } from "rxjs";
  const example$ = of(1, 2, 3);
  example$.subscribe((val) => console.log(val));
  ```

- The `$` is just a preference. You can omit it if you want.

# from

- `from` creates an observable from a promise or anything array or observable like.

- Example:

  ```ts
  import { from } from "rxjs";
  const anotherExample$ = from([1, 2, 3]);
  anotherExample$.subscribe((val) => console.log(val));
  ```

# for vs of

- `from` is used whenever you are working with is kind of an observable. Like arrays, promises
- `of` is used when it's nothing like an observable. Like primitive types (strings, numbers, booleans, null, undefined), objects
- `from` is used with Observable, Promise, ReadableStream, Array,AsyncIterable, or Iterable.

# Ways of listening to Observables

- Old way (deprecated)

```ts
example$.subscribe(
  (value) => {
    console.log("The first function is called when a value is emitted.");
  },
  (error) => {
    console.log("The second function is called when an error shows up.");
  },
  () => {
    console.log("The third function is called when the observable completes.");
  }
);
```

- New Way

```ts
example$.subscribe({
  next: (value) => {
    console.log("Emitted Value: ", value);
  },
  error: (error) => {
    console.log("Error: ", error);
  },
  complete: () => {
    console.log("Complete");
  },
});
```

- we usually need the value most of the times - hence using first method will work.

# Subscribe

-In RxJS, `subscribe()` is a method used to attach an observer to an observable. This allows the observer to listen for and react to events emitted by the observable. The `subscribe()` method takes up to three arguments: `next`, `error`, and `complete`. These are callback functions that handle different types of notifications from the observable:

- `next`: A function that handles the emission of a new value from the observable.
- `error`: A function that handles errors thrown by the observable.
- `complete`: A function that handles the completion notification from the observable, indicating that it has finished emitting values.

# Handling Async Data

```ts
import { from } from "rxjs";

const promise = new Promise(returnPromise);

function returnPromise(resolve, reject) {
  resolve("Hello There!");
}

const someObservable$ = from(promise);
someObservable$.subscribe({
  next: function nextData(data) {
    console.log("Data from Promise ->", data); // gets printed second
  },
  error: function errorData(error) {
    console.error("Promise Error ->", error);
  },
  complete: function completionData() {
    console.log("Observable Complete"); // gets printed third
  },
});
console.log("This will be printed first"); // gets printed first
```

- In the above example, the `nextData` message prints first as the promise is resolved immediately, followed by the `completionData`

- In case of an error, only the error message inside `errorData` will be printed. The log inside `completionData` will not be printed as there was an error and the observable didn't complete.

# Events

- We can create observables based on event listeners using `fromEvent`

- The RxJS fromEvent is a creation function that creates an Observable that emits events of a specific type coming from the given event target.

- It works with any document object model (DOM) element, such as buttons, input elements, and the document itself.

- Example:

  ```ts
  import { fromEvent } from "rxjs";

  const button = document.querySelector("button");

  const fromEvent$ = fromEvent(button, "click");
  fromEvent$.subscribe((event) => {
    console.log("Button Text ->", event.target.innerText);
  });
  ```

- In the above example we fetch the `button` element from the DOM and add a `click` event listener using `fromEvent$` observable.

- we then subscribe `fromEvent$` obervable and pass a callback function which prints the inner text of the button.

- optionally, you can pass a function to format the event

  ```ts
  import { fromEvent } from "rxjs";

  const button = document.querySelector("button");

  const fromEvent$ = fromEvent(button, "click", (event) => {
    console.log("Button Text ->", event.target.innerText);
  });
  fromEvent$.subscribe();
  ```

- the output will be the same as the previous example

# bindCallback

- turns anything that takes a callback into an observable.

- useful if you are using node.js, node takes the error as the first argument and the result as the second argument.

- bindCallback takes node kind of arguments and turns it into an observable.

  ```ts
  const get = bindCallback(jQuery.get);
  const data$ = get("/api/endpoint");

  const readFile = bindCallback(fs.readFile);
  const content$ = readFile("./groceries.md", "utf-8");
  ```

# fromFetch

- wraps the fetch api in an observable

```ts
import { fromFetch } from "rx.js/fetch";
const data$ = fromFetch("api/endpoint");
```

- acts like promise but has additional option of aborting request etc.
