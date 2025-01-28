---
title: Page 3
deprecated: false
hidden: false
metadata:
  robots: index
---
Tensor9 maintains a **digital twin** of each **stack** running in a customer **appliance**.  Your digital twin is your window into the operational state of its corresponding customer appliance.  You observe that customer's logs, metrics, and hardware failures through its digital twin, which Tensor9 continuously synchronizes.

In this guide, we’ll walk you through how to observe logs, metrics and hardware failures through a digital twin.

***

## **Prerequisites**

Before you begin, you'll need...

* An **appliance** running your Tensor9 **app**.  If you haven't done this yet, finish this guide first: [Set Up An Appliance](doc:set-up-an-appliance#top)

***

## Mental model

Tensor9 supports all AWS resources supported by CloudFormation.  This means that any AWS resource in your CloudFormation origin stack can be deployed to a customer appliance, and Tensor9 will maintain a digital twin of that resource in your Tensor9 AWS account.  Some AWS resources support **low-fidelity digital twins**, while others support **high-fidelity digital twins**.

* **A low-fidelity digital twin** indicates whether or not its corresponding resource exists inside the customer appliance. This is useful to know whether a deployment to an appliance was successful.
* **A high-fidelity digital twin** offers the same deployment-related information that a low-fidelity digital twin does, but also allows you to observe and affect the real-time operational state of its corresponding resource in an appliance.

### Low-fidelity digital twins

A **low-fidelity digital twin** indicates whether or not its corresponding resource exists inside the customer appliance. This is useful to know whether a deployment to a customer appliance was successful. For example, a **Route 53 Hosted Zone** digital twin is low-fidelity. When you deploy a Route 53 Hosted Zone to a customer appliance, Tensor9 creates a corresponding [CloudFormation custom resource](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/template-custom-resources.html) in your Tensor9 AWS account.

***This is not a real hosted zone.*** It is only a CloudFormation custom resource indicating that the hosted zone was successfully created inside the appliance. If you delete the Route 53 hosted zone digital twin (by deleting its owning CloudFormation stack), then the corresponding real hosted zone inside the appliance will also be deleted.

![](https://files.readme.io/868c32e18c1fd7242e0860465e27a71cc4bc1c5465277912d29db9b1be8d7518-image.png)

<br />

### High-fidelity digital twins

A **high-fidelity digital twin** offers the deployment-related information that a low-fidelity digital twin does, but also allows you to observe and affect the real-time operational state of its corresponding resource in an appliance.

For example, ECS cluster digital twins are high-fidelity.  If you deploy an ECS cluster to an appliance, Tensor9 maintains a corresponding ***real ECS cluster*** in your Tensor9 AWS account that acts as its digital twin.  The ECS cluster digital twin is synchronized with the corresponding service, tasks, containers, logs and metrics in the appliance.  You can log into your AWS console, open the ECS console, see the ECS cluster digital twin there, click on a service, click on a container, and look at the container's logs.  Those container logs will be synchronized with the logs produced by the corresponding container inside the appliance.  If you kill an ECS container digital twin, then the corresponding container in the appliance will also be killed.  Similarly, if a container inside an appliance dies (e.g. due to hardware failure), then the corresponding ECS container digital twin will also die.

It is important to understand that high-fidelity digital twin uses minimal resources.  In the case ECS, the CPU and memory requirements of an ECS task digital twin will be minimally sized - despite the fact that the corresponding ECS task in the appliance may have large CPU and memory requirements.  This keeps your costs infrastructure down: high-fidelity digital twins are designed to be far less expensive relative to their corresponding resources inside the appliance.

![](https://files.readme.io/ad6c621b57a4ce08a30292d5171b5bef403d8c1ffbe65b4def5b9e2956ff1897-image.png)

<br />

***

## Explore your digital twins

Take this opportunity to explore the digital twins Tensor9 created in your Tensor9 AWS account.  This guide doesn't know exactly what those are because it depends upon the composition of your stack.  Below are examples of commonly used stack components - and how their digital twins behave.

| Resource                               | Resource Type    | Fidelity      |
| :------------------------------------- | :--------------- | :------------ |
| AWS CloudFormation Stack               | Infrastructure   | High-Fidelity |
| AWS ECS Cluster                        | Containers       | High-Fidelity |
| AWS EKS Cluster                        | Kubernetes       | High-Fidelity |
| AWS EC2 Instance                       | Virtual Machine  | High-Fidelity |
| AWS Lambda Function                    | Serverless       | High-Fidelity |
| AWS S3 Bucket                          | Blob Storage     | High-Fidelity |
| AWS RDS Database                       | SQL              | High-Fidelity |
| AWS Aurora Database                    | Serverless SQL   | High-Fidelity |
| AWS SQS Queue                          | Queues           | High-Fidelity |
| AWS Kinesis Stream                     | Streaming        | High-Fidelity |
| AWS Managed Streaming for Apache Kafka | Streaming        | High-Fidelity |
| AWS API Gateway                        | Serverless       | High-Fidelity |
| AWS Elastic MapReduce                  | Batch processing | High-Fidelity |
| AWS Batch                              | Batch processing | High-Fidelity |
| All other AWS resources                | Various          | Low-Fidelity  |

<br />

### AWS CloudFormation Stack (Infrastructure) (High-Fidelity)

You'll find a real CloudFormation stack in your Tensor9 AWS account.

This **digital twin stack** corresponds to a stack of infrastructure inside an appliance.  Think of this digital twin stack as the root of a tree of constituent digital twins - each in turn corresponding to constituent AWS resources inside the appliance.

You can explore further by inspecting the resources in this digital twin stack using the AWS CloudFormation console.

### AWS ECS Cluster (Containers) (High-Fidelity)

You'll find a real ECS cluster in your Tensor9 AWS account.

This **ECS cluster digital twin** mirrors the operational state of the ECS cluster running in the customer appliance. Here's what you can do:

* **View Services and Tasks**: Open the ECS console in your Tensor9 AWS account to view the services and tasks associated with the digital twin. These correspond directly to the services and tasks running inside the customer's appliance.
* **Observe Container Logs**: Access individual containers and view their logs. These logs are synchronized in real time with the logs produced by the corresponding containers in the customer appliance.
  * **Log Allow-List**: The customer must explicitly allow-list the logs that can be viewed through the digital twin.
  * **Audit Logging**: All logs viewed through the digital twin are appended to the customer's audit log for full traceability.
* **Monitor Metrics**: Metrics such as CPU and memory usage are synchronized and viewable within the ECS console.
* **Debug Failures**: If a container in the customer appliance crashes due to a failure, its corresponding digital twin in your ECS cluster will also terminate. This allows you to detect container failures without needing direct access to the customer's environment.
* **Perform Actions**: You can restart or terminate ECS tasks through the digital twin. However, any targeted actions, such as restarting or terminating tasks, require explicit customer allow-listing. All actions performed through the digital twin are logged in the customer's audit log.

The ECS cluster digital twin is designed to use minimal resources. For example:

* A container requiring 16 vCPUs and 32 GB RAM in the customer appliance may be represented in the ECS cluster digital twin by a container requiring just 0.25 vCPUs and 512 MB RAM.

This lightweight design keeps costs low while maintaining real-time synchronization of logs and health events.

### AWS EKS Cluster (Kubernetes) (High-Fidelity)

You'll find a real EKS cluster in your Tensor9 AWS account.

This **EKS cluster digital twin** mirrors the operational state of the Kubernetes cluster running in the customer appliance. Here's what you can do:

* **View Namespaces, Pods, and Deployments**: Open the EKS console or use `kubectl` in your Tensor9 AWS account to view namespaces, pods, and deployments. These resources correspond to the Kubernetes components running inside the customer appliance.
  * **Allow-Listed Access**: Customers must explicitly allow-list which namespaces, pods, and deployments are accessible through the digital twin.
  * **Audit Logging**: All access and operations performed through the digital twin are appended to the customer’s audit log for full traceability.
* **Observe Pod Logs**: Access individual pod logs through the EKS console or via `kubectl logs`. These logs are synchronized in real time with the logs produced by the corresponding pods in the customer appliance.
  * **Log Allow-List**: The customer must specify which logs can be accessed through the digital twin.
  * **Audit Logging**: All viewed logs are recorded in the customer’s audit log.
* **Monitor Metrics**: Metrics such as CPU and memory usage for nodes and pods are synchronized and viewable through CloudWatch or other monitoring tools integrated with the EKS cluster.
* **Debug Failures**: If a pod crashes inside the customer appliance, the corresponding pod in the EKS cluster digital twin will also terminate. This allows you to detect and diagnose pod failures without needing direct access to the customer’s environment.
* **Perform Actions**: You can delete, restart, or scale deployments within the EKS cluster digital twin. However, all actions, such as scaling down pods, require explicit allow-listing by the customer and will be logged in the customer’s audit log.

The EKS cluster digital twin is designed to be lightweight:

* A pod requiring multiple vCPUs and high memory in the customer appliance may be represented by a pod requiring minimal resources (e.g., 0.25 vCPUs and 256 MB RAM) in the digital twin.

This keeps infrastructure costs low while preserving real-time observability and control.

### AWS EC2 Instance (Virtual Machine) (High-Fidelity)

You'll find a real EC2 instance in your Tensor9 AWS account.

This **EC2 instance digital twin** mirrors the operational state of the EC2 instance running in the customer appliance. Here's what you can do:

* **Observe Instance State**: Open the EC2 console in your Tensor9 AWS account to view the state of the EC2 instance (e.g., running, stopped, or terminated). This reflects the actual state of the instance in the customer appliance.
* **Monitor Metrics**: View key metrics such as CPU utilization, disk usage, and network activity. These metrics are synchronized in real time with the corresponding EC2 instance in the customer appliance.
* **Observe Logs**: System logs (e.g., instance initialization logs) can be viewed, provided they are explicitly allow-listed by the customer.
  * **Log Allow-List**: The customer must define which logs can be viewed through the digital twin.
  * **Audit Logging**: All logs accessed by you through your digital twin are appended to the customer’s audit log for full traceability.
* **Send SSH Commands**: You can send **non-interactive SSH commands** to the EC2 instance digital twin. However, this feature is subject to the following conditions:
  * **Command Allow-List**: The customer must explicitly allow-list which commands can be sent to their EC2 instances.
  * **Audit Logging**: All issued commands, along with their corresponding output, are logged in the customer’s audit log for traceability.
* **Debug Failures**: If an EC2 instance in the customer appliance experiences an issue (e.g., hardware failure or OS-level crash), the corresponding EC2 instance digital twin will reflect this failure by entering a stopped or terminated state. This allows you to detect and investigate issues without direct access to the customer’s environment.

The EC2 instance digital twin is designed to use minimal resources:

* An EC2 instance requiring `m5.4xlarge` in the customer appliance may be represented by a `t2.micro` in the digital twin, keeping infrastructure costs low.

This lightweight design maintains real-time synchronization and observability while reducing resource usage.

### AWS Lambda Function (Serverless) (High-Fidelity)

The **Lambda function digital twin** is represented by a set of CloudWatch resources in your Tensor9 AWS account rather than a real Lambda function. Here's how it works:

When you deploy a Lambda function to an appliance, Tensor9 creates the following digital twin resources in your Tensor9 AWS account:

1. **CloudWatch Dashboard:** Displays Lambda-related metrics such as invocation count, average duration, error rate, and throttles. These metrics are populated by telemetry sent from the corresponding Lambda function in the customer appliance.
2. **CloudWatch Log Group:** Contains logs for the Lambda function, synchronized in real time from the appliance. Logs are appended to the customer’s audit log for traceability.

#### Key Features:

* **Monitor Metrics:** The CloudWatch dashboard provides a real-time view of key Lambda function metrics. This includes:
  * Invocation count
  * Duration
  * Error rate
  * Throttled invocations
* **View Logs:** The CloudWatch log group captures the logs generated by the Lambda function inside the appliance. You can explore these logs just as you would for a real Lambda function in your AWS account.
* **Audit-Logged Telemetry:** All logs synchronized to your digital twin are appended to the customer’s audit log to maintain transparency.
* **Customer-Controlled Data Flow:** The customer explicitly allow-lists what telemetry (e.g., logs or metrics) can be sent from the appliance to the digital twin.

#### Important Notes:

* **No Test Invocations:** You cannot invoke the Lambda function directly through the digital twin, as this would require duplicating the compute resources and incurring corresponding costs.
* **No Automatic Alerts:** The Lambda function digital twin does not come with predefined CloudWatch alarms. However, you can create your own CloudWatch alarms if needed, just as you would for a real Lambda function.

By providing a CloudWatch dashboard and log group, the Lambda function digital twin enables you to observe the operational state of serverless workloads within customer appliances without requiring direct access to the customer’s environment.

### AWS S3 Bucket (Blob Storage) (High-Fidelity)

You'll find a digital twin of your S3 bucket in your Tensor9 AWS account. This **S3 bucket digital twin** is comprised of three components:

1. **CloudWatch Dashboard**: Displays key metrics for monitoring bucket activity.
2. **Forwarding Bucket**: A real S3 bucket that allows you to forward `PUT` and `DELETE` operations to the corresponding bucket in the appliance.
3. **SNS Topic for Status Notifications**: Receives notifications about the status of forwarding operations.

Here's how each component works:

#### **CloudWatch Dashboard**

The dashboard provides key metrics such as:

* **Read/Write/Delete Operations**: Number of read, write, and delete requests.
* **Request Latency**: Average latency for operations forwarded through the digital twin.
* **Data Transfer Volume**: The amount of data sent through the forwarding bucket.
* **Error Rate**: Number of failed requests (e.g., due to permission issues or conflicts).

This dashboard allows you to monitor how your customer's appliance is interacting with its S3 bucket.

#### **Forwarding Bucket**

The forwarding bucket is a real S3 bucket in your Tensor9 AWS account. It allows you to:

* **PUT objects** (including multi-part uploads) with tags and metadata.
* **DELETE objects** within the allowed prefixes.

All operations in the forwarding bucket are forwarded to the corresponding bucket in the appliance.

**Important notes:**

* You cannot change the configuration of the forwarding bucket—it is managed by Tensor9 and is used solely for object-level operations (`PUT` and `DELETE`).
* ACLs are not supported. Instead, access control relies on the bucket policies defined in the appliance.
* The forwarding bucket syncs periodically to the appliance’s bucket, but there is no SLA for synchronization timing.
* Overwrites are allowed, and you can retry uploads to the forwarding bucket as needed.

**Read/List Operations:**

* You can `LIST` and `GET` objects from the forwarding bucket, but these operations reflect the contents of the forwarding bucket, *not* the bucket inside the appliance.
* This helps verify the contents being forwarded without requiring direct access to the appliance.

#### **SNS Topic for Status Notifications**

Your S3 bucket digital twin also exposes an SNS topic to provide status updates on forwarding operations. Each operation you perform in the forwarding bucket generates a status message that is appended to the SNS topic. Notifications may include:

* **Success**: The object was successfully forwarded to the appliance bucket.
* **Conflict**: There was a conflict (e.g., the object already exists with different data).
* **Failure**: An error occurred (e.g., due to permission issues).

The SNS topic allows you to programmatically track the results of your forwarding requests and handle any issues efficiently.

#### **Security and Audit Logging**

* The customer must explicitly allow-list the prefixes that the forwarding bucket can write to and delete from.
* All forwarded operations (writes and deletes) are appended to the customer’s audit log for transparency and traceability.
* Any logs synchronized to your digital twin will also be appended to the customer’s audit log to maintain a full record of your interactions.

By using the S3 bucket digital twin, you can securely interact with your customer’s S3 bucket without direct access, maintaining data privacy and compliance while still enabling essential workflows.

### AWS DynamoDB Table (NoSQL) (High-Fidelity)

You'll find a digital twin of your DynamoDB table in your Tensor9 AWS account. This **DynamoDB table digital twin** is comprised of of three components:

1. **CloudWatch Dashboard**: Displays key metrics for monitoring table activity.
2. **Forwarding Table**: A real DynamoDB table that allows you to forward `PutItem` and `DeleteItem` operations to the corresponding table in the appliance.
3. **SNS Topic for Status Notifications**: Receives notifications about the status of forwarding operations.

Here's how each component works:

#### **CloudWatch Dashboard**

The dashboard provides key metrics such as:

* **Read/Write/Delete Requests**: Number of requests performed through the forwarding table.
* **Request Latency**: Average latency for operations forwarded through the digital twin.
* **Item Size Metrics**: Average and maximum item sizes for `PutItem` operations.
* **Error Rate**: Number of failed requests (e.g., due to conflicts or permission issues).

This dashboard provides insights into the performance and reliability of your application’s interactions with the forwarding table.

#### **Forwarding Table**

The forwarding table is a real DynamoDB table in your Tensor9 AWS account. It allows you to:

* **`PutItem`and`PutItem` Batch Operations**: Add or update items in the forwarding table.
* **`DeleteItem`and`DeleteItem` Batch Operations**: Delete items from the forwarding table.

**Important notes:**

* **Transactions are not supported**: The forwarding table does not support `TransactWriteItems` or `TransactGetItems` operations.
* You cannot modify the schema, provisioned throughput, or settings of the forwarding table—it is managed by Tensor9.
* The forwarding table syncs periodically to the appliance’s table, but there is no SLA for synchronization timing.
* Overwrites are allowed, and you can retry requests to the forwarding table as needed.

**Read Operations:**

* You can `GetItem` and `Scan` the forwarding table, but these operations reflect the contents of the forwarding table, *not* the table inside the appliance.
* This helps verify the data being forwarded without requiring direct access to the appliance.

#### **SNS Topic for Status Notifications**

Your DynamoDB table digital twin also exposes an SNS topic to provide status updates on forwarding operations. Each operation you perform in the forwarding table generates a status message that is appended to the SNS topic. Notifications may include:

* **Success**: The item was successfully forwarded to the appliance table.
* **Conflict**: There was a version conflict (e.g., the item already exists with different values).
* **Failure**: An error occurred (e.g., due to permission issues or exceeding partition throughput limits).

The SNS topic allows you to programmatically track the results of your forwarding requests and handle any issues efficiently.

#### **Security and Audit Logging**

* The customer must explicitly allow-list the keys or key patterns that the forwarding table can write to and delete from.
* All forwarded operations (`PutItem`, `PutItem` batch, `DeleteItem`, `DeleteItem` batch) are appended to the customer’s audit log for transparency and traceability.
* Any logs synchronized to your digital twin will also be appended to the customer’s audit log to maintain a full record your interactions.

By using the DynamoDB table digital twin, you can securely interact with your customer’s NoSQL data without direct access, maintaining data privacy and compliance while still enabling essential workflows.

### More resources not listed above

Tensor9 supports high-fidelity digital twins for the following resources:

* AWS RDS Database (SQL)
* AWS Aurora Database (Serverless SQL)
* AWS SQS Queue (Queues)
* AWS Kinesis Stream (Streaming)
* AWS Managed Streaming for Apache Kafka (Streaming)
* AWS API Gateway (Serverless)
* AWS Elastic MapReduce (Batch processing)
* AWS Batch (Batch processing)

Tensor9 supports all other AWS resources supported by CloudFormation as low-fidelity digital twins.

## All done!

You’ve explored how Tensor9’s digital twins provide real-time observability and control over the resources running in customer appliances. You can use digital twins to understand the operational state of resources deployed inside customer appliances, enabling better operational awareness, faster debugging, and mean lower time to issue resolution.  In some cases (e.g. S3 buckets and DynamoDB tables), you can also use digital twins to affect the operational state of resources inside customer appliances.

***

### Glossary

* **Digital Twin**: A resource in your Tensor9 AWS account that mirrors the operational state of a corresponding resource in a customer appliance.
* **Forwarding Resource**: A digital twin component (e.g., forwarding bucket or forwarding table) that forwards operations (like writes and deletes) to the corresponding resource in the appliance.
* **Allow-List**: A list of approved logs, commands, or operations that a customer explicitly allows you to access or perform via a digital twin.
* **Audit Log**: A secure, immutable record of all interactions (e.g., logs viewed, actions performed) between you and your customer's appliance.  This includes all data sent from your customer's appliance to your environment.
* **Appliance**: A self-contained, secure environment where your Tensor9 app runs inside your customer’s infrastructure, providing compute, storage, and networking while ensuring data remains in the customer’s control.
* **App**: Your software and infrastructure packaged for delivery to your customers via Tensor9. Like an iOS app runs on a user’s phone, a Tensor9 app runs securely in a customer appliance.
* **Origin Stack**: Your existing CloudFormation stack that defines the software, infrastructure, and services for your product. Tensor9 synchronizes this stack across customer appliances.