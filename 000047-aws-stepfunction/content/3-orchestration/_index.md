---
title : "Orchestrate the services"
weight : 3
chapter : false
pre : " <b> 3. </b> "
---

The simplest way to connect the services would be to put the logic inside the Account Applications service: after a new application is stored, call Data Checking twice, then flag or approve the application. It works, but every service then has to handle timeouts, retries and errors on its own, and nobody can see the shape of the process or the history of one application.

**AWS Step Functions** solves this. You describe the workflow as a **state machine**: a set of named states, one start state, and transitions between them. Step Functions runs each step, passes the data from one step to the next, retries when needed, and keeps a visual history of every execution.

In this part you:

1. Create a first state machine with **Pass** states in the console.
2. Replace them with **Task** states that call the Data Checking function.
3. Give the state machine permission to call Lambda, this time with AWS SAM.
4. Control the data passed between states with **ResultPath**.
5. Add a **Choice** state to route applications that need a review.
