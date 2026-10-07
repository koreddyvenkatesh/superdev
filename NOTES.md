### Summary of changes
I tried to focus on the bugs that broke the core logic or user experience the most:
1. **The SQL bug:** I noticed `TaskRepository` was missing parentheses around the `OR` condition. Because `AND` takes priority, searching by description was accidentally bypassing the `archived = FALSE` check. I added brackets to fix the grouping.
2. **Slow API:** I found a `Thread.sleep()` block in `TaskController` that was intentionally punishing short search queries. I just deleted it so the API is fast again.
3. **500 Errors on Status:** If a bad status string was passed in the URL, the app crashed. I wrapped `TaskStatus.valueOf()` in a try-catch to handle it safely.
4. **Blank pages on search:** In `App.jsx`, doing a new search while on page 3 would result in a blank table. I updated the state handlers to reset the page to 1 whenever the query or filter changes.
5. **React race conditions & infinite loading:** In `useTasks.js`, typing fast caused older, slower API responses to overwrite newer ones. I added a cleanup flag to ignore out-of-order responses. I also added a `.finally()` block so the loading spinner actually stops if the network fails.

### What I chose not to change and why
I saw that pagination is handled in-memory using `allResults.subList()` instead of at the database level. I decided not to touch it because swapping to Spring Data's `Pageable` would require changing the repository, controller, and frontend mapping, which felt way too risky for a 90-minute timebox.

### The biggest remaining risk
Definitely the in-memory pagination. Pulling every single matching row from H2 into Java memory just to show 10 items is a massive bottleneck. Once the task table gets big enough, this will cause an `OutOfMemoryError` and crash the backend.

