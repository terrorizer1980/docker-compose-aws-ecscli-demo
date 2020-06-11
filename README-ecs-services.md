# docker-compose-aws-ecscli-demo

This is a copy/paste version of
[Tutorial: Creating a Cluster with an EC2 Task Using the Amazon ECS CLI](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-cli-tutorial-ec2.html)

## Contents

1. [Prerequisites](#prerequisites)
    1. [Install ECS CLI](#install-ecs-cli)
    1. [Multi-factor authentication](#multi-factor-authentication)
    1. [Clone repository](#clone-repository)
1. [Tutorial](#tutorial)
    1. [Identify metadata](#identify-metadata)
    1. [Configure ECS CLI](#configure-ecs-cli)
    1. [Create cluster](#create-cluster)
    1. [View tasks](#view-tasks)
    1. [View services](#view-services)
1. [Cleanup](#cleanup)
    1. [Bring down task](#bring-down-task)
    1. [Bring down cluster](#bring-down-cluster)
    1. [Clean logs](#clean-logs)
    1. [Verify cleanup in AWS console](#verify-cleanup-in-aws-console)
1. [References](#references)

## Prerequisites

### Install ECS CLI

To install `ecs-cli`, follow steps at
[Configuring the Amazon ECS CLI](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html).

### Multi-factor authentication

:thinking: **Optional:** If multi-factor authentication is used to access AWS,
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

1. :pencil2: Set AWS metadata.
   Example:

    ```console
    export AWS_REGION=us-east-1
    export AWS_KEYPAIR=aws-default-key-pair
    ```

#### Identify project

1. :pencil2: Choose a prefix used in AWS object names.
   Example:

    ```console
    export AWS_PROJECT=project01
    ```

#### EULA

To use the Senzing code, you must agree to the End User License Agreement (EULA).

1. :warning: This step is intentionally tricky and not simply copy/paste.
   This ensures that you make a conscious effort to accept the EULA.
   Example:

    <pre>export SENZING_ACCEPT_EULA="&lt;the value from <a href="https://github.com/Senzing/knowledge-base/blob/master/lists/environment-variables.md#senzing_accept_eula">this link</a>&gt;"</pre>

### Configure ECS CLI

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [configure](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-configure.html).
   Example:

    ```console
    ecs-cli configure \
       --cluster ${AWS_PROJECT}-cluster \
       --config-name ${AWS_PROJECT}-config-name \
       --default-launch-type EC2 \
       --region ${AWS_REGION}
    ```

1. :thinking: **Optional:** To view configuration values, see `~/.ecs/config`.

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
      --capability-iam \
      --cluster-config ${AWS_PROJECT}-config-name \
      --force \
      --instance-type t2.large \
      --keypair ${AWS_KEYPAIR} \
      --size 1
    ```

1. :thinking: **Optional:** View changes.
    1. [cloudformation](https://console.aws.amazon.com/cloudformation/home?#/stacks)
    1. [ec2](https://console.aws.amazon.com/ec2/v2/home)
        1. [auto scaling groups](https://console.aws.amazon.com/ec2autoscaling/home?#/details)
        1. [instances](https://console.aws.amazon.com/ec2/v2/home?#Instances)
        1. [launch configurations](https://console.aws.amazon.com/ec2/autoscaling/home?#LaunchConfigurations)
    1. [ecs](https://console.aws.amazon.com/ecs/home)
        1. Select ${AWS_PROJECT}-cluster
        1. Click "Update Cluster" to update information.
        1. Click "ECS instances" tab.

### Open ports

1. Open ports.
    1. View [ec2 instances](https://console.aws.amazon.com/ec2/v2/home?#Instances)
    1. Choose "ECS instance" instance
    1. **Security groups:**, click on security group.
    1. In "Security Groups" dialog, edit "Inbound rules"
    1. Open following ports:
        1. HTTP
        1. SSH
        1. Custom TCP
            1. 5432 - Postgres
            1. 5672 - RabbitMQ service
            1. 8250 - API server
            1. 8251 - Web app
            1. 8254 - Senzing X-Term
            1. 9171 - phpPgAdmin
            1. 9178 - Jupyter notebooks
            1. 15672 - RabbitMQ user interface

### Install Senzing

Install Senzing onto `/opt/senzing`.

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-up.html)
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${AWS_PROJECT}-config-name \
      --ecs-params ${GIT_REPOSITORY_DIR}/ecs-params.yaml \
      --file ${GIT_REPOSITORY_DIR}/docker-compose-init.yaml \
      --project-name ${AWS_PROJECT}-project-name-init \
      up \
        --create-log-groups \
        --launch-type EC2
    ```

1. :thinking: **Optional:** View changes.
    1. [ec2](https://console.aws.amazon.com/ec2/v2/home)
        1. [instances](https://console.aws.amazon.com/ec2/v2/home?#Instances)
    1. [ecs](https://console.aws.amazon.com/ecs/home)
        1. Select ${AWS_PROJECT}-cluster
        1. Click "Update Cluster" to update information.
        1. Click "ECS instances" tab.

### Provision Postgres

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision Postgres database service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${AWS_PROJECT}-config-name \
      --ecs-params ${GIT_REPOSITORY_DIR}/ecs-params.yaml \
      --file ${GIT_REPOSITORY_DIR}/docker-compose-postgres.yaml \
      --project-name ${AWS_PROJECT}-project-name-postgres \
      service up \
        --create-log-groups \
        --launch-type EC2
    ```

1. :thinking: **Optional:** View service definition.
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${AWS_PROJECT}-cluster \
      --services ${AWS_PROJECT}-project-name-postgres
    ```

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address definition.
   Example:

    ```console
    export SENZING_IP_ADDRESS_POSTGRES=$( \
      ecs-cli ps \
        --cluster-config ${AWS_PROJECT}-config-name \
      | grep  postgres \
      | awk '{print $3}' \
      | awk -F \: {'print $1'} \
    )
    ```

### Create Senzing database schema

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-up.html)
   to create Senzing database schema.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${AWS_PROJECT}-config-name \
      --ecs-params ${GIT_REPOSITORY_DIR}/ecs-params.yaml \
      --file ${GIT_REPOSITORY_DIR}/docker-compose-postgres-init.yaml \
      --project-name ${AWS_PROJECT}-project-name-postgres-init \
      up \
        --create-log-groups \
        --launch-type EC2
    ```

### Provision phpPgAdmin

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision phpPgAdmin database client service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${AWS_PROJECT}-config-name \
      --ecs-params ${GIT_REPOSITORY_DIR}/ecs-params.yaml \
      --file ${GIT_REPOSITORY_DIR}/docker-compose-phppgadmin.yaml \
      --project-name ${AWS_PROJECT}-project-name-phppgadmin \
      service up \
        --create-log-groups \
        --launch-type EC2
    ```

1. View phpPgAdmin.
   Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address and port.
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${AWS_PROJECT}-config-name \
    | grep phppgadmin
    ```

### Provision RabbitMQ

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [service](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-service-up.html)
   to provision Postgres database service.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${AWS_PROJECT}-config-name \
      --ecs-params ${GIT_REPOSITORY_DIR}/ecs-params.yaml \
      --file ${GIT_REPOSITORY_DIR}/docker-compose-rabbitmq.yaml \
      --project-name ${AWS_PROJECT}-project-name-rabbitmq \
      service up \
        --create-log-groups \
        --launch-type EC2
    ```

1. :thinking: **Optional:** View service definition.
   Example:

    ```console
    aws ecs describe-services \
      --cluster ${AWS_PROJECT}-cluster \
      --services ${AWS_PROJECT}-project-name-rabbitmq
    ```

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html)
   to find IP address definition.
   Example:

    ```console
    export SENZING_IP_ADDRESS_RABBITMQ=$( \
      ecs-cli ps \
        --cluster-config ${AWS_PROJECT}-config-name \
      | grep  rabbitmq \
      | awk '{print $3}' \
      | awk -F \: {'print $1'} \
    )
    ```

### Mock data generator

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   [up](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-up.html)
   to create Senzing database schema.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${AWS_PROJECT}-config-name \
      --ecs-params ${GIT_REPOSITORY_DIR}/ecs-params.yaml \
      --file ${GIT_REPOSITORY_DIR}/docker-compose-mock-data-generator.yaml \
      --project-name ${AWS_PROJECT}-project-name-mock-data-generator \
      up \
        --create-log-groups \
        --launch-type EC2
    ```

### View tasks

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html).
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${AWS_PROJECT}-config-name
    ```

1. View tasks in AWS Console:
    1. [ecs](https://console.aws.amazon.com/ecs/home)
        1. Select ${AWS_PROJECT}-cluster
        1. Click "Update Cluster" to update information.
        1. Click "Tasks" tab.
1. View logs:
   [cloudwatch](https://console.aws.amazon.com/cloudwatch/home)
   &gt; [log groups](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups)
   &gt; [senzing-docker-compose-aws-ecscli-demo](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups/log-group/senzing-docker-compose-aws-ecscli-demo)

### View services

1. To find IP addresses and ports, run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [ps](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-ps.html).
   Example:

    ```console
    ecs-cli ps \
      --cluster-config ${AWS_PROJECT}-config-name
    ```

   Open a web browser to the various `http://ip-address:port` locations.

1. [Senzing API in Swagger editor](http://editor.swagger.io/?url=https://raw.githubusercontent.com/Senzing/senzing-rest-api/master/senzing-rest-api.yaml)

## Cleanup

### Bring down init task

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   down.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${AWS_PROJECT}-config-name \
      --ecs-params ${GIT_REPOSITORY_DIR}/ecs-params-init.yaml \
      --file ${GIT_REPOSITORY_DIR}/docker-compose-init.yaml \
      --project-name ${AWS_PROJECT}-project-name-init \
      down \
        --cluster-config ${AWS_PROJECT}-config-name
    ```

1. Task is a "job".
   When the task state is `STOPPED`, the job has finished.

### Bring down task

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [compose](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose.html)
   down.
   Example:

    ```console
    ecs-cli compose \
      --cluster-config ${AWS_PROJECT}-config-name \
      --ecs-params ${GIT_REPOSITORY_DIR}/ecs-params.yaml \
      --file ${GIT_REPOSITORY_DIR}/docker-compose.yaml \
      --project-name ${AWS_PROJECT}-project-name-main \
      down \
        --cluster-config ${AWS_PROJECT}-config-name
    ```

### Bring down cluster

1. Run
   [ecs-cli](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_reference.html)
   [down](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-down.html).
   Example:

    ```console
    ecs-cli down \
      --force \
      --cluster-config ${AWS_PROJECT}-config-name
    ```

### Clean logs

1. Logs:
   [cloudwatch](https://console.aws.amazon.com/cloudwatch/home)
   &gt; [log groups](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups)
   &gt; [senzing-docker-compose-aws-ecscli-demo](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups/log-group/senzing-docker-compose-aws-ecscli-demo)

### Verify cleanup in AWS console

1. Verify in AWS Console:
    1. [ec2](https://console.aws.amazon.com/ec2/v2/home)
        1. [auto scaling groups](https://console.aws.amazon.com/ec2/autoscaling/home?#AutoScalingGroups)
        1. [instances](https://console.aws.amazon.com/ec2/v2/home?#Instances)
        1. [launch configurations](https://console.aws.amazon.com/ec2/autoscaling/home?#LaunchConfigurations)
        1. [network interfaces](https://console.aws.amazon.com/ec2/v2/home?#NIC)
    1. [cloudformation](https://console.aws.amazon.com/cloudformation/home?#/stacks)
    1. [cloudwatch](https://console.aws.amazon.com/cloudwatch/home)
        1. [log groups](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups)
    1. [ecs](https://console.aws.amazon.com/ecs/home)

## References

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
1. [AWS](https://aws.amazon.com/)
   &gt; [Documentation](https://docs.aws.amazon.com/index.html)
   &gt; [AWS CLI](https://docs.aws.amazon.com/cli/)
    1. [aws](https://docs.aws.amazon.com/cli/latest/reference/index.html)
        1. [ecs](https://docs.aws.amazon.com/cli/latest/reference/ecs/index.html)
            1. [describe-services](https://docs.aws.amazon.com/cli/latest/reference/ecs/describe-services.html)
1. AWS console
    1. [cloudformation](https://console.aws.amazon.com/cloudformation/home?#/stacks)
    1. [cloudwatch](https://console.aws.amazon.com/cloudwatch/home)
        1. [log groups](https://console.aws.amazon.com/cloudwatch/home?#logsV2:log-groups)
    1. [ec2](https://console.aws.amazon.com/ec2/v2/home)
        1. [auto scaling groups](https://console.aws.amazon.com/ec2autoscaling/home?#/details)
        1. [instances](https://console.aws.amazon.com/ec2/v2/home?#Instances)
        1. [launch configurations](https://console.aws.amazon.com/ec2/autoscaling/home?#LaunchConfigurations)
        1. [network interfaces](https://console.aws.amazon.com/ec2/v2/home?#NIC)
    1. [ecs](https://console.aws.amazon.com/ecs/home)
1. [Installing the Amazon ECS CLI](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI_installation.html).
1. [Using the awslogs Log Driver](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_awslogs.html)
1. YAML file formats
    1. [Using Docker Compose File Syntax](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-parameters.html)
    1. [Using Amazon ECS Parameters](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cmd-ecs-cli-compose-ecsparams.html)
