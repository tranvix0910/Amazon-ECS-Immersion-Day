---
title : "Networking"
date :  2025-01-01
weight : 7
chapter : false
pre : " <b> 7. </b> "  
---

The current application is built from multiple distributed components (microservices) communicating with each other. For example, the UI component communicates via API with the Checkout component, which is linked to a persistent storage layer using Redis, as illustrated in the architecture diagram below. At the same time, both UI and Checkout communicate via API with the Orders service, which uses PostgreSQL as the data storage layer.

![Architecture](/images/7-networking/1.png)

In this section we will learn about **ECS Networking** and how it works related to Fargate.
