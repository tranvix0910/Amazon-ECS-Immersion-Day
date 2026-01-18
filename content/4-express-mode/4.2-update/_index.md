---
title : "Express Mode Update"
date :  2025-12-15
weight : 2
chapter : false
pre : " <b> 4.2. </b> "  
---

#### Update Service

In this section, we will update service ui-express-mode by changing the default application interface to orange color through updating environment variable.

```bash
"environment": [{
  "name": "RETAIL_UI_THEME",
  "value": "orange"
}]
```

Run the following command to update Service `ui-express-mode`.

```bash
aws ecs update-express-gateway-service \
    --service-arn arn:aws:ecs:${AWS_REGION}:${ACCOUNT_ID}:service/retail-store-ecs-cluster/ui-express-mode \
    --primary-container '{
        "image": "public.ecr.aws/aws-containers/retail-store-sample-ui:1.2.3",
        "environment": [{
          "name": "RETAIL_UI_THEME",
          "value": "orange"
        }]
    }'
```

![Update Service ui-express-mode](/images/4-express-mode/4.2-update/1.png)

Open Amazon ECS Console:

- Select Deployments tab.

- Select the deployment in progress.

- View deployment process details.

![Update Service ui-express-mode](/images/4-express-mode/4.2-update/2.png)

![Update Service ui-express-mode](/images/4-express-mode/4.2-update/3.png)

Amazon ECS Express Mode uses ECS **canary deployment strategy**.

This mechanism works as follows:

- Initially, a small percentage of traffic will be routed to the new revision for testing.

- After the canary phase completes successfully, all remaining traffic will be routed to the new revision in one go.

![Update Service ui-express-mode](/images/4-express-mode/4.2-update/4.png)


#### Check application

Get Application Load Balancer DNS:

```bash
ECS_ALB_DNS=$(aws ecs describe-express-gateway-service \
  --service-arn arn:aws:ecs:${AWS_REGION}:${ACCOUNT_ID}:service/retail-store-ecs-cluster/ui-express-mode \
  --query service.activeConfigurations[0].ingressPaths[0].endpoint \
  --output text)

echo "https://${ECS_ALB_DNS}"
```

![Get Application Load Balancer DNS](/images/4-express-mode/4.2-update/5.png)

Proceed to access the application using DNS:

![Access application using DNS](/images/4-express-mode/4.2-update/6.png)
