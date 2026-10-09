# Compute and Application Integration

## Select the operating model first

The useful distinction is how much of the runtime and infrastructure the team wants to operate:

- **Amazon EC2** gives control over the instance operating system and runtime, with responsibility for patching, scaling, and capacity design.
- **AWS Elastic Beanstalk** manages application environments while exposing more of the underlying AWS resources for configuration.
- **AWS App Runner** is a managed path for deploying supported web applications and container images without building a full container platform.
- **Amazon ECS** schedules and maintains containers. A task definition describes a workload; an ECS service maintains a desired number of tasks. With the EC2 capacity model, the customer manages container instances. With Fargate, AWS manages the host capacity and the customer selects task resources.

Fargate tasks using `awsvpc` receive task-level network interfaces and security groups. A task in a private subnet needs a route to its dependencies. Private ECR image pulls can use ECR interface endpoints; image layers, logs, secrets, and other integrations may require their own endpoints or another egress path. Configure only the endpoints needed by the task and verify the current service-specific requirements.

ECS Anywhere registers customer-managed external instances with ECS. It provides a common control plane for task scheduling, but the customer still supplies, operates, and secures the external compute and its connectivity.

**Fresh example:** A small team wants to deploy an HTTPS API from a container but does not need custom host management. App Runner may minimize operational work. A platform team that needs placement control, several cooperating containers, or a customized network model may prefer ECS.

### Elastic Beanstalk rollout controls

For supported Linux platforms, Beanstalk platform hooks can run deployment scripts at defined lifecycle points: `prebuild` before platform setup, `predeploy` after setup but before the new version is placed in its runtime location, and `postdeploy` after deployment. Use hooks for deployment-time customization, not as a substitute for an infrastructure-as-code plan.

A blue/green rollout can use two Beanstalk environments: validate the candidate environment, then swap the Beanstalk environment CNAMEs so the stable application name maps to the new environment. The Route 53 record for a custom domain can continue to point at the stable Beanstalk name. Keep the old environment available for rollback and account for client DNS caching; a CNAME swap is not a promise that every client changes paths at the same instant.

## Capacity, interruptions, and job processing

EC2 purchasing options and placement groups solve different constraints:

- **On-Demand** capacity avoids a long-term commitment and is suited to variable or interrupt-sensitive workloads.
- **Spot Instances** trade a lower price for the possibility of interruption. Use them for work that can checkpoint, retry, or move to other capacity.
- **Savings Plans and Reserved Instances** are commitment-based cost tools; they are not substitutes for resilient architecture. Compare their coverage, flexibility, term, and utilization against the workload.
- **Cluster placement** favors low-latency, high-throughput communication among a group of instances; **spread placement** separates instances across distinct hardware to reduce correlated failure risk; **partition placement** separates groups of instances across partitions for large distributed systems.

When Spot capacity hosts ECS tasks, design for interruption: handle termination notifications, stop new placements on draining instances, allow tasks to finish or restart elsewhere, and keep enough alternative capacity for service availability.

**AWS Batch** is for queued batch jobs with job definitions, queues, and compute environments. It schedules work onto suitable capacity, including supported EC2 or Fargate options. Use Lambda for short event-driven functions, Batch for managed job queues and compute placement, and Step Functions when the main problem is coordinating a multi-step workflow. These services can be combined.

## Serverless limits and orchestration

Lambda concurrency is the number of simultaneous executions. **Reserved concurrency** sets an allocation and a ceiling for a function; **provisioned concurrency** prepares execution environments to reduce initialization latency. Neither removes downstream capacity limits. Monitor throttles, duration, retries, and the capacity of databases or APIs called by the function.

Step Functions coordinates stateful workflows and service integrations:

- **Standard workflows** support long-running, durable executions and an exactly-once workflow model unless retry behavior is configured.
- **Express workflows** support high-volume, short-running work. Asynchronous Express has at-least-once execution; synchronous Express has at-most-once execution. Design side effects to be idempotent where duplicate processing is possible.

Choose a workflow type before creating the state machine; the type cannot be changed in place. Check service integration patterns, execution history, duration limits, and pricing for the selected workflow.

Amazon SQS decouples senders from workers. A source queue's redrive policy uses `maxReceiveCount` to decide how many receives without successful deletion are allowed before a message moves to its **dead-letter queue (DLQ)**. Set the threshold high enough to tolerate transient failures; a low value can divert recoverable messages prematurely. A DLQ isolates failures for investigation, while a separate redrive operation moves messages back for another processing attempt. Plan alarms, retention, and idempotent consumers because retries can repeat work.

## Event, data, and device integration

- **Kinesis Data Streams** gives consumers access to a retained stream for custom or multiple real-time processing applications. **Amazon Data Firehose** manages delivery and buffering to configured destinations, with supported transformations. Choose based on whether consumers need to control stream processing or primarily need managed delivery.
- **AWS AppSync** exposes GraphQL APIs that can combine data sources, authorize requests, and publish subscriptions. Offline behavior requires a client-side strategy and compatible data synchronization configuration; a GraphQL endpoint alone does not make every client automatically offline-capable.
- **AWS IoT Core Basic Ingest** routes device messages to the rules engine without the usual message-broker publish/subscribe path for that ingestion flow. It is useful when device telemetry is sent onward for processing rather than consumed through broker subscriptions.
- **AWS IoT Greengrass** runs supported components and local processing on edge devices, including during intermittent connectivity. It is software deployed to customer-managed edge capacity, not a regional extension of AWS hardware like Outposts.

## Legacy configuration-management note

AWS OpsWorks appears in older study material as a way to model configuration and lifecycle automation. The historical lesson is to distinguish configuration management from application deployment and infrastructure provisioning. Its older documentation endpoint was unavailable during preparation, so do not treat OpsWorks as a current recommendation without first checking AWS service lifecycle and support status. For new designs, select currently supported infrastructure-as-code, configuration, and deployment tools based on the specific job.

## References

- [AWS App Runner](https://docs.aws.amazon.com/apprunner/latest/dg/what-is-apprunner.html)
- [AWS Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/Welcome.html)
- [Extending Elastic Beanstalk Linux platforms](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/platforms-linux-extend.html)
- [Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
- [ECS networking and VPC endpoints](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/vpc-endpoints.html)
- [AWS Batch](https://docs.aws.amazon.com/batch/latest/userguide/what-is-batch.html)
- [Lambda concurrency](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)
- [Step Functions workflow types](https://docs.aws.amazon.com/step-functions/latest/dg/choosing-workflow-type.html)
- [Amazon SQS dead-letter queues](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html)
- [Amazon SQS DLQ redrive](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-configure-dead-letter-queue-redrive.html)
- [Kinesis Data Streams and Firehose](https://docs.aws.amazon.com/streams/latest/dev/introduction.html)
- [AWS AppSync data synchronization](https://docs.aws.amazon.com/appsync/latest/devguide/conflict-detection-and-sync.html)
- [AWS IoT Core Basic Ingest](https://docs.aws.amazon.com/iot/latest/developerguide/iot-basic-ingest.html)
- [AWS IoT Greengrass](https://docs.aws.amazon.com/greengrass/v2/developerguide/what-is-iot-greengrass.html)
