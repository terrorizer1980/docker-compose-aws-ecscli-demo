# docker-compose-aws-ecscli-demo

This repository holds examples of how to bring up Senzing on Amazon Elastic Container Service (ECS)

1. **Beginner** - A very simple deployment.
    1. Features:
        1. A single EC2 implementation
        1. Backing services (PostgreSQL and RabbitMQ/Kafka) are hosted on the EC2 in separate containers
            1. AWS SQS is a separate AWS service
        1. Storage on the EC2.  The AWS Elastic File System (EFS) is not used.
        1. Minimal security.
        1. Not designed for scaling up AWS ECS Services.
    1. Variations:
        1. [beginner-rabbitmq](docs/beginner-rabbitmq) - uses RabbitMQ for queue
        1. [beginner-aws-sqs](docs/beginner-aws-sqs) - uses AWS SQS for queue
        1. [beginner-kafka](docs/beginner-kafka) - uses Kafka for queue
1. [Intermediate](docs/intermediate) - In addition to
   [beginner](docs/beginner), it features:
    1. Use of AWS Fargate
        1. Uses `awsvpc`
    1. AWS Elastic File System (EFS)
    1. Connects to AWS Aurora PostgreSQL
    1. RabbitMQ persistence
    1. Shows scale-up
