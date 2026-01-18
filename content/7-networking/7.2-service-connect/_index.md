---
title : "ECS Service Connect"
date :  2025-01-01
weight : 2
chapter : false
pre : " <b> 7.2. </b> "  
---

**ECS Service Connect** is the recommended method for handling communication between services (service-to-service communication), providing features such as service discovery, connectivity, and traffic monitoring. 

With Service Connect, your applications can use short names and standard ports to connect to ECS services in the same cluster, different clusters, different VPCs, and even between AWS Accounts in the same AWS Region. [For more detailed information, please refer to AWS Documentation](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/networking-connecting-services.html#networking-connecting-services-serviceconnect).

{{< youtube 4xU7AvjVlIY >}}

Alternatives to configure inter-service communication in Amazon ECS Services include:

- [Internal Load Balancer](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/networking-connecting-services.html#networking-connecting-services-elb)

- [Service Discovery](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/networking-connecting-services.html#networking-connecting-services-direct)

- [Amazon VPC Lattice](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-vpc-lattice.html)

