# docker-compose-aws-ecscli-demo-intermediate

## Contents

1. [Prerequisites](#prerequisites)
    1. [Install ECS CLI](#install-ecs-cli)
    1. [Multi-factor authentication](#multi-factor-authentication)
    1. [Clone repository](#clone-repository)
1. [Tutorial](#tutorial)
    1. [Identify metadata](#identify-metadata)
    1. [Create backing services](#create-backing-services)
        1. [Provision Elastic File system](#provision-elastic-file-system)
    1. [Configure ECS CLI](#configure-ecs-cli)
    1. [Create cluster](#create-cluster)
    1. [Find security group ID](#find-security-group-id)
    1. [Open inbound ports](#open-inbound-ports)
    1. [Create tasks and services](#create-tasks-and-services)
        1. [Run install Senzing task](#run-install-senzing-task)
        1. [Create Postgres service](#create-postgres-service)
        1. [Run Senzing database schema task](#run-senzing-database-schema-task)
        1. [Create phpPgAdmin service](#create-phppgadmin-service)
        1. [Run init-container task](#run-init-container-task)
        1. [Create RabbitMQ service](#create-rabbitmq-service)
        1. [Run Mock data generator task](#run-mock-data-generator-task)
        1. [Create Stream loader service](#create-stream-loader-service)
        1. [Create Senzing API server service](#create-senzing-api-server-service)
        1. [Create Senzing Web App service](#create-senzing-web-app-service)
        1. [Create Jupyter notebook service](#create-jupyter-notebook-service)
        1. [Create Senzing X-Term service](#create-senzing-x-term-service)
1. [Cleanup](#cleanup)
    1. [Bring down cluster](#bring-down-cluster)
    1. [Delete tasks definitions](#delete-tasks-definitions)
    1. [Clean logs](#clean-logs)
    1. [Review cleanup in AWS console](#review-cleanup-in-aws-console)
1. [References](#references)

## Prerequisites

### Install ECS CLI

To install `ecs-cli`, follow steps at
[Configuring the Amazon ECS CLI](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html).

### Multi-factor authentication

:thinking: **Optional:**
If multi-factor authentication is used to access AWS,
see [How to set AWS multi-factor authentication credentials](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/set-aws-mfa-credentials.md).

### Clone repository

For more information on environment variables,
see [Environment Variables](https://github.com/Senzing/knowledge-base/blob/master/lists/environment-variables.md).

1. Set these environment variable values:

    ```console
    export GIT_ACCOUNT=senzing
    export GIT_REPOSITORY=docker-compose-aws-ecscli-demo
    export GIT_ACCOUNT_DIR=~/${GIT_ACCOUNT}.git
    export GIT_REPOSITORY_DIR="${GIT_ACCOUNT_DIR}/${GIT_REPOSITORY}"
    ```

1. Using the environment variables values just set, follow steps in [clone-repository](https://github.com/Senzing/knowledge-base/blob/master/HOWTO/clone-repository.md) to install the Git repository.

## Tutorial

### Identify metadata

#### AWS metadata

1. :pencil2: Set AWS metadata:
   [region](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html) and
   [key pair](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html).
   Example:

    ```console
    export AWS_REGION=us-east-1
    export AWS_KEYPAIR=aws-default-key-pair
    ```

#### Identify project

1. :pencil2: Choose a prefix used in AWS object names.
   Example:

    ```console
    export SENZING_AWS_PROJECT=project01
    ```

#### EULA

To use the Senzing code, you must agree to the End User License Agreement (EULA).

1. :warning: This step is intentionally tricky and not simply copy/paste.
   This ensures that you make a conscious effort to accept the EULA.
   Example:

    <pre>export SENZING_ACCEPT_EULA="&lt;the value from <a href="https://github.com/Senzing/knowledge-base/blob/master/lists/environment-variables.md#senzing_accept_eula">this link</a>&gt;"</pre>

#### Synthesize variables

1. Create additional environment variables.
   Example:

    ```console
    export SENZING_AWS_ECS_CLUSTER=${SENZING_AWS_PROJECT}-cluster
    export SENZING_AWS_ECS_CLUSTER_CONFIG=${SENZING_AWS_PROJECT}-config-name
    export SENZING_AWS_ECS_PARAMS_FILE=${GIT_REPOSITORY_DIR}/resources/intermediate/ecs-params.yaml
    ```

### Configure ECS CLI

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [configure](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-configure.html).
   Example:

    ```console
    ecs-cli configure \
       --cluster ${SENZING_AWS_ECS_CLUSTER} \
       --config-name ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
       --default-launch-type FARGATE \
       --region ${AWS_REGION}
    ```

1. :thinking: **Optional:**
   To view configuration values, see `~/.ecs/config`.

    ```console
    cat ~/.ecs/config
    ```

### Create cluster

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-up.html).
   Example:

    ```console
    ecs-cli up \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --force
    ```

1. :thinking: **Optional:**
   View aspects of AWS ECS cluster in AWS console.
    1. [cloudformation](https://console.aws.amazon.com/cloudformation/home?#/stacks)
    1. [ecs](https://console.aws.amazon.com/ecs/home)
    1. [vpc](https://console.aws.amazon.com/vpc/home)
        1. [internet gateways](https://console.aws.amazon.com/vpc/home?#igws)
        1. [network acls](https://console.aws.amazon.com/vpc/home?#acls)
        1. [route tables](https://console.aws.amazon.com/vpc/home?#RouteTables)
        1. [security groups](https://console.aws.amazon.com/vpc/home?#SecurityGroups)
        1. [subnets](https://console.aws.amazon.com/vpc/home?#subnets)
        1. [vpc](https://console.aws.amazon.com/vpc/home?#vpcs)

### Save Cluster metadata

1. The `ecs-cli up` command that just completed prints metadata
   that needs to be captured in environment variables for later use.
   Example:

    ```console
    :
    INFO[0001] Waiting for your cluster resources to be created...
    INFO[0002] Cloudformation stack status         stackStatus=CREATE_IN_PROGRESS
    INFO[0063] Cloudformation stack status         stackStatus=CREATE_IN_PROGRESS
    VPC created: vpc-00000000000000000
    Subnet created: subnet-11111111111111111
    Subnet created: subnet-22222222222222222
    Cluster creation succeeded.
    ```

1. :pencil2: Set environment variable with VPC ID.
   Example:

    ```console
    export SENZING_AWS_VPC_ID=vpc-00000000000000000
    ```

1. :pencil2: Set environment variable with Subnet #1
   Example:

    ```console
    export SENZING_AWS_SUBNET_ID_1=subnet-11111111111111111
    ```

1. :pencil2: Set environment variable with Subnet #2
   Example:

    ```console
    export SENZING_AWS_SUBNET_ID_2=subnet-22222222222222222
    ```

1. :thinking: **Optional:**
   Review environment variables.
   Example:

    ```console
    env | grep SENZING | sort
    ```

### Find security group ID

1. Find the AWS security group.
   Run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [ec2](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/index.html)
   [describe-security-groups](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/describe-security-groups.html)
   Save security group ID in `SENZING_AWS_EC2_SECURITY_GROUP` environment variable.
   Example:

    ```console
    export SENZING_AWS_EC2_SECURITY_GROUP=$( \
      aws ec2 describe-security-groups \
        --filters Name=vpc-id,Values=${SENZING_AWS_VPC_ID} \
        --region ${AWS_REGION} \
      | jq --raw-output ".SecurityGroups[0].GroupId"
    )
    ```

1. :thinking: **Optional:**
   View security group ID.
   Example:

    ```console
    echo ${SENZING_AWS_EC2_SECURITY_GROUP}
    ```

### Open inbound ports

:warning: **Warning:** The following inbound port specifications are **wide open**.
Meaning they can be accessed from anywhere.
For demonstration purposes, this is fine.
For production purposes it is not fine.

1. Open inbound ports.
   Run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [ec2](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/index.html)
   [authorize-security-group-ingress](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/authorize-security-group-ingress.html).
   Example:

    ```console
    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=22,ToPort=22,IpRanges='[{CidrIp=0.0.0.0/0,Description="SSH"}]'

    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=2049,ToPort=2049,IpRanges='[{CidrIp=0.0.0.0/0,Description="NFS"}]'

    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=8250,ToPort=8250,IpRanges='[{CidrIp=0.0.0.0/0,Description="Senzing API server"}]'

    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=8251,ToPort=8251,IpRanges='[{CidrIp=0.0.0.0/0,Description="Senzing Web App"}]'

    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=8254,ToPort=8254,IpRanges='[{CidrIp=0.0.0.0/0,Description="Senzing X-Term"}]'

    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=9171,ToPort=9171,IpRanges='[{CidrIp=0.0.0.0/0,Description="phpPgAdmin"}]'

    aws ec2 authorize-security-group-ingress \
      --group-id ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --ip-permissions \
        IpProtocol=tcp,FromPort=9178,ToPort=9178,IpRanges='[{CidrIp=0.0.0.0/0,Description="Senzing Jupyter notebooks"}]'

    ```


1. :thinking: **Optional:**
   To view Security Group, run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [ec2](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/index.html)
   [describe-security-groups](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/describe-security-groups.html).
   Example:

    ```console
    aws ec2 describe-security-groups \
      --group-ids ${SENZING_AWS_EC2_SECURITY_GROUP}
    ```

1. :thinking: **Optional:**
   View Security Group in AWS console.
    1. View [VPC > Security Groups](https://console.aws.amazon.com/vpc/home?#SecurityGroups:)
    1. In "Security group ID" column, click ID having the value stored in the `SENZING_AWS_EC2_SECURITY_GROUP` environment variable.

### AWS bug work-around

1. An AWS `aws`/`ecs-cli` [bug](https://github.com/aws/amazon-ecs-cli/issues/1083) prevents the use of CLI-only instructions.
To work around the bug:
    1. Visit [AWS Console for Security Groups](https://console.aws.amazon.com/ec2/v2/home?#SecurityGroups:)
    1. Choose Security Group ID (e.g. "`e${SENZING_AWS_PROJECT}-cluster")
        1. Can be found by running:

            ```console
            echo ${SENZING_AWS_EC2_SECURITY_GROUP}
            ```
    1. In "inbound rules", click "edit inbound rules" button
    1. For "NFS" rule, click "Delete" button to remove rule.
    1. At bottom, click "Save rules" button.

### Create backing services

FIXME: Provision in same VPC and Subnets.

#### Provision Elastic File system

1. Create EFS file system.
   Run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [efs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/efs/index.html)
   [create-file-system](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/efs/create-file-system.html).
   Save file system ID in `SENZING_AWS_EFS_FILESYSTEM_ID` environment variable.
   Example:

    ```console
    export SENZING_AWS_EFS_FILESYSTEM_ID=$( \
      aws efs create-file-system \
        --creation-token ${SENZING_AWS_PROJECT}-efs \
        --tags Key=Name,Value=${SENZING_AWS_PROJECT}-ecs-cluster-efs \
      | jq --raw-output ".FileSystemId"
    )
    ```

1. Create mount in first subnet.
   Example:

    ```console
    aws efs create-mount-target \
      --file-system-id ${SENZING_AWS_EFS_FILESYSTEM_ID} \
      --security-groups ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --subnet-id ${SENZING_AWS_SUBNET_ID_1}
    ```

1. Create mount in second subnet.
   Example:

    ```console
    aws efs create-mount-target \
      --file-system-id ${SENZING_AWS_EFS_FILESYSTEM_ID} \
      --security-groups ${SENZING_AWS_EC2_SECURITY_GROUP} \
      --subnet-id ${SENZING_AWS_SUBNET_ID_2}
    ```

1. :thinking: **Optional:**
   View file system ID.
   Example:

    ```console
    echo ${SENZING_AWS_EFS_FILESYSTEM_ID}
    ```

1. :thinking: **Optional:**
   View [Elastic File Systems](https://console.aws.amazon.com/efs/home?#/filesystems)
   in AWS console.

#### Provision Aurora PostgreSQL

FIXME:

1. Create Aurora PostgreSQL database.
   Run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [rds](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/rds/index.html)
   [create-db-instance](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/rds/create-db-instance.html).
   Save file system ID in `SENZING_AWS_SQS_ID` environment variable.
   Example:

    ```console
      aws rds create-db-instance \
        --allocated-storage 20 \
        --db-instance-class db.t3.medium \
        --db-instance-identifier ${SENZING_AWS_PROJECT}-aurora-postgresql \
        --db-name G2 \
        --engine aurora-postgresql \
        --master-user-password g2password \
        --master-username g2username \
        --publicly-accessible \
        --tags Key=Name,Value=${SENZING_AWS_PROJECT}-aurora-postgresql \
      > ~/aws-rds-create-db-instance.json
    ```

#### Provision Simple Queue Service

1. Create SQS queue.
   Run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [sqs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sqs/index.html)
   [create-queue](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sqs/create-queue.html).
   Save file system ID in `SENZING_AWS_SQS_ID` environment variable.
   Example:

    ```console
    export SENZING_AWS_SQS_QUEUE_URL=$( \
      aws sqs create-queue \
        --queue-name ${SENZING_AWS_PROJECT}-sqs-queue \
        --tags Key=Name,Value=${SENZING_AWS_PROJECT}-sqs-queue \
      | jq --raw-output ".QueueUrl"
    )
    ```

1. :thinking: **Optional:**
   View Simple Queue Service (SQS) URL.
   Example:

    ```console
    echo ${SENZING_AWS_SQS_QUEUE_URL}
    ```

1. :thinking: **Optional:**
   View [Simple Queue Service](https://console.aws.amazon.com/sqs/v2/home?#/queues)
   in AWS console.

### Create tasks and services

#### Run install Senzing task

Install Senzing onto the Elastic File System.

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-up.html)
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/intermediate/docker-compose-yum.yaml \
      --project-name ${SENZING_AWS_PROJECT}-project-name-yum \
      up \
        --create-log-groups
    ```

1. This task is a short-lived "job", not a long-running service.
   When the task state is `STOPPED`, the job has finished.

1. :thinking: **Optional:**
   View progress.
    1. [ecs](https://console.aws.amazon.com/ecs/home)
        1. Select ${SENZING_AWS_ECS_CLUSTER}
        1. Click "Update Cluster" to update information.
        1. Click "Tasks" tab.
        1. If task is seen, it is still "RUNNING".  Wait until task is complete.

#### Create Postgres service

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision Postgres database service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/intermediate/docker-compose-postgres.yaml \
      --project-name ${SENZING_AWS_PROJECT}-project-name-postgres \
      service up
    ```

1. :thinking: **Optional:**
   To view service definition, run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [ecs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/index.html)
   [describe-services](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/describe-services.html).
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${SENZING_AWS_ECS_CLUSTER} \
      --services ${SENZING_AWS_PROJECT}-project-name-postgres
    ```

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address definition.
   Save host IP in `SENZING_POSTGRES_HOST` environment variable.
   Example:

    ```console
    export SENZING_POSTGRES_HOST=$( \
      ecs-cli ps \
        --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      | grep  postgres \
      | awk '{print $3}' \
      | awk -F \: {'print $1'} \
    )
    ```

1. :thinking: **Optional:**
   View `SENZING_POSTGRES_HOST` value.
   Example:

    ```console
    echo $SENZING_POSTGRES_HOST
    ```

#### XXX AWS example

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-up.html)
   to create Senzing database schema.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/intermediate/docker-compose-aws-example.yaml \
      --project-name ${SENZING_AWS_PROJECT}-project-name-aws-example \
      up
    ```

#### Run Senzing database schema task

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-up.html)
   to create Senzing database schema.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/intermediate/docker-compose-postgres-init.yaml \
      --project-name ${SENZING_AWS_PROJECT}-project-name-postgres-init \
      up
    ```

1. This task is a short-lived "job", not a long-running service.
   When the task state is `STOPPED`, the job has finished.

1. :thinking: **Optional:**
   View progress.
    1. [ecs](https://console.aws.amazon.com/ecs/home)
        1. Select ${SENZING_AWS_ECS_CLUSTER}
        1. Click "Update Cluster" to update information.
        1. Click "Tasks" tab.
        1. If task is seen, it is still "RUNNING".  Wait until task is complete.

#### Create phpPgAdmin service

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision phpPgAdmin database client service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/intermediate/docker-compose-phppgadmin.yaml \
      --project-name ${SENZING_AWS_PROJECT}-project-name-phppgadmin \
      service up
    ```

1. :thinking: **Optional:**
   To view phpPgAdmin,
   run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address and port.
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
    | grep phppgadmin
    ```

   **Username:** postgres
   **Password:** postgres

#### Run init-container task

Configure Senzing in `/etc/opt/senzing` and `/var/opt/senzing` files.

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-up.html)
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/intermediate/docker-compose-init-container.yaml \
      --project-name ${SENZING_AWS_PROJECT}-project-name-init-container \
      up
    ```

1. This task is a short-lived "job", not a long-running service.
   When the task state is `STOPPED`, the job has finished.

1. :thinking: **Optional:**
   View progress.
    1. [ecs](https://console.aws.amazon.com/ecs/home)
        1. Select ${SENZING_AWS_ECS_CLUSTER}
        1. Click "Update Cluster" to update information.
        1. Click "Tasks" tab.
        1. If task is seen, it is still "RUNNING".  Wait until task is complete.

#### Create RabbitMQ service

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision RabbitMQ service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/intermediate/docker-compose-rabbitmq.yaml \
      --project-name ${SENZING_AWS_PROJECT}-project-name-rabbitmq \
      service up
    ```

1. :thinking: **Optional:**
   To view service definition, run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [ecs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/index.html)
   [describe-services](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/describe-services.html).
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${SENZING_AWS_ECS_CLUSTER} \
      --services ${SENZING_AWS_PROJECT}-project-name-rabbitmq
    ```

1. :thinking: **Optional:**
   To view RabbitMQ,
   run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address and port.
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
    | grep rabbitmq
    ```

   Use the value having port 15672 which is the RabbitMQ web application.

   **Username:** user
   **Password:** bitnami

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address definition.
   Save host IP in `SENZING_RABBITMQ_HOST` environment variable.
   Example:

    ```console
    export SENZING_RABBITMQ_HOST=$( \
      ecs-cli ps \
        --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      | grep  rabbitmq \
      | awk '{print $3}' \
      | awk -F \: {'print $1'} \
    )
    ```

1. :thinking: **Optional:**
   View `SENZING_RABBITMQ_HOST` value.
   Example:

    ```console
    echo $SENZING_RABBITMQ_HOST
    ```

#### Run Mock data generator task

Read JSON lines from a URL-addressable file and send to RabbitMQ.

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-up.html)
   to send messages to RabbitMQ.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/intermediate/docker-compose-mock-data-generator.yaml \
      --project-name ${SENZING_AWS_PROJECT}-project-name-mock-data-generator \
      up
    ```

1. This task is a short-lived "job", not a long-running service.
   When the task state is `STOPPED`, the job has finished.
   However, this is a long-running job.
   There is no need to wait for its completion.

#### Create Stream loader service

The stream loader service reads messages from RabbitMQ and inserts them into the Senzing Model.

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision stream-loader service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/intermediate/docker-compose-stream-loader.yaml \
      --project-name ${SENZING_AWS_PROJECT}-project-name-stream-loader \
      service up
    ```

1. :thinking: **Optional:**
   To view service definition, run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [ecs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/index.html)
   [describe-services](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/describe-services.html).
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${SENZING_AWS_ECS_CLUSTER} \
      --services ${SENZING_AWS_PROJECT}-project-name-stream-loader
    ```

#### Create Senzing API server service

The Senzing API server communicates with the Senzing Engine to provide an HTTP
[Senzing REST API](https://github.com/Senzing/senzing-rest-api).

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision Senzing API server service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/intermediate/docker-compose-apiserver.yaml \
      --project-name ${SENZING_AWS_PROJECT}-project-name-apiserver \
      service up
    ```

1. :thinking: **Optional:**
   To view service definition, run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [ecs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/index.html)
   [describe-services](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/describe-services.html).
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${SENZING_AWS_ECS_CLUSTER} \
      --services ${SENZING_AWS_PROJECT}-project-name-apiserver
    ```

1. :thinking: **Optional:**
   To view API server, run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address and port.
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
    | grep apiserver
    ```

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address definition.
   This information will be used in subsequent steps.
   Save host IP in `SENZING_RABBITMQ_HOST` environment variable.
   Example:

    ```console
    export SENZING_IP_ADDRESS_APISERVER=$( \
      ecs-cli ps \
        --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      | grep  apiserver \
      | awk '{print $3}' \
      | awk -F \: {'print $1'} \
    )
    ```

1. :thinking: **Optional:**
   Verify Senzing API server is running.
   A JSON response should be given to the following `curl` request.
   Example:

    ```console
    curl -X GET "http://${SENZING_IP_ADDRESS_APISERVER}:8250/heartbeat"
    ```

1. :thinking: **Optional:**
   Play with
   [Senzing API in Swagger editor](http://editor.swagger.io/?url=https://raw.githubusercontent.com/Senzing/senzing-rest-api/master/senzing-rest-api.yaml).
   In **Server variables** > **host** text field, enter value of `SENZING_IP_ADDRESS_APISERVER`.
   To find the value, run

    ```console
    echo $SENZING_IP_ADDRESS_APISERVER
    ```

#### Create Senzing Web App service

The Senzing Web App provides a user interface to Senzing functionality.

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision Senzing Web App service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/intermediate/docker-compose-webapp.yaml \
      --project-name ${SENZING_AWS_PROJECT}-project-name-webapp \
      service up
    ```

1. :thinking: **Optional:**
   To view service definition, run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [ecs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/index.html)
   [describe-services](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/describe-services.html).
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${SENZING_AWS_ECS_CLUSTER} \
      --services ${SENZING_AWS_PROJECT}-project-name-webapp
    ```

1. :thinking: **Optional:**
   To view Senzing web app, run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address and port.
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
    | grep webapp
    ```

#### Create Jupyter notebook service

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision Senzing Web App service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/intermediate/docker-compose-jupyter.yaml \
      --project-name ${SENZING_AWS_PROJECT}-project-name-jupyter \
      service up
    ```

1. :thinking: **Optional:**
   To view service definition, run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [ecs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/index.html)
   [describe-services](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/describe-services.html).
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${SENZING_AWS_ECS_CLUSTER} \
      --services ${SENZING_AWS_PROJECT}-project-name-jupyter
    ```

1. :thinking: **Optional:**
   To view Jupyter, run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address and port.
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
    | grep jupyter
    ```

#### Create Senzing X-Term service

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision Senzing X-Term service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
      --ecs-params ${SENZING_AWS_ECS_PARAMS_FILE} \
      --file ${GIT_REPOSITORY_DIR}/resources/intermediate/docker-compose-xterm.yaml \
      --project-name ${SENZING_AWS_PROJECT}-project-name-xterm \
      service up
    ```

1. :thinking: **Optional:**
   To view service definition, run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [ecs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/index.html)
   [describe-services](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/describe-services.html).
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${SENZING_AWS_ECS_CLUSTER} \
      --services ${SENZING_AWS_PROJECT}-project-name-xterm
    ```

1. :thinking: **Optional:**
   To view Senzing X-Term, run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address and port.
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG} \
    | grep xterm
    ```

## Cleanup

### Bring down cluster

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [down](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-down.html).
   Example:

    ```console
    ecs-cli down \
      --force \
      --cluster-config ${SENZING_AWS_ECS_CLUSTER_CONFIG}
    ```

### Delete tasks definitions

1. Identify suffixes for `${SENZING_AWS_PROJECT}-project-name-`.
   Example:

    ```console
    export SENZING_ECS_TASK_DEFINITIONS=( \
      "apiserver" \
      "init" \
      "init-container" \
      "jupyter" \
      "mock-data-generator" \
      "phppgadmin" \
      "postgres" \
      "postgres-init" \
      "rabbitmq" \
      "stream-loader" \
      "webapp" \
      "xterm" \
    )
    ```

1. Delete task definitions. Run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [ecs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/index.html)
   [deregister-task-definition](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/deregister-task-definition.html)
   in a loop over `SENZING_ECS_TASK_DEFINITIONS` values.
   Example:

    ```console
    for SENZING_ECS_TASK_DEFINITION in ${SENZING_ECS_TASK_DEFINITIONS[@]};\
    do \
      aws ecs deregister-task-definition \
        --task-definition $( \
          aws ecs list-task-definitions \
            --family-prefix "${SENZING_AWS_PROJECT}-project-name-${SENZING_ECS_TASK_DEFINITION}" \
          | jq --raw-output .taskDefinitionArns[0] \
        ) > /dev/null; \
    done
    ```

### Delete simple queue service

1. Delete EFS file system.
   Run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [sqs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sqs/index.html)
   [delete-queue](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/sqs/delete-queue.html).
   Example:

    ```console
    aws sqs delete-queue \
      --queue-url ${SENZING_AWS_SQS_QUEUE_URL}
    ```

### Delete elastic file system

1. Delete EFS file system.
   Run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [efs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/efs/index.html)
   [delete-file-system](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/efs/delete-file-system.html).
   Example:

    ```console
    aws efs delete-file-system \
      --file-system-id ${SENZING_AWS_EFS_FILESYSTEM_ID}
    ```

### Clean logs

1. Delete logs. Run
   [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
   [logs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/logs/index.html)
   [delete-log-group](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/logs/delete-log-group.html)
   Example:

    ```console
    aws logs delete-log-group \
      --log-group-name senzing-docker-compose-aws-ecscli-demo
    ```

### Review cleanup in AWS console

1. In AWS Console:
    1. [ec2](https://console.aws.amazon.com/ec2/v2/home)
        1. [network interfaces](https://console.aws.amazon.com/ec2/v2/home?#NIC)
        1. [security groups](https://console.aws.amazon.com/ec2/v2/home?#SecurityGroups)
    1. [cloudformation](https://console.aws.amazon.com/cloudformation/home?#/stacks)
    1. [cloudwatch](https://console.aws.amazon.com/cloudwatch/home)
        1. [log groups](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups)
    1. [ecs](https://console.aws.amazon.com/ecs/home)
        1. [clusters](https://console.aws.amazon.com/ecs/home?#/clusters)
        1. [task definitions](https://console.aws.amazon.com/ecs/home?#/taskDefinitions)
    1. [efs](https://console.aws.amazon.com/efs/home?#/filesystems)
    1. [rds](https://console.aws.amazon.com/rds/home?#databases:)
    1. [sqs](https://console.aws.amazon.com/sqs/v2/home)

## References

### AWS Console

1. [cloudformation](https://console.aws.amazon.com/cloudformation/home?#/stacks)
1. [cloudwatch](https://console.aws.amazon.com/cloudwatch/home)
    1. [log groups](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups)
1. [ec2](https://console.aws.amazon.com/ec2/v2/home)
    1. [auto scaling groups](https://console.aws.amazon.com/ec2autoscaling/home?#/details)
    1. [launch configurations](https://console.aws.amazon.com/ec2/autoscaling/home?#LaunchConfigurations)
    1. [network interfaces](https://console.aws.amazon.com/ec2/v2/home?#NIC)
1. [ecs](https://console.aws.amazon.com/ecs/home)
1. [efs](https://console.aws.amazon.com/efs/home?#/filesystems)
1. [rds](https://console.aws.amazon.com/rds/home?#databases:)
1. [sqs](https://console.aws.amazon.com/sqs/v2/home)

### AWS Documentation

1. [AWS](https://aws.amazon.com/)
   &gt; [Documentation](https://docs.aws.amazon.com/index.html)
   &gt; [AWS CLI](https://awscli.amazonaws.com/v2/documentation/api/latest/index.html)
    1. [aws](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html)
        1. [cloudformation](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cloudformation/index.html)
            1. [list-stack-resources](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cloudformation/list-stack-resources.html)
        1. [cloudwatch](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cloudwatch/index.html)
        1. [ec2](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/index.html)
            1. [authorize-security-group-ingress](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/authorize-security-group-ingress.html)
            1. [describe-security-groups](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/describe-security-groups.html)
        1. [ecs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/index.html)
            1. [describe-services](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ecs/describe-services.html)
        1. [efs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/efs/index.html)
        1. [logs](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/logs/index.html)
1. [AWS](https://aws.amazon.com/)
   &gt; [Documentation](https://docs.aws.amazon.com/index.html)
   &gt; [Amazon ECS](https://docs.aws.amazon.com/ecs/index.html)
   &gt; [Developer Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
    1. [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
        1. [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
            1. [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
                1. [rm](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-rm.html)
                1. [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
            1. [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-up.html)
        1. [configure](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-configure.html)
        1. [down](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-down.html)
        1. [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
        1. [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-up.html)

### Etc

1. [Installing the Amazon ECS CLI](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html).
1. [Using the awslogs Log Driver](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_awslogs.html)
1. YAML file formats
    1. [Using Docker Compose File Syntax](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-parameters.html)
    1. [Using Amazon ECS Parameters](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-ecsparams.html)
1. [Tutorial: Creating a Cluster with an EC2 Task Using the Amazon ECS CLI](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-cli-tutorial-ec2.html)
