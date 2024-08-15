# What are combination operators

- these operators join multiple observables into a single stream (eg. combine multiple requests, perform calculations based on multiple stream inputs, tracking on values to the beginning or the end of a stream)

# startWith

- the `startWith` operator appends specified value or values to a stream emitting them first on subscription.

- In the below diagram, we are supplying a value of 0 using the startWith operator. On subscription, this value (0) will be emitted first immediately, followed by any other values emitted by the input observable.

  ![startWith-diagram](image-12.png)

  ```js
  import { of, startWith } from "rxjs";

  const num$ = of(1, 2, 3);
  num$.pipe(startWith("a", "b", "c")).subscribe(console.log); // a b c 1 2 3
  ```

  ```js
  import {
    interval,
    scan,
    map,
    fromEvent,
    takeUntil,
    takeWhile,
    startWith,
  } from "rxjs";

  const countdown = document.getElementById("countdown");
  const message = document.getElementById("message");
  const abort = document.getElementById("abort");

  const interval$ = interval(1000);
  const abortBtn$ = fromEvent(abort, "click");

  const TIMER_START = 30;

  interval$
    .pipe(
      map(() => -1),
      scan((acc, nxt) => {
        return acc + nxt;
      }, TIMER_START),
      takeWhile((val) => val >= 0),
      takeUntil(abortBtn$),
      startWith(TIMER_START)
    )
    .subscribe((val) => {
      countdown.innerText = val;
      if (!val) {
        message.innerText = "Takeoff";
        return;
      }
    });
  ```

# endWith

- the `endWith` operator appends specified value or values to the end of a stream

  ```js
  import { of, startWith, endWith } from "rxjs";

  const num$ = of(1, 2, 3);
  num$
    .pipe(startWith("a", "b", "c"), endWith("c", "d", "e"))
    .subscribe(console.log); // a b c 1 2 3 c d e
  ```

# concat

- the `concat` creation operator lets you create an observable from a variable number of other observable you supply. On subscription, `concat` will subscribe to the inner observables in order - so as the first completes, the second is subscribed to and so on.

- whenever you have multiple observables you want to execute in order, you can use `concat`

- any values emitted by the inner observables are emitted by `concat`. In the below example, `concat` subscribe to obs1$ first. On completion of obs1$, concat will then subscribe to obs2$ and so on if more observables were supplied.

  ![concat-diagram](image-13.png)

# merge
