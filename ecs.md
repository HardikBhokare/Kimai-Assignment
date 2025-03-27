# Deploying Kimai on AWS ECS with RDS

This guide provides step-by-step instructions to deploy the **Kimai** time-tracking application on **AWS ECS (Fargate)** with **Amazon RDS (MySQL/PostgreSQL)**.

## **1. Prerequisites**

Before starting, ensure you have:

- An **AWS account** with IAM permissions to manage ECS, RDS, and ALB.
- AWS CLI installed and configured with necessary permissions.
- An **Amazon RDS (MySQL/PostgreSQL) instance** created.
- An **Amazon ECS cluster** set up.
- A **VPC with private subnets** for RDS and ECS tasks.
- An **Application Load Balancer (ALB)** created.
- AWS **Secrets Manager** to securely store database credentials.

## **2. Create an ECS Task Definition**

Save the following content as `kimai-task-def.json`:

```json
{
  "family": "kimai-task",
  "executionRoleArn": "arn:aws:iam::ACCOUNT_ID:role/ecsTaskExecutionRole",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "containerDefinitions": [
    {
      "name": "kimai",
      "image": "ghcr.io/kimai/kimai2:latest",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 8001,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {
          "name": "DATABASE_URL",
          "value": "mysql://kimai_user:password@rds-hostname:3306/kimai_db"
        },
        {
          "name": "TRUSTED_HOSTS",
          "value": "^your-domain.com$"
        },
        {
          "name": "APP_ENV",
          "value": "prod"
        }
      ],
      "secrets": [
        {
          "name": "DATABASE_URL",
          "valueFrom": "arn:aws:secretsmanager:region:ACCOUNT_ID:secret:kimai-db-credentials"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/kimai",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

### **Register the Task Definition**
Run the following command to register the task definition:
```bash
aws ecs register-task-definition --cli-input-json file://kimai-task-def.json
```

## **3. Create an ECS Service**

Save the following content as `kimai-service-def.json`:

```json
{
  "serviceName": "kimai-service",
  "cluster": "kimai-cluster",
  "taskDefinition": "kimai-task",
  "desiredCount": 2,
  "launchType": "FARGATE",
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": ["subnet-xxxxxxxx", "subnet-yyyyyyyy"],
      "securityGroups": ["sg-xxxxxxxx"],
      "assignPublicIp": "DISABLED"
    }
  },
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:region:ACCOUNT_ID:targetgroup/kimai-target-group/xxxxxxxxxxxx",
      "containerName": "kimai",
      "containerPort": 8001
    }
  ],
  "healthCheckGracePeriodSeconds": 60,
  "deploymentController": {
    "type": "ECS"
  }
}
```

### **Create the ECS Service**
Run the following command to create the ECS service:
```bash
aws ecs create-service --cli-input-json file://kimai-service-def.json
```

## **4. Configure AWS Load Balancer**
1. Ensure an **Application Load Balancer (ALB)** is set up with an HTTPS listener.
2. Attach the **Kimai ECS service** to the ALB **target group**.
3. Update security groups to allow traffic from the ALB.

## **5. Scaling and Monitoring**
- **Enable Auto Scaling** for ECS service:
  ```bash
  aws application-autoscaling register-scalable-target \
    --service-namespace ecs \
    --scalable-dimension ecs:service:DesiredCount \
    --resource-id service/kimai-cluster/kimai-service \
    --min-capacity 2 --max-capacity 5
  ```
- **Monitor logs in CloudWatch**:
  ```bash
  aws logs tail /ecs/kimai --follow
  ```

## **6. Security Best Practices**
- Store **database credentials** in **AWS Secrets Manager**.
- Use **IAM roles** with **least privilege access**.
- Ensure **RDS is in a private subnet**.
- Enable **automatic backups** for RDS.
- Use **AWS WAF** to secure ALB.

## **7. Next Steps**
- Configure **Kimai admin user**.
- Set up **automated backups for RDS**.
- Enable **CloudTrail for security monitoring**.

---
🚀 **Kimai is now running on AWS ECS with best practices!** 🚀
