# interval

- creates an observable that emits an event every `n` miliseconds.

  ```ts
  import { interval } from "rxjs";

  const startingTime = Date.now();
  const tick$ = interval(1000);

  tick$.subscribe(function findTimeDifference() {
    console.log(Date.now() - startingTime); // logs 1489 2496 3487 4493 5488
  });
  ```

# timer

- creates an observable that emits after a certain amount of time.

  ```ts
  import { timer } from "rxjs";

  const startingTime = Date.now();
  const tick$ = timer(5000);

  tick$.subscribe(function calculateTimeDifference() {
    console.log(Date.now() - startingTime);
  });
  ```

- timer can also be used to emit at regular intervals

  ```ts
  import { timer } from "rxjs";

  const startingTime = Date.now();
  const tick$ = timer(1000, 5000); // emits the first value at 1s and after then emits value after 5s each

  tick$.subscribe(function calculateTimeDifference() {
    console.log(Date.now() - startingTime); // logs 1399 7400 12402 17404 22404
  });
  ```

# Unsubscribing

- Allows to clean up when you don't want to emit any more values.

  ```ts
  const startingTime = Date.now();
  const tick$ = timer(1000, 5000);

  const subscription = tick$.subscribe(function calculateTimeDifference() {
    console.log(Date.now() - startingTime); // logs 1115, 7115
  });

  setTimeout(function stopObservervable() {
    subscription.unsubscribe();
    console.log("Unsubscribed!"); // Unsubscribed
  }, 7000);
  ```
