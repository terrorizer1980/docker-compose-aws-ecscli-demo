# docker-compose-aws-ecscli-demo

This repository holds examples of how to bring up Senzing on Amazon Elastic Container Service (ECS)

1. [Beginner](docs/beginner) - A very simple deployment.  It features:
    1. A single EC2 implementation
    1. Backing services (PostgreSQL and RabbitMQ) are hosted on the EC2 in separate containers
    1. Storage on the EC2.  The AWS Elastic File System (EFS) is not used.
    1. Minimal security.
    1. Not designed for scaling up AWS ECS Services.
