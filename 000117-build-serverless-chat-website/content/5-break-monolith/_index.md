---
title : "Break Up the Monolith"
weight : 5
chapter : false
pre : " <b> 5. </b> "
---

`Chat-Proxy` does everything in one function: it checks the path and the HTTP method itself. In this part you replace it with one small function per API method, describe the API with real resources, and let API Gateway generate a JavaScript client for the website.
