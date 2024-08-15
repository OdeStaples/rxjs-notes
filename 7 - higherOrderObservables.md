# Higher Order Observables

- These observables emit other observables! This creates a nested structure where the outer observable acts like a container for the inner observables.

  ```js
  import { delay, of, map } from "rxjs";

  const example = of(1, 2, 3).pipe(
    map((val) => of(val).pipe(delay(val * 1000)))
  );

  example.subscribe(console.log);
  // Observable {source: Observable {...}, operator:...}
  // Observable {source: Observable {...}, operator:...} -->
  // Observable {source: Observable {...}, operator:...}
  ```

  ```js
  import { fromFetch, map, fromEvent } from "rxjs";

  const example$ = fromEvent(btnClick, "click").pipe(
    map(() => fromFetch("api/important-stuff"))
  );
  ```

### mergeAll

- takes a stream of branch observables and merges them into the main timeline.

  ```js
  import { delay, of, map, mergeAll } from "rxjs";

  const example = of(1, 2, 3).pipe(
    map((val) => of(val).pipe(delay(val * 1000))),
    mergeAll()
  );

  example.subscribe((val) => console.log(val)); // prints 1,2,3 after 1,2,3 seconds respectively
  ```

  - Flattens an Observable-of-Observables.
    ![mergeAll-image](https://rxjs.dev/assets/images/marble-diagrams/mergeAll.png)

### mergeMap

- The `mergeMap` operator in RxJS is a transformation operator that is used to map each value emitted by an observable into another observable, and then merge the output of these resulting observables into a single observable stream.

- Here’s a high-level overview of how mergeMap works:

  - **Mapping**: For each value emitted by the source observable, mergeMap applies a projection function you provide.

  - **Merging**: It then merges the observables returned by the projection function into one output observable.

  - **Concurrency**: By default, mergeMap subscribes to all inner observables immediately. However, you can limit the number of concurrent subscriptions using the concurrent parameter.

- Key Characteristics:

  - **Flat Mapping**: It’s often referred to as a “flatMap” because it flattens multiple observables into one.

  - **Order**: The order of emissions is not guaranteed to be preserved, as it emits values as soon as they are available from the inner observables.

  - **Use Cases**: It’s useful when you have tasks that can run in parallel and when the order of results doesn’t matter.

  - **Caution**: Since mergeMap can handle multiple active inner subscriptions at once, it’s possible to create a memory leak if the inner observables are long-lived or infinite.

  ```js
  import { of, mergeMap, interval, map } from "rxjs";

  const letters = of("a", "b", "c");
  const result = letters.pipe(
    mergeMap((letter) => interval(1000).pipe(map((i) => letter + i)))
  );

  result.subscribe((value) => console.log(value)); // a0, b0, c0, a1, b1, c1, ...
  ```

  - In this example, `mergeMap` takes each letter from the letters observable and maps it to an interval observable that emits a number every second. It then concatenates the letter with the number and emits the result. The output would be a continuous stream of values like ‘a0’, ‘b0’, ‘c0’, ‘a1’, ‘b1’, ‘c1’, and so on.

  ```js
  import { delay, of, map, mergeMap } from "rxjs";

  const example$ = of(1, 2, 3, 4)
    .pipe(
      map((val) => of(val).pipe(delay(val * 1000))),
      mergeMap((val) => val)
    )
    .subscribe((val) => console.log(val)); // prints 1,2,3,4 after 1,2,3,4 seconds respectively
  ```

  - Maps each value to an Observable, then flattens all of these inner Observables using _mergeAll_.
    ![mergeMap-Image](https://rxjs.dev/assets/images/marble-diagrams/mergeMap.png)
