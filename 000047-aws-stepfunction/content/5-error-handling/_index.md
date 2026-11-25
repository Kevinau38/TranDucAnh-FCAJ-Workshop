---
title : "Handle errors"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

So far the workflow cannot cope with errors. By default, when a state fails, the whole execution fails. Errors can come from:

- the state machine definition (for example a Choice state with no matching rule),
- a task (for example an exception thrown by a Lambda function),
- temporary problems (for example the network or Lambda throttling).

Step Functions has two tools for this: **Retry** runs a failed state again, **Catch** moves to another state when a specific error happens.
