# Promise vs Observables

- A Promise is an eventual value. An observable is a series of eventual values.

# Use cases of Observables

- Promises work great with happy path. But whar if you want to retry a certain number of times? What happens when you want to reload the data at some regular interval? What happens when you come across a paginated API (When an API endpoint returns a large amount of data, pagination allows the data to be divided into smaller, more manageable chunks or pages. Each page contains a limited number of records or entries.) but also show data as it comes in? Observables can help with that.

- You might come across an issue where you have multiple APIs that you need to coordinate in order to display a given piece of UI. This can be complicated and error prone. Observables can help with that.

- Figuring out when to show a spinner - when an API request is taking too long, it can be helpful to give users a visual feedback. If the API response is fast you don't want the immediately flash the spinner only to remove it 5ms later.Observables can help with that.

- Dragging and dropping feature can be implemented with help of observables.
