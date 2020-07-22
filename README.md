# docker-compose-aws-ecscli-demo

This repository holds examples of how to bring up Senzing on Amazon Elastic Container Service (ECS)

1. **Beginner** - A very simple deployment.
    1. Features:
        1. A single EC2 implementation.
        1. Backing services (PostgreSQL and RabbitMQ/Kafka) are hosted on the EC2 in separate containers.
            1. AWS SQS is a separate AWS service.
        1. Storage on the EC2.  The AWS Elastic File System (EFS) is not used.
        1. Minimal security.
        1. Not designed for scaling up AWS ECS Services.
    1. Variations:
        1. [beginner-rabbitmq](docs/beginner-rabbitmq) - uses RabbitMQ for queue
        1. [beginner-aws-sqs](docs/beginner-aws-sqs) - uses AWS SQS for queue
        1. [beginner-kafka](docs/beginner-kafka) - uses Kafka for queue
1. [Intermediate](docs/intermediate) - In addition to
   **beginner**, it features:
    1. AWS [Fargate](https://aws.amazon.com/fargate/) instead of EC2.
        1. Uses `awsvpc`.
    1. AWS [Simple Queue Service (SQS)](https://aws.amazon.com/sqs/) instead of RabbitMQ/Kafka.
    1. AWS [Aurora](https://aws.amazon.com/rds/aurora/) PostgreSQL instead of an internal PostgreSQL.
    1. AWS [Elastic File System (EFS)](https://aws.amazon.com/efs/).
