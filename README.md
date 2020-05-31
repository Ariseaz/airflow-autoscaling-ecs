# Airflow Autoscaling ECS

Setup to run Airflow in AWS ECS (Elastic Container Service) Fargate with autoscaling enabled for all services. 
All infrastructure is created with Cloudformation and Secrets are managed by AWS Secrets Manager.

## Requirements
* Create an AWS IAM User for the infrastructure deployment, with admin permissions
* Install AWS CLI running `pip install awscli`
* Setup your IAM User credentials inside `~/.aws/config`
```
    [profile my_aws_profile]
    aws_access_key_id = <my_access_key_id> 
    aws_secret_access_key = <my_secret_access_key>
    region = us-east-1
```
* Create a virtual environment
* Setup env variables in your .zshrc or .bashrc, or in your the terminal session that you are going to use:
```shell script
	export AWS_REGION=us-east-1;
	export AWS_PROFILE=my_aws_profile;
	export ENVIRONMENT=dev;
```

## Deploy Airflow on AWS ECS
To deploy or update your stack run the following command:
```shell script
make cloudformation-deploy
```

To destroy your stack run the following command:
```shell script
make cloudformation-destroy
```

## Features
* Control all Airflow infrastructure from a single `service.yml` file.
* Metadata DB Passwords Managed with AWS Secrets Manager and rotated on a configurable schedule.
* Autoscaling enabled and configurable for all Airflow sub-services (workers, flower, webserver, scheduler)
* 


### Adjust Airflow workers easily: 
```yaml
  workers:
    port: 8793
    cpu: 1024
    memory: 2048
    desiredCount: 2
    autoscaling:
      maxCapacity: 8
      minCapacity: 2
      cpu:
        target: 70
        scaleInCooldown: 60
        scaleOutCooldown: 120
      memory:
        target: 70
        scaleInCooldown: 60
        scaleOutCooldown: 120
```

### Or modify Airflow Metadata DB: 
```yaml
metadataDb:
  instanceType: db.t3.micro
  port: 5432
  dbName: airflow
  engine: aurora-postgresql
  engineMode: provisioned
  engineVersion: 11.7
  family: aurora-postgresql11
  deletionProtection: false
  enableHttpEndpoint: true
  storageEncrypted: true
  enableIAMDatabaseAuthentication: true
  secrets:
    rotationScheduleDays: 30
  parameters:
    maxConnections: 100
```

*Inspired by the work done by [Nicor88](https://github.com/nicor88/aws-ecs-airflow)*