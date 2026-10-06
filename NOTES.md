# Bug Fix Notes

## 1. Search and Status Filter

**Issue:** Search results were not consistently respecting the `status` and `archived` conditions.

**How found:** I tested search with different status filters and inspected the native SQL query.

**Root cause:** SQL `AND` has higher precedence than `OR`, so the conditions were not grouped correctly.

**Fix:** Added parentheses around the title/description search conditions so the `archived` and `status` filters apply to the complete search condition.

**Result:** Search now correctly filters non-archived tasks by the selected status.

## 2. Artificial API Delay

**Issue:** Task search/pagination requests were unnecessarily slow.

**How found:** While testing the API, I noticed consistent delays and found a `Thread.sleep()` in `TaskController`.

**Root cause:** The controller intentionally added a delay based on query length.

**Fix:** Removed the artificial delay and the unused complexity calculation.

**Result:** API responses are no longer artificially delayed.

## 3. Invalid Request Parameters

**Issue:** Invalid page numbers and status values could result in server errors.

**How found:** Tested `page=0`, negative page values, and an invalid status.

**Root cause:** Page validation allowed `page=0`, and invalid status values caused `IllegalArgumentException`.

**Fix:** Require `page >= 1` and `pageSize > 0`, and return HTTP 400 for invalid status values.

**Result:** Invalid client input is handled safely with a proper Bad Request response.

## 4. Async Request Handling

**Issue:** Rapid searches could cause stale results, and failed requests could leave the loading state active.

**How found:** Reviewed the asynchronous request flow in `useTasks.js`.

**Root cause:** Previous requests were not ignored when a newer request started, and loading was not reset on errors.

**Fix:** Added request cancellation/ignore handling, reset errors when a request starts, and used `finally` to reliably stop loading.

**Result:** Stale responses are ignored and loading/error states remain consistent.

## Future Improvement

Database-level pagination could be introduced instead of fetching all matching tasks and slicing them in the backend. This was left unchanged to keep the exercise focused on the highest-value bugs.
