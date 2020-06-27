# secrets_manager_aws

A demo template on using CloudFormation to create an RDS database and Secrets Manager. The ability to rotate a secret is also a key portion of this template.

## Template Parameters

This template uses multiple default parameters, most of which use default values. It does, however, have a parameter that you will need to update the value of before deploying the stack.

| Parameter | Description | Current Value |
| --------- |------------ |-------------- |
| VPCStackName | The name of your parent stack that builds a VPC network with public and private subnets. | sharina-cf-built |

## How to Deploy

### Prerequisites

If you prefer to deploy this stack via the command line, you will need the AWS CLI.

You will need to have already deployed a stack that builds out a VPC network with public and private subnets in three AWS Availability Zones. The parent template used to accomplish this for this template can be found [in GitHub](https://github.com/1Strategy/vpc-starter-template). 

You will also need to create a folder called "paramaters," and within it, a file called "create_params.json" file. Within this file, add the parameter noted above in order to deploy from the AWS CLI. The format for the Json should be as follows, give the `create-stack` command:

```json
[
    {
        "ParameterKey": "VPCStackName",
        "ParameterValue": "sharina-cf-built"
    }
]
```

### Validate/Lint Stack

```shell
aws cloudformation validate-template --template-body file://ramp-up-project-rotation.yaml
```

### Create the Stack

If you have multiple profiles you could deploy to, make sure to include the command `--profile <profile-name>` into your command.

```shell
aws cloudformation create-stack \
--template-body file://templates/ramp-up-project-rotation.yaml \
--parameters file://parameters/create_params.json \
--stack-name demo-db-secrets \
--capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND \
--disable-rollback
```

To update the stack, use the `update-stack` command:

```shell
aws cloudformation update-stack \
--template-body file://templates/ramp-up-project-rotation.yaml \
--parameters file://parameters/create_params.json \
--stack-name demo-db-secrets \
--capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND
```

## Install PSQL onto the Webserver

If you wish, here are some additional follow-up steps to take, via your terminal, to install PostgreSQL to your webserver. This will allow you to interact with your database using SQL. Note that your username and password will be located in AWS Secrets Manager.
SSH into your webserver via your bastion host

```shell
ssh -J ec2-user@bastionHostIPAddress ec2-user@webserverIPAddress -i publicKeyFileName.pem
```

Do an update

```shell
sudo yum update
```

Install PostgreSQL.

* Which version of PostgreSQL you get will depend on the version of the distribution. More about this [here](https://www.postgresql.org/download/linux/redhat/).

```shell
sudo yum install postgresql-server
```

Initialize your database:

```shell
sudo postgresql-setup initdb
```

Connect to your AWS PostgreSQL DB:

* You'll find your Database Name (DBName) in the AWS console, under the Configuration tab, after you have clicked on your Database link (in your list of Databases).

```shell
psql -h databaseEndPointAddress -U yourDBUsername yourDBName
```

Interact with your DB as usual, using SQL.

## Lambda

Check what Lambdas exist in your AWS account:

```
aws lambda list-functions --profile training
```

## Contributors

Sharina Stubbs
Many thanks to the following for help with debugging, providing suggestions, and sharing their knowledge.

* Alexandra Shumway
* Doug Ireton
* Julie Erlemeier
* Stephanie Lingwood

## Resources Used

AWS Docs:

* [ImportValue](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-importvalue.html)

RDS:

* [About Creating a DB in a VPC](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_VPC.WorkingWithRDSInstanceinaVPC.html)
* [Database Security Group](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-rds-security-group.html)
* [Database Cloudformation Doc](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-rds-database-instance.html)
* [Database storage types](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Storage.html)
* [RDS Automated Backups](https://aws.amazon.com/rds/details/backup/)
* [Download and Install PostgreSQL on the instance](https://github.com/snowplow/snowplow/wiki/Setting-up-PostgreSQL)
* [Downloading PostgreSQL - Linux downloads, Red Hat family](https://www.postgresql.org/download/linux/redhat/)
* [Using Pseudoparameters](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/pseudo-parameter-reference.html)
* [Using Service-Linked Roles for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/UsingWithRDS.IAM.ServiceLinkedRoles.html)

Blog Posts and Resources Outside AWS:

* [Hints on Using Join and Sub](https://theburningmonk.com/2019/05/cloudformation-protip-use-fnsub-instead-of-fnjoin/)

Secrets Manager:

* [Example Template from AWS](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-secretsmanager-rotationschedule.html#cfn-secretsmanager-rotationschedule-rotationlambdaarn)
* [CloudFormation and Secrets Manager](https://aws.amazon.com/blogs/security/how-to-create-and-retrieve-secrets-managed-in-aws-secrets-manager-using-aws-cloudformation-template/)
* [Rotation Schedule for Secrets Manager](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-secretsmanager-rotationschedule.html#cfn-secretsmanager-rotationschedule-rotationlambdaarn)
* [Templates for Rotation Lambda Functions](https://docs.aws.amazon.com/secretsmanager/latest/userguide/reference_available-rotation-templates.html)
* [Dynamic References and Secrets Manager](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/dynamic-references.html)
* [Sample CloudFormation template for Rotation](https://github.com/aws-samples/aws-secrets-manager-rotation-lambdas/blob/master/SecretsManagerRDSPostgreSQLRotationSingleUser/lambda_function.py)
* [CLI commands for Secrets Manager](https://aws.amazon.com/premiumsupport/knowledge-center/lambda-function-secrets-manager/)