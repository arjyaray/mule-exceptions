## MuleSoft Exception Handling Scenarios

---

### Scenario 1
**A Mule flow contains multiple sequential processors. An error occurs in the middle of execution, and the flow has an On Error Continue handler.**

- **Will the processors after the error execute?**
No. Once an error occurs, Mule immediately stops normal flow execution. Control goes to the Error Handler. Any processors after the failure point are skipped.

- **What happens after the error handler completes?**
On Error Continue-> Flow is treated as successful. A response is returned from the error handler.

- **Does the flow resume from the failure point or terminate?**
Mule does NOT resume from where it failed.

---

### Scenario 2
**A main flow calls a subflow. The subflow throws an error, and the subflow has its own On Error Continue handler.**

- **Will the remaining processors inside the subflow execute?**
No, it will go to error handler and skip the after processes just like before.
- **Will control return to the main flow?**
Yes, with sucess response
- **Will the main flow continue execution after the subflow call?**
Yes, main flow sees no failure

---

### Scenario 3
**Same as Scenario 2, but the subflow does NOT have an error handler. The main flow has On Error Continue.**

- **Where is the error handled?**
In the main flow, error is propagated automatically to the parent flow.
- **Will the main flow continue execution after the subflow call?**
No, remaining process after flow reference are skipped.
- **Does Mule resume execution after the failed component?**
No, mule never resume process after failure point.

---

### Scenario 4
**A flow has an error handler with On Error Propagate.**

- **What happens to the error after it is caught?**
The error is handled by the error handler scope, it executes it logic there then it re throws the error again.
- **If this flow is called by another flow, what happens?**
It re-throw the error to the parent flow.
- **What happens if the parent flow does not have an error handler?**
If the parent flow doesnt have error handler then it will crash the flow.

---

### Scenario 5
**Compare: one flow using On Error Continue vs. another using On Error Propagate.**

- **How does execution differ?**
-- On error continue flow consumes the error and continues processing the remaining process.
-- On error propagate flow re-throws the error and skips the remaining process.
- **What happens to the error object in each case?**
-- error is consumed and sent as normal response in continue whereas error is propagated and throws a error response in on error propagate.
- **Which one should be used in business vs. system errors?**
On error continue should be used in business errors and on error propagate should be used in system errors.

---

### Scenario 6
**A Try scope is used inside a flow. An error occurs inside the Try block, and it has an On Error Continue.**

- **Will the flow outside the Try block continue?**
Yes, error is handled inside try block.
- **Will processors after the error inside Try execute?**
No, since error occured.. everything else inside the block will be skipped.
- **How is Try scope different from flow-level error handling?**
try only affects a part of flow.

---

### Scenario 7
**Nested behavior: Try scope inside a flow. Try uses On Error Propagate. Flow has On Error Continue.**

- **Which handler executes first?**
try scope handler executes first and triggers on error propagate.
- **Does the error reach the flow level?**
Yes, error propagates from try to main flow then flow level error handler recieves it.
- **Final outcome of execution?**
On error continue throws normal response but the flow stops after try block.

---

### Scenario 8
**You define a Global Error Handler and reference it in a flow.**

- **When does the global error handler execute?**
When a flow has error and it doesnt handled locally, it refers to the global error handler.
- **What happens if the flow has its own error handler?**
When the local handler handles it with on error continue, the global handles gets ignored.
- **Which one takes precedence?**
local level(flow level) > global level
---

### Scenario 9
**A flow uses a Local Error Handler with On Error Propagate. A Global Error Handler also exists.**

- **Will the global error handler execute?**
Yes, since the flow propagates(re-throws) the error, global error handler recieves it.
- **What is the execution order?**
Flow error handler(local) -> Global error handler -> Default mule error
- **What happens if the global handler also uses On Error Propagate?**
Then error re throws outside of the flow and the API throws error response.(API failure)

---

### Scenario 10
**A connector inside Until Successful fails repeatedly.**

- **How many times will Mule retry?**
Based on ```maxRetries``` config of ubtil successful scope.
- **What happens after max retries?**
If all retries fail, it throw ```MULE:RETRY_EXHAUSTED``` error.
- **Does the flow continue or fail?**
Depends on error handler, flow fails if it is handled by on error propagate.
---

### Scenario 11
**Until Successful is wrapped inside a Try scope.**

- **What happens when retries are exhausted?**
If all retries fail, it throw ```MULE:RETRY_EXHAUSTED``` error.
- **Can Try scope catch this failure?**
Yes, if we use a error handler in try scope, it can catch it.
- **What happens if Try uses On Error Continue?**
Error is consumed inside try block and is sent as normal response after the processes that comes after try block.

---

### Scenario 12
**You explicitly raise an error using the Raise Error component.**

- **How is this different from system errors?**
```Raise Error:``` Manually triggered; Used for business logic failures; Fully controlled by developer
```System Errors:``` Automatically thrown by Mule/connectors; Examples: DB failure,HTTP timeout,Not directly controlled
- **Can you control error types?**
Yes, we can define error types by using ```raise error``` component
- **How do you handle only specific error types?**
By using the ```type``` config in error handler, by default it is ```ANY``` type.

---

### Scenario 13
**Raise multiple error types: BUSINESS error and SYSTEM error.**

- **How will you handle them differently?**
```Business Errors``` should be handled by on error continue since they are expected errors.
```System errors``` should be handled by on error propagate since they are unexpected failures.
- **What happens if no matching handler exists?**
Then flow and the API fails and it throws default error type with HTTP 500 status code.

---

### Scenario 14
**A DataWeave script fails due to missing fields.**

- **What type of error is thrown?**
Usually ```MULE:EXPRESSION``` error is thrown when when DataWeave fails (null access, missing field, type mismatch).
- **How can you prevent failure using default values?**
By using the ```default``` operator.
```e.g: payload.name default "Unknown"```
- **How do you safely handle nulls?**
By using ```?``` safe navigator operator. It returns "null" instead of an error in the dw
---

### Scenario 15
**Handling errors inside DataWeave.**

- **How can you use `try` in DataWeave?**
`Syntax: try(() -> expression)`, It passes a lambda function  () -> expression & it returns and object instead of direct value.
Return structure: `{
  "success": true/false,
    "result": value (if success),
      "error": {...} (if failure)
}`

- **What happens when a transformation fails?**
Without handling, it the flow fails and it returns `MULE:EXPRESSION` error.
- **Can DataWeave suppress errors?**
Yes, by using `try +/default` .
---

### Scenario 16
**You have a Try scope, a flow error handler, and a global error handler — all present in one application.**

- **What is the order of execution?**
Try Scope -> Flow Error Handler -> Global Error Handler -> Default Mule Error
- **Which handler gets priority?**
Try Scope > Flow Error Handler > Global Error Handler
- **Under what condition does the global handler execute?**
When error is not handled before it i.e. not in the flow level neither it is using propagate.
---

### Scenario 17
**Using Scatter-Gather where one route succeeds and one route fails.**

- **What is the final output?**
Scatter-gather returns aggregated result (if handled) OR it returns error object
- **Does Mule fail the entire flow?**
If not handled and even if one route fails, the entire flow fails by default.
- **How are partial failures handled?**
We can do that by using try scope inside each route or it can by done by aggregating the success and failure info in the flow level error handler.

---

### Scenario 18: Global Error Handler

- **When does the global handler execute?**
When error is not handled before it i.e. not in the flow level neither it is using propagate.
- **Does a local handler override the global?**
Yes, if a local error handler handles it with on error continue then it ignore the global error handler.
- **What is the handler precedence?**
Try Scope -> Flow Error Handler -> Global Error Handler -> Default Mule Error

---

### Scenario 19: Local Propagate + Global Handler

- **Will the global handler execute?**
Yes, since the local error handler uses on error propagate then it will rethrow the error to the global level error handler.
- **What is the execution order?**
Flow level error handler -> propagates -> global error handler
- **What if the global handler also propagates?**
If the global error handler also propagates then it rethrows the error again and ultimately the flow fails.