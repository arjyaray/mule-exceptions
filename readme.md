# MuleSoft Exception Handling Scenarios

---

> 📝 **Note:** Scenarios **14, 15, 18, and 19** do not require a practical Mule application to be built.
> You can try the DataWeave expressions directly in the **[DataWeave Playground](https://developer.mulesoft.com/learn/dataweave/)**!

---

## Scenario 1

**A Mule flow contains multiple sequential processors. An error occurs in the middle of execution, and the flow has an On Error Continue handler.**

**Will the processors after the error execute?**
> No. Once an error occurs, Mule immediately stops normal flow execution. Control goes to the Error Handler. Any processors after the failure point are skipped.

**What happens after the error handler completes?**
> On Error Continue → Flow is treated as successful. A response is returned from the error handler.

**Does the flow resume from the failure point or terminate?**
> Mule does NOT resume from where it failed.

---

## Scenario 2

**A main flow calls a subflow. The subflow throws an error, and the subflow has its own On Error Continue handler.**

**Will the remaining processors inside the subflow execute?**
> No, it will go to error handler and skip the after processes just like before.

**Will control return to the main flow?**
> Yes, with a success response.

**Will the main flow continue execution after the subflow call?**
> Yes, main flow sees no failure.

---

## Scenario 3

**Same as Scenario 2, but the subflow does NOT have an error handler. The main flow has On Error Continue.**

**Where is the error handled?**
> In the main flow — error is propagated automatically to the parent flow.

**Will the main flow continue execution after the subflow call?**
> No, remaining processes after the flow reference are skipped.

**Does Mule resume execution after the failed component?**
> No, Mule never resumes processing after a failure point.

---

## Scenario 4

**A flow has an error handler with On Error Propagate.**

**What happens to the error after it is caught?**
> The error is handled by the error handler scope — it executes its logic there, then re-throws the error again.

**If this flow is called by another flow, what happens?**
> It re-throws the error to the parent flow.

**What happens if the parent flow does not have an error handler?**
> If the parent flow doesn't have an error handler, the flow will crash.

---

## Scenario 5

**Compare: one flow using On Error Continue vs. another using On Error Propagate.**

**How does execution differ?**
> - **On Error Continue:** Consumes the error and continues processing any remaining processors.
> - **On Error Propagate:** Re-throws the error and skips any remaining processors.

**What happens to the error object in each case?**
> - **On Error Continue:** Error is consumed and sent as a normal response.
> - **On Error Propagate:** Error is propagated and results in an error response.

**Which one should be used in business vs. system errors?**
> - **Business Errors** → Use `On Error Continue` (expected/handled errors).
> - **System Errors** → Use `On Error Propagate` (unexpected failures).

---

## Scenario 6

**A Try scope is used inside a flow. An error occurs inside the Try block, and it has an On Error Continue.**

**Will the flow outside the Try block continue?**
> Yes, the error is handled inside the Try block and the outer flow proceeds normally.

**Will processors after the error inside Try execute?**
> No — once an error occurs inside the Try block, remaining processors within the block are skipped.

**How is Try scope different from flow-level error handling?**
> A Try scope only affects a specific portion of the flow, not the entire flow.

---

## Scenario 7

**Nested behavior: Try scope inside a flow. Try uses On Error Propagate. Flow has On Error Continue.**

**Which handler executes first?**
> The Try scope handler executes first and triggers On Error Propagate.

**Does the error reach the flow level?**
> Yes — the error propagates from the Try scope to the main flow, where the flow-level error handler receives it.

**Final outcome of execution?**
> On Error Continue at the flow level returns a normal response, but execution stops after the Try block.

---

## Scenario 8

**You define a Global Error Handler and reference it in a flow.**

**When does the global error handler execute?**
> When a flow has an error and it is not handled locally — the error is then delegated to the global error handler.

**What happens if the flow has its own error handler?**
> If the local handler handles the error with On Error Continue, the global handler is ignored.

**Which one takes precedence?**
> `Local (Flow Level)` → takes precedence over `Global Level`.

---

## Scenario 9

**A flow uses a Local Error Handler with On Error Propagate. A Global Error Handler also exists.**

**Will the global error handler execute?**
> Yes — since the flow propagates (re-throws) the error, the global error handler receives it.

**What is the execution order?**
> `Flow Error Handler (Local)` → `Global Error Handler` → `Default Mule Error`

**What happens if the global handler also uses On Error Propagate?**
> The error is re-thrown outside of the flow and the API returns an error response (API failure).

---

## Scenario 10

**A connector inside Until Successful fails repeatedly.**

**How many times will Mule retry?**
> Based on the `maxRetries` configuration of the Until Successful scope.

**What happens after max retries?**
> If all retries fail, Mule throws a `MULE:RETRY_EXHAUSTED` error.

**Does the flow continue or fail?**
> Depends on the error handler — the flow fails if the error is handled by On Error Propagate.

---

## Scenario 11

**Until Successful is wrapped inside a Try scope.**

**What happens when retries are exhausted?**
> If all retries fail, it throws `MULE:RETRY_EXHAUSTED` error.

**Can Try scope catch this failure?**
> Yes — if there is an error handler inside the Try scope, it can catch this error.

**What happens if Try uses On Error Continue?**
> The error is consumed inside the Try block, and processing continues with the processors that come after the Try block.

---

## Scenario 12

**You explicitly raise an error using the Raise Error component.**

**How is this different from system errors?**

| | Raise Error | System Errors |
|---|---|---|
| Trigger | Manually triggered | Automatically thrown by Mule/connectors |
| Use case | Business logic failures | e.g., DB failure, HTTP timeout |
| Control | Fully controlled by developer | Not directly controlled |

**Can you control error types?**
> Yes — you can define custom error types using the `Raise Error` component.

**How do you handle only specific error types?**
> By using the `type` configuration in the error handler. By default, it catches `ANY` type.

---

## Scenario 13

**Raise multiple error types: BUSINESS error and SYSTEM error.**

**How will you handle them differently?**
> - **Business Errors** → Handle with `On Error Continue` since they are expected errors.
> - **System Errors** → Handle with `On Error Propagate` since they are unexpected failures.

**What happens if no matching handler exists?**
> The flow and the API fail, throwing the default error type with an HTTP `500` status code.

---

## Scenario 14


**A DataWeave script fails due to missing fields.**

**What type of error is thrown?**
> Usually `MULE:EXPRESSION` is thrown when DataWeave fails (null access, missing field, type mismatch).

**How can you prevent failure using default values?**
> By using the `default` operator.
> ```dataweave
> payload.name default "Unknown"
> ```

**How do you safely handle nulls?**
> By using the `?` safe navigator operator — it returns `null` instead of an error.

---

## Scenario 15


**Handling errors inside DataWeave.**

**How can you use `try` in DataWeave?**
> Syntax: `try(() -> expression)` — it passes a lambda function and returns an object instead of a direct value.

> Return structure:
> ```json
> {
>   "success": true/false,
>   "result": value,
>   "error": { ... }
> }
> ```

**What happens when a transformation fails?**
> Without handling, the flow fails and returns a `MULE:EXPRESSION` error.

**Can DataWeave suppress errors?**
> Yes — by using `try` or the `default` operator.

---

## Scenario 16

**You have a Try scope, a flow error handler, and a global error handler — all present in one application.**

**What is the order of execution?**
> `Try Scope` → `Flow Error Handler` → `Global Error Handler` → `Default Mule Error`

**Which handler gets priority?**
> `Try Scope` > `Flow Error Handler` > `Global Error Handler`

**Under what condition does the global handler execute?**
> When the error is not handled at either the Try scope level or the flow level (or when On Error Propagate is used at those levels).

---

## Scenario 17

**Using Scatter-Gather where one route succeeds and one route fails.**

**What is the final output?**
> Scatter-Gather returns an aggregated result (if handled), otherwise it returns an error object.

**Does Mule fail the entire flow?**
> Yes — by default, if even one route fails, the entire flow fails.

**How are partial failures handled?**
> By using a Try scope inside each route, or by aggregating success and failure info in the flow-level error handler.

---

## Scenario 18


**Global Error Handler behavior.**

**When does the global handler execute?**
> When the error is not handled at the flow level — either because there is no local handler, or the local handler uses On Error Propagate.

**Does a local handler override the global?**
> Yes — if a local error handler handles it with On Error Continue, the global error handler is ignored.

**What is the handler precedence?**
> `Try Scope` → `Flow Error Handler` → `Global Error Handler` → `Default Mule Error`

---

## Scenario 19


**Local Propagate + Global Handler.**

**Will the global handler execute?**
> Yes — since the local error handler uses On Error Propagate, it re-throws the error to the global error handler.

**What is the execution order?**
> `Flow Level Error Handler` → propagates → `Global Error Handler`

**What if the global handler also propagates?**
> If the global error handler also propagates, it re-throws the error again and ultimately the flow fails with an API error response.