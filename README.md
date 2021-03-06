# Amazon ECS CLI

The ECS Command Line Interface (ECS CLI) is a command line interface for Amazon 
EC2 Container Service (ECS) that provides high level commands to simplify 
creating, updating, and monitoring clusters and tasks from a local development 
environment. The Amazon ECS CLI supports 
[Docker Compose](https://docs.docker.com/compose/), a popular open-source tool 
for defining and running multi-container applications. Use the ECS CLI as part 
of your everyday development and testing cycle as an alternative to the AWS 
Management Console.

For more information about Amazon ECS, see the 
[Amazon ECS Developer Guide](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html).
For information about installing and using the ECS CLI, see the 
[ECS Command Line Interface](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI.html).

The AWS Command Line Interface (AWS CLI) is a unified client for AWS services 
that provides commands for all public API operations. These commands are lower 
level than those provided by the ECS CLI. For more information about supported 
services and to download the AWS Command Line Interface, see the 
[AWS Command Line Interface](http://aws.amazon.com/cli/) product detail page.

## Installing

Download the binary archive for your platform, decompress the archive, and 
install the binary on your `$PATH`. You can use the provided `md5` hash to 
verify the integrity of your download.

* Linux: 
  * [https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-linux-amd64-latest](https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-linux-amd64-latest)
  * [https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-linux-amd64-latest.md5](https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-linux-amd64-latest.md5)
* Macintosh:
  * [https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-darwin-amd64-latest](https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-darwin-amd64-latest)
  * [https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-darwin-amd64-latest.md5](https://s3.amazonaws.com/amazon-ecs-cli/ecs-cli-darwin-amd64-latest.md5)
* Windows:
  * (Not yet implemented)

## Configuring the CLI

Before using the CLI, you need to configure your AWS credentials, the AWS 
region in which to create your cluster, and the name of the ECS cluster to use 
with the `ecs-cli configure` command. These settings are stored in 
`~/.ecs/config`. You can use any existing AWS named profiles in 
`~/.aws/credentials` for your credentials with the `--profile` option.

```
$ ecs-cli help configure
NAME:
   configure - Configures your AWS credentials, the AWS region to use, and the EC2 Container Service cluster name to use with the ECS CLI.

USAGE:
   command configure [command options] [arguments...]

OPTIONS:
   --region, -r 	Specify the AWS Region to use. [$AWS_REGION]
   --access-key 	Specify the AWS access key to use. [$AWS_ACCESS_KEY_ID]
   --secret-key 	Specify the AWS secret key to use. [$AWS_SECRET_ACCESS_KEY]
   --profile, -p 	Specify your AWS credentials with an existing named profile from ~/.aws/credentials. [$AWS_PROFILE]
   --cluster, -c 	Specify the ECS cluster name to use. If the cluster does not exist, it will be created.
```

## Using the CLI
After installing the ECS CLI and configuring your credentials, you are ready to 
create an ECS cluster using the ECS CLI.

```
$ ecs-cli help up
NAME:
   up - Create the ECS Cluster (if it does not already exist) and the AWS resources required to set up the cluster.

USAGE:
   command up [command options] [arguments...]

OPTIONS:
   --keypair 		Specify the name of an existing Amazon EC2 key pair to enable SSH access to the EC2 instances in your cluster.
   --capability-iam	Acknowledge that this command may create IAM resources.
   --size 		[Optional] Specify the number of instances to register to the cluster. The default is 1.
   --azs 		[Optional] Specify a comma-separated list of 2 VPC availability zones in which to create subnets (these AZs must be in the 'available' status). This option is recommended if you do not specify a VPC ID with the --vpc option. WARNING: Leaving this option blank can result in failure to launch container instances if an unavailable AZ is chosen at random.
   --security-group 	[Optional] Specify an existing security group to associate it with container instances. Defaults to creating a new one.
   --cidr 		[Optional] Specify a CIDR/IP range for the security group to use for container instances in your cluster. Defaults to 0.0.0.0/0 if --security-group is not specified
   --port 		[Optional] Specify a port to open on a new security group that is created for your container instances if an existing security group is not specified with the --security-group option. Defaults to port 80.
   --subnets 		[Optional] Specify a comma-separated list of existing VPC Subnet IDs in which to launch your container instances. This option is required if you specify a VPC with the --vpc option.
   --vpc 		[Optional] Specify the ID of an existing VPC in which to launch your container instances. If you specify a VPC ID, you must specify a list of existing subnets in that VPC with the --subnets option. If you do not specify a VPC ID, a new VPC is created with two subnets.
   --instance-type 	[Optional] Specify the EC2 instance type for your container instances.
```

For example, to create an ECS cluster with two Amazon EC2 instances:

```
$ ecs-cli up --keypair my-key --capability-iam --size 2
```

It will take a few minutes to create the resources requested by `ecs-cli up`. 
To see when the cluster is ready to run tasks you can use the AWS CLI to 
confirm the ECS instances are registered:


```
$ aws ecs list-container-instances --cluster your-cluster-name
{
    "containerInstanceArns": [
        "arn:aws:ecs:us-east-1:980116778723:container-instance/6a302e06-0aa6-4bbc-9428-59b17089b887",
        "arn:aws:ecs:us-east-1:980116778723:container-instance/7db3c588-0ef4-49fa-be32-b1e1464f6eb5",
    ]
}

```

**Note:** The default security group created by `ecs-cli up` allows inbound 
traffic on port 80 by default. To allow inbound traffic from a different port, 
specify the port you wish to open with the `--port` option. To add more ports 
to the default security group, go to EC2 Security Groups in the AWS Management 
Console and search for the security group with “ecs-cli”. Add a rule as 
described in 
[the documentation]( http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html#adding-security-group-rule).
Alternatively, you may specify an existing security group ID with the 
`--security-group` option.

Once the cluster is created you can run tasks – groups of containers – on the 
ECS cluster. First, author a 
[Docker Compose configuration file]( https://docs.docker.com/compose). 
You can run the configuration file locally using Docker Compose. Here is an 
example Docker Compose configuration file that creates a web page:

```
web:
  image: amazon/amazon-ecs-sample
  ports:
   - "80:80"
```

To run the configuration file on Amazon ECS use `ecs-cli compose up`. This 
creates an ECS task definition and starts an ECS task. You can see the task 
that is running with `ecs-cli compose ps`, for example:

```
$ ecs-cli compose ps
Name                                      State    Ports
fd8d5a69-87c5-46a4-80b6-51918092e600/web  RUNNING  54.209.244.64:80->80/tcp
```

Navigate your web browser to the task’s IP address to see the sample app 
running in the ECS cluster.

You can also run tasks as services. The ECS service scheduler ensures that the 
specified number of tasks are constantly running and reschedules tasks when a 
task fails (for example, if the underlying container instance fails for some 
reason).

```
$ ecs-cli compose --project-name wordpress-test service create

INFO[0000] Using Task definition                         TaskDefinition=ecscompose-wordpress-test:1
INFO[0000] Created an ECS Service                        serviceName=ecscompose-service-wordpress-test taskDefinition=ecscompose-wordpress-test:1

```

You can then start the tasks in your service with the following command:
`$ ecs-cli compose --project-name wordpress-test service start`

It may take a minute for the tasks to start. You can monitor the progress using 
this command:
```
$ ecs-cli compose --project-name wordpress-test service ps
Name                                            State    Ports
34333aa6-e976-4096-991a-0ec4cd5af5bd/mysql      RUNNING  
34333aa6-e976-4096-991a-0ec4cd5af5bd/wordpress  RUNNING  54.186.138.217:80->80/tcp
```


## ECS CLI Commands

For a complete list of commands, see the 
[ECS CLI documentation](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_CLI.html).


## Building the CLI
### Developing

Running ``scripts/vendor.sh`` creates/updates the vedor/ directory with
dependencies.

For developing code, the correct GOPATH can be printed by running
`./scripts/shared_env` script.

This can be set as GOPATH on the dev box.

### Building

Running `make build` creates a standalone executable in the `bin/local` 
directory

```bash
$ pwd
/home/ubuntu/github/src/github.com/aws/amazon-ecs-cli
$ make build
$ ls bin/local
ecs-cli
```

### Cross-compiling 

The `make docker-build` target will build standalone amd64 executables for
darwin and linux. The output will be in `bin/darwin-amd64` and `bin/linux-amd64`
respectively.

If you have set up the appropriate bootstrap environments, you may also directly
run the `make supported-platforms` target to create standalone amd64 executables
for darwin and linux platforms.

## Testing

### Running tests

Running ``make test`` runs unit tests in the package.

## License

The ECS CLI is distributed under the
[Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0),
see LICENSE and NOTICE for more information.
