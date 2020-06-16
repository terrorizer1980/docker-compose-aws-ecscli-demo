# docker-compose-aws-ecscli-demo

This repository holds examples of how to bring up Senzing on Amazon Elastic Container Service (ECS)

1. [Beginner](docs/beginner) - A very simple deployment.  It features:
    1. A single EC2 implementation
    1. Backing services (PostgreSQL and RabbitMQ) are hosted on the EC2 in separate containers
    1. Storage on the EC2.  The AWS Elastic File System (EFS) is not used.
    1. Minimal security.
    1. Not designed for scaling up AWS ECS Services.
1. [Intermediate](docs/intermediate) - In addition to
   [beginner](docs/beginner), it features:
    1. Use of AWS Fargate
    1. AWS Elastic File System (EFS)
    1. Connects to AWS Aurora PostgreSQL
    1. Shows scale-up
