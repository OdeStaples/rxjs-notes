- operators are often used to manipulate the data stream

# Pipe Operator

- every observable has a `.pipe` method

- this method takes in one or more functions called operators

- each operator takes the observable, does something and returns a new observable

- In RxJS, the `pipe()` method can technically accept an unlimited number of operators. However, when using TypeScript, there is an apparent limit of 9 operators within a single `pipe()` call due to TypeScript's type inference capabilities. This limit is not a hard restriction of RxJS itself but rather a limitation imposed by TypeScript's type system to manage the complexity of inferring types across multiple operators.

- Once you have more than 9 operators, TypeScript's type inference between consecutive parameters stops, and `pipe` returns `Observable<{}>`. This means that TypeScript cannot infer the type of the observable anymore, and you will need to use a type assertion to indicate the observable type explicitly. For example, if you have more than 9 operators and you want to maintain type safety, you would need to do something like this:

  ```typescript
  pipe(/* lots of operators */) as Observable<Boolean>;
  ```

- If you need to use more than 9 operators while maintaining type safety, you can split your operators across multiple `pipe()` calls. Each `pipe()` call can contain up to 9 operators, and you can chain these calls together. This approach allows you to maintain type safety after the 9th operator by ensuring that each `pipe()` call is correctly typed. Here's an example:

  ```typescript
  source$
    .pipe(
      tap(() => {}),
      tap(() => {}),
      tap(() => {}),
      tap(() => {}),
      tap(() => {}),
      tap(() => {}),
      tap(() => {}),
      tap(() => {}),
      tap(() => {})
    )
    .pipe(
      // Add the next operators after the 9th here
      tap(() => {})
    );
  ```

- This method of chaining `pipe()` calls is a practical workaround for the TypeScript limitation and allows you to use as many operators as you need while keeping your code type-safe.

# Take Operator

- take a certain number of values from an observable and then stop.

  ```ts
  export class AppComponent implements OnInit {
    val!: number;
    ngOnInit(): void {
      function* fibonacci() {
        var values = [0, 1];
        while (true) {
          var [current, next] = values;
          yield current;
          values = [next, current + next];
        }
      }

      function getFibonacciData(this: AppComponent, value: number) {
        console.log(value); // // logs 0,1,1,2,3,5,8,13,21,34
        this.val = value;
      }

      const example$ = from(fibonacci()).pipe(take(10));
      example$.subscribe(getFibonacciData);
    }
  }
  ```

  ![take-diagram](image-23.png)

  - in the above example, we convert the `fibonacci` function into an observable using `from` operator and then we use `pipe` to transform the data and `take` operator to take 10 values.

# Skip Operator

- ignore the first however many values.
  ```ts
  const example$ = from([1, 2, 3, 4, 5]).pipe(skip(2));
  example$.subscribe((val) => {
    console.log(val); // logs 3, 4, 5
  });
  ```

# takeWhile and skipWhile

- take or skip values as long a certain condition is met. Complete the observable when the conditon returns false.

- takeWhile also has a second argument (boolean) that emits the value which completes the observable. By default it is `false`

![takeWhile-diagram](image-24.png)

```ts
const under200$ = from(fibonacci()).pipe(takeWhile(value)=>value<200);
under200$.subscribe((val)=>console.log(val)); // logs 0,1,2,3,5,8,13,21,34,55,89,144
under200$.subscribe((val)=>console.log(val), true); // logs 0,1,2,3,5,8,13,21,34,55,89,144,233

const over100$ = from(fibonacci()).pipe(skipWhile(value)=>value<100);
over100$.subscribe((val)=>console.log(val)); // logs 144, 233, 377, 610
```

# This code will not print any values. Why?

```ts
import { takeWhile, from, skipWhile } from "rxjs";

function* generateFibonacci() {
  var values = [0, 1];
  while (true) {
    let [curr, next] = values;
    yield curr;
    values = [next, curr + next];
  }
}

const example$ = from(generateFibonacci()).pipe(
  skipWhile((value) => value < 100),
  takeWhile((value) => value > 500)
);
example$.subscribe(function generateSequence(value) {
  console.log(value);
});
```

- in the above example, the no value will be printed.

- since the fibonacci sequence starts from 0, the `skipWhile` condition will be true but the `takeWhile` condition will be false.

- the operators provided will emit values as long as the condition specified in the operator is true. Once the condition becomes false, the operator will stop emitting values. This behaviour is characteristic of operators like `takeWhile` and `skipWhile`, which are used to control the flow of values based on conditions.

- and since the `takeWhile` condtion becomes false for the first fibonacci value, so no value is printed.

# takeUntil

- The `takeUntil` operator in RxJS allows you to specify a condition under which an observable sequence stops emitting items. Essentially, it completes the observable sequence when a notifier observable emits its first item. This is particularly useful for unsubscribing from observables based on certain events, such as the destruction of a component in Angular applications.

  ```ts
  import { Subject } from "rxjs";
  import { takeUntil } from "rxjs/operators";

  class MyComponent {
    private destroy$ = new Subject<void>();

    constructor(private myService: MyService) {
      this.myService
        .getDataStream()
        .pipe(takeUntil(this.destroy$))
        .subscribe((data) => console.log(data));
    }

    ngOnDestroy() {
      this.destroy$.next();
      this.destroy$.complete();
    }
  }
  ```

  ![takeUntil-diagram](image-25.png)

- In this example, `getDataStream()` returns an observable that emits data. The `takeUntil` operator is used with a `Subject` (`destroy$`) that emits a value when the component is destroyed (`ngOnDestroy`). This effectively unsubscribes from `getDataStream()` when the component is no longer needed, preventing memory leaks.

# skipUntil

- The `skipUntil` operator is similar to `takeUntil` but instead of completing the observable sequence upon the first emission of the notifier observable, it ignores all items from the source observable until the notifier emits. After the notifier emits, skipUntil starts forwarding items from the source observable.

  ```ts
  import { fromEvent } from "rxjs";
  import { skipUntil } from "rxjs/operators";

  const clicks = fromEvent(document, "click");
  const result = clicks.pipe(skipUntil(fromEvent(document, "dblclick")));

  result.subscribe((x) => console.log(x));
  ```

- In this example, `clicks` is an observable of click events. The `skipUntil` operator is used with another observable that emits when a double-click event occurs. Until a double-click happens, all click events are ignored. Once a double-click is detected, subsequent click events are logged to the console.

# filter

- works just like it does on arrays

  ```ts
  const evenNum$ = of(1, 2, 3, 4, 5, 6, 7, 8, 9).pipe(
    filter((num) => num % 2 === 0)
  );
  evenNum$.subscribe((val) => console.log(val)); // logs 2,4,6,8
  ```

  ![filter-diagram](image-19.png)

# map

- works just like it does on arrays

  ```ts
  mapExample$ = of(1, 2, 3).pipe(map((num) => num * 2));
  mapExample$.subscribe((val) => console.log(val)); // logs 2,4,6
  ```

  ![map-diagram](image-18.png)

# pluck

- Like map, but meant only for picking one of the nested properties of every emitted value.

  ```ts
  import { fromEvent, pluck } from "rxjs";

  const clicks = fromEvent(document, "click");
  const tagNames = clicks.pipe(pluck("target", "tagName"));

  tagNames.subscribe((x) => console.log(x));
  ```

  ![pluck-operator](https://rxjs.dev/assets/images/marble-diagrams/pluck.png)

# mapTo

- this is just a simplified version of map

  ```ts
  const over100$ = from(generateFibonacci()).pipe(
    skipWhile((val) => val < 100),
    take(4),
    mapTo("Hello")
  );

  over100$.subscribe((val) => console.log(val)); // logs Hello, Hello, Hello, Hello
  ```

# reduce

- works just like it does on arrays

```ts
const sum$ = from(generateFibonacci()).pipe(
  takeWhile((val) => val < 200),
  reduce((acc, val) => acc + val, 0)
);
```

![reduce-diagram](image-20.png)

- the `0` passed in the end is the initial value for the accumulator.
- Check Frontend Masters javascript course for more.

# scan

- works like reduce but instead of returning a single value, it reutrns sum for each iteration.

  ```ts
  const sum$ = from(generateFibonacci()).pipe(
    takeWhile((val) => val < 200),
    scan((acc, total) => acc + total, 0)
  );

  sum$.subscribe((val) => console.log(val)); // logs 1,3,6,11,19,87,142,231,375
  ```

- `scan` can take an infinite range

  ```ts
  const range$ = range(0, Infinity).pipe(
    scan((acc, total) => acc + total, 0),
    take(4)
  );
  range$.subscribe((val) => console.log(val)); // logs 0,1,3,6
  ```

  ![scan-diagram](image-21.png)

# tap

- useful for side-effects like DOM Manipulation and debugging.

- can be used in various scenarios where you need to perform actions based on the observable stream's values without affecting the stream's data flow.

The `tap` operator in RxJS is a powerful tool for performing side effects in an observable stream without altering the stream itself. It's particularly useful for debugging, logging, or executing any action that doesn't change the emitted values in the observable stream. Here are some examples to illustrate its usage:

### Basic Usage

```javascript
import { tap } from "rxjs/operators";

observable$.pipe(
  tap((value) => {
    console.log("Received value:", value);
    // Perform side effects here, but be careful not to modify the value
  })
);
```

![tap-diagram](image-22.png)

### Logging Authentication Status

```javascript
import { tap } from "rxjs/operators";

authService.isAuthenticated$().pipe(
  tap((isAuthenticated) => {
    if (isAuthenticated) {
      console.log("User is authenticated");
    } else {
      console.log("User is not authenticated");
    }
  })
);
```

### Handling HTTP Requests

```javascript
import { tap, catchError } from "rxjs/operators";

http.get("https://api.example.com/data").pipe(
  tap((response) => {
    console.log("Received response:", response);
  }),
  catchError((error) => {
    console.error("An error occurred:", error);
    throw error; // Re-throw the error to handle it further
  })
);
```

### Caching Emitted Values

```javascript
import { tap } from "rxjs/operators";

let cachedData: any;

observable$.pipe(
  tap((value) => {
    // Store the value in the cache
    cachedData = value;
  })
);
```

### Showing a Loading Spinner

```javascript
import { tap } from "rxjs/operators";

let isLoading = true;

observable$.pipe(
  tap(() => {
    // Show loading spinner
    isLoading = true;
  }),
  finalize(() => {
    // Hide loading spinner when the observable completes
    isLoading = false;
  })
);
```

### Saving Data to LocalStorage

```javascript
import { tap } from "rxjs/operators";

observable$.pipe(
  tap((value) => {
    localStorage.setItem("Name", value);
  })
);
```

### Stopping Event Propagation

```javascript
clickStream$
  .pipe(
    tap((event) => {
      event.stopPropagation();
      event.preventDefault();
    }),
    debounce(300),
    map((event) => event.key)
  )
  .subscribe((key) => console.log("key debounced: ", key));
```

The `tap` operator is versatile and can be used in various scenarios where you need to perform actions based on the observable stream's values without affecting the stream's data flow. It's a valuable tool for enhancing the maintainability and readability of your code, especially in Angular applications.

# delay

- this operator delays each emitted value from the source observable by a certain amount of time.

  ```js
  const buttonClicks$ = fromEvent(button, "click").pipe(delay(2000));
  ```

- `delay` isn't necessarily spacing them out. If we rage click, they all come in roughly the same time, just two seconds in the future.

# toArray

- Collects all source emissions and emits them as an array when the source completes.

- `toArray` will wait until the source Observable completes before emitting the array containing all emissions. When the source Observable errors no array will be emitted.

```js
import { of, toArray } from "rxjs";

const source$ = of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

source$.pipe(toArray()).subscribe((data) => console.log(data)); // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

# throttleTime

```js
const buttonClicks$ = fromEvent(button, "click").pipe(
  throttleTime(2000)
  // delay(2000)
);
buttonClicks$.subscribe(console.log);
```

- You can keep clicking, but a new message will only show up every two seconds provided you keep on clicking the button

- in the above example, the click will be registered on the first click and after every 2secs another click will be registered.

![throttleTime-diagram](image-29.png)

# debounceTime

- `debounceTime` ignores emitted values until there is a period of silence and then we take the last one and deal with it.

- I can click that button to my heart's content, but a new notification will only be displayed after I chill out for a second.

  ```js
  const buttonClicks$ = fromEvent(button, "click").pipe(debounceTime(1000));
  ```

- in the above example, the repeated clicks till the las click will be ignored, then after the final(last) click is done, after a 1sec delay the logic will be invoked.

![debounceTime-diagram](image-28.png)

![debouneTime v throttleTime](image.png)

# throttle and debounce

- These operators work just like `throttleTime` and `debounceTime` but instead of relying on the given amount of time, they rely on the dependant observables until they emit a value.

# sampleTime and sample

- sampleTime allows you to take a sample of source observable on a duration that you define.

  ```js
  import { fromEvent, map, sampleTime } from "rxjs";

  const clicks = fromEvent(document, "click");
  const result = clicks.pipe(
    sampleTime(4000),
    map(({ clientX, clientY }) => {
      return { clientX, clientY };
    })
  );

  result.subscribe(console.log);
  ```

  ![sampleTime-diagram](image-30.png)

- in the above example, every 4 seconds, we’ll sample the latest click event that occurred during that time window.

- if you need to sample a latest value based on another observable, you can use sample

  ```js
  import { fromEvent, sampleTime, interval } from "rxjs";

  const click$ = fromEvent(document, "click");
  const timer$ = interval(1000);

  timer$.pipe(sample(click$)).subscribe(console.log);
  ```

![Sample-Time-Digram](image-1.png)

![sample-diagram](image-31.png)

![sampleTime vs throttleTime vs debounceTime](image-2.png)

# auditTime

- `auditTime` ignores source values for the duration you specify after an emitted value. After the window is passed, the last emitted value from the source observable is emitted.

- `auditTime` differs from `sampleTime` as a silence window is triggered by the emitted values from the source rather than repeated after subscription.

- `auditTime` differs from `throttleTime` as auditTime emits values at the trailing edge of the silence window as opposed to the leading edge.

- if you want to uniformly sample the source ever so often, then you `sampleTime`, if you want to instead sample a source for duration only after some event, then use `auditTime`

- you can achieve the behaviour identical to auditTime with throttleTime by providing a asyncScheduler config ({leading: false, trailing: true})

![Audit-Time-Diagram](image-3.png)

![Comparision](image-4.png)

# merge

- The merge operator in RxJS is used to combine multiple Observables into a single Observable. It allows you to handle emissions (data) from those Observables concurrently, creating a unified stream of values.

- It emits values as soon as any of the input Observables emit.

  ### Use Cases

  - Combining Events: When you want to react to events happening independently in your application, like user clicks on different buttons or keyboard inputs. Merging these Observables allows you to handle them in a single stream.

  - Parallel Tasks: If you need to execute multiple asynchronous tasks (like API requests) simultaneously, merge ensures you can work with the responses as they arrive, without waiting for each task to complete.

  ```js
  import { interval, merge } from "rxjs";

  const observable1 = interval(1000); // Emits every 1 second
  const observable2 = interval(2000); // Emits every 2 seconds

  const mergedObservable = merge(observable1, observable2);

  mergedObservable.subscribe((value) => {
    console.log(value); // Output: Values from both observables interleaved
  });
  ```

# concat

- The concat operator in RxJS is used to chain the emissions (data) from multiple Observables sequentially. It creates a single Observable that emits all the values from the provided Observables, one after the other, in the order they are specified.

  ### Use Cases:

  - **Sequential Operations:** When you have a sequence of tasks that depend on the completion of the previous task, concat ensures each Observable completes before moving on to the next. This is useful for scenarios like fetching data in a specific order or performing actions one after another.

  - **Data Loading:** If you need to load data from multiple sources and display it sequentially, concat allows you to show the data from each source as it becomes available, maintaining the order.

  ```js
  import { of, concat } from "rxjs";

  const observable1 = of(1, 2, 3); // Emits values 1, 2, 3
  const observable2 = of(4, 5, 6); // Emits values 4, 5, 6

  const concatenatedObservable = concat(observable1, observable2);

  concatenatedObservable.subscribe((value) => {
    console.log(value); // Output: 1, 2, 3, 4, 5, 6 (in order)
  });
  ```

# race

- The race operator in RxJS is designed for competitive scenarios where you want to identify the first Observable to emit a value (including errors or completion). It essentially creates a race between multiple Observables and returns a single Observable that emits the value from the winner.

- Imagine a race where only the first runner to cross the finish line is considered, and the others are disregarded.

  ### Use Cases:

  - **Timeouts:** When you need to set a time limit for an operation and react differently based on whether it completes within that time. You can create an Observable that emits after a specific delay and use race to see if the main operation finishes before the timeout.

  - **Competing Requests:** If you have multiple asynchronous requests fetching data from different sources, and you only need the data from the fastest source, race allows you to capture the first response and potentially cancel the others.

  ```js
  import { interval, race } from "rxjs";

  const observable1 = interval(1000); // Emits every 1 second
  const observable2 = interval(2000); // Emits every 2 seconds
  const observable3 = of("Slow source").pipe(delay(3000)); // Emits 'Slow source' after 3 seconds

  const winnerObservable = race(observable1, observable2, observable3);

  winnerObservable.subscribe((value) => {
    console.log(value); // Output: Will likely be 0 or 1 (from observable1 or observable2)
  });
  ```

# distinctUntilChanged and distinctUntilKeyChanged

- Only emit when the current value is different than the last.

- distinctUntilChanged uses `===` comparison by default, object references must match. If you want to compare based on an object property, you can use distinctUntilKeyChanged instead.

- if the types dont't match (comparing 1 with '1'), both values will be emitted since they are differnt

  ```js
  import { distinctUntilChanged } from "rxjs";

  const source$ = of(1, "1", 2, 2, 3, 3, 3, 4, 5, 3);
  source$.pipe(distinctUntilChanged()).subscibe((val) => console.log); // 1 1 2 3 4 5 3
  ```

  ![distinctUntilChanged-diagram](image-26.png)

- in the above example, the string 1 is different than the previous emission (numeric 1) hence it is emitted. And 3 is different than 5 (previous emission wrt to 3) hence 3 is also printed.

  ```js
  import { scan, distinctUntilChanged, from } from "rxjs";

  const users = [
    { name: "Brian", loggedIn: false, token: null },
    { name: "Brian", loggedIn: true, token: "abc" },
    { name: "Brian", loggedIn: true, token: "xyz" },
  ];

  const usersList$ = from(users);

  usersList$
    .pipe(
      scan((acc, nxt) => {
        return { ...acc, ...nxt };
      }, {}),
      distinctUntilChanged((prev, curr) => prev.name === curr.name)
    )
    .subscribe((val) => console.log(val)); // { name: "Brian", loggedIn: false, token: null }
  ```

  ```js
  import { scan, distinctUntilKeyChanged, from } from "rxjs";

  const users = [
    { name: "Brian", loggedIn: false, token: null },
    { name: "Brian", loggedIn: true, token: "abc" },
    { name: "Brian", loggedIn: true, token: "xyz" },
  ];

  const usersList$ = from(users);

  usersList$
    .pipe(
      scan((acc, nxt) => {
        return { ...acc, ...nxt };
      }, {}),
      distinctUntilKeyChanged("name")
    )
    .subscribe((val) => console.log(val)); // { name: "Brian", loggedIn: false, token: null }
  ```
