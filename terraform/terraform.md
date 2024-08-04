---
layout: page
title: "Terraform"
permalink: /terraform
---

[Installing and Configuring]({% link terraform/install_and_configure.md %})

## Terraform commands

`terraform plan` - parse your HCL code and show what will happen.
`terraform apply` - execute the plan
`terraform destroy` - tear down 
`terraform graph` - Show the dependency graph between the objects in your HCL code. You can use Graphviz or [GraphvizOnline](http://dreampuf.github.io/GraphvizOnline) to visualize the otput.

## AdHoc notes to organize

The general syntax for creating a resource in Terraform is as below where `PROVIDER` is the name of the provider (such as aws), `TYPE` is the type of resource to create in that provider (e.g., `instance`) and `NAME` is an identifier you can use throughout the Terraform code to refer to this resource (e.g., `my_instance`). `CONFIG` consists of one or more arguments that are specific to that resource.

```hcl
resource "<PROVIDER>_<TYPE>" "<NAME>" {
 [CONFIG …]
}
```

Docs can be found at <https://developer.hashicorp.com/terraform/docs> and <https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance>

In your `.gitignore` file for a Terraform repo you should put the following...

```text
.terraform
*.tfstate
*.tfstate.backup
```

You have a good example of this in your terraform_practice directory, but `<<-EOF` and `EOF` are Terraform's **heredoc** syntax which allows you to create multi line strings without having to insert `\n` characters all over the place.

`user_data_replace_on_change` parameter if se to `true` when you change the `user_data` parameter and run `apply` will terminate the original instance and launch a totally new one.  Terraform's default behavior is to update the original instance in place. Since user data only runs on first boot you need this parameter if you want to change behavior that occurs in the user_data script.

To use a reference in HCL language you use the format of `<PROVIDER>_<TYPE>.<NAME>.<ATTRIBUTE>` for example in your example you created a security group instance called "instance" and you can reference it with `aws_security_group.instance.id` (see the example for full details)

`+-` in a `terraform plan` output means "replace". Look for text "forces replacement" in the plan output to figure out what is forcing Terraform to do a replacement.

To allow you to not have to repeat yourself, Terraform lets you to define **input variables**. The syntax for an input variable is ...

```hcl
variable "NAME" {
    [CONFIG ...]
}
```

The body of the variable declaration can contain the following optional parameters.

* `description` - good idea to use this parameter to document how a variable is used.
* `default` - There are a number of ways to provide a value for the variable, including passing it in at the command line (using the `-var` option), via a file (using the `-var-file` option), or via an environment variable (Terraform looks for environment variables of the name `TF_VAR_<variable_name>`) If no value is passed in the variable will fall back to its default value.
* `type` - Allows you to enforce type constraints. Type supported are `string`, `number`, `bool`, `list`, `map`, `set`, `object`, `tuple` and `any`
* `validation` - Allows you to define custom validation rules for the input variable that go beyond basic types checks, such as enforcing minimum or maximum values on a number.
* `sensitive` - If set to `true` on an input variable, Terraform will not log it when you run `plan` or `apply`. Should be used on secrets passed into Terraform code via variables.

You can also define output variables using syntax ...

```hcl
output "<NAME>" {
  value = <VALUE>
  [CONFIG ...]
}
```

Config can contain

* `description`
* `sensitive`
* `depends_on` - Normally Terraform automatically figures out your dependency graph, but in rare situations, you have to give it extra hints. For example you have a variable that depends on IP address of a server, but that IP address won't be accessible until a security group is properly configured. This lets you explicitly tell Terraform there is a dependency between the IP address output variable and the security group.

Note that the ASG uses a reference to fill in the launch configuration name. This leads to a problem: launch configurations are immutable, so if you change any parameter of your launch configuration, Terraform will try to replace it. Normally, when replacing a resource, Terraform would delete the old resource first and then creates its replacement, but because your ASG now has a reference to the old resource, Terraform won’t be able to delete it.
To solve this problem, you can use a lifecycle setting. Every Terraform resource supports several lifecycle settings that configure how that resource is created, updated, and/or deleted. A particularly useful lifecycle setting is create_before_destroy. If you set create_before_destroy to true, Terraform will invert the order in which it replaces resources, creating the replacement resource first (including updating any references that were pointing at the old resource to point to the replacement) and then deleting the old resource. Add the lifecycle block to your aws_launch_configuration as follows:

There’s also one other parameter that you need to add to your ASG to make it work: subnet_ids. This parameter specifies to the ASG into which VPC subnets the EC2 Instances should be deployed. Each subnet lives in an isolated AWS AZ (that is, isolated datacenter), so by deploying your Instances across multiple subnets, you ensure that your service can keep running even if some of the datacenters have an outage. You could hardcode the list of subnets, but that won’t be maintainable or portable, so a better option is to use data sources to get the list of subnets in your AWS account.

A data source represents a piece of read-only information that is fetched from the provider (in this case, AWS) every time you run Terraform. Adding a data source to your Terraform configurations does not create anything new; it’s just a way to query the provider’s APIs for data and to make that data available to the rest of your Terraform code. Each Terraform provider exposes a variety of data sources. For example, the AWS Provider includes data sources to look up VPC data, subnet data, AMI IDs, IP address ranges, the current user’s identity, and much more.

The syntax for using a data source is very similar to the syntax of a resource:

```hcl
data "<PROVIDER>_<TYPE>" "<NAME>" {
  [CONFIG ...]
}
```

Here, PROVIDER is the name of a provider (e.g., aws), TYPE is the type of data source you want to use (e.g., vpc), NAME is an identifier you can use throughout the Terraform code to refer to this data source, and CONFIG consists of one or more arguments that are specific to that data source. For example, here is how you can use the aws_vpc data source to look up the data for your Default VPC:

```hcl
data "aws_vpc" "default" {
  default = true
}
```

Note that with data sources, the arguments you pass in are typically search filters that indicate to the data source what information you’re looking for. With the aws_vpc data source, the only filter you need is default = true, which directs Terraform to look up the Default VPC in your AWS account.

To get the data out of a data source, you use the following attribute reference syntax:

```hcl
data.<PROVIDER>_<TYPE>.<NAME>.<ATTRIBUTE>
```

For example, to get the ID of the VPC from the aws_vpc data source, you would use the following: `data.aws_vpc.default.id`

You can combine this with another data source, aws_subnets, to look up the subnets within that VPC:

```hcl
data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}
```

## Managing terraform state

There are 3 issues that need to be addressed when dealing with Terraform state files. They are in the following sections.

### Shared storage of state files

The issue of each team member needs to be able to access the state file if you are working as a team on your infrastructure.

Storing state file in Git is a bad idea because...

* Manual error - Its too easy to forget to pull down the latest changes before running Terraform or forget to push your changes after running Terraform.
* Locking - There is no locking that would prevent two people from running `terraform apply` on the same state file at the same time.
* Secrets - All data in Terraform state files is stored in plain text. Certain resources store sensitive data. For example `aws_db_instance` resource will store the username and password for the database in the state file in plain text.  This should not go into source control.

Instead of version control you should use Terraform's built-in support for [remote backends](https://developer.hashicorp.com/terraform/language/settings/backends/configuration). 

A Terrafomrm **backend** determines how Terraform loads and stores state. The default backend is the **local backend** which stores the state file on your local disk. **Remote backends** allow you to store the state file in a remote, shared store. There are a number of remote backends that are supported including S3, Azure Storage, Google Cloud Storage, and HashiCorp's Terraform Cloud and Terraform Enterprise. 

Remote backends solve the 3 issues.

* Manual error - after remote backend is configured Terraform will automatically load the state file from that backend every time run `plan` or `apply` and will automatically store the sate file in that backend after each `apply` so there is not chance for manual error.
* Locking - Most of the remote backends support locking. To run `terraform apply`. You can run `apply` with the `--lock-timeout=<TIME>` parameter to tell Terraform to give up trying to get lock after certain amount of time for example `-lock-timeout=10m`.
* Secrets - Most remote backends natively support encryption in rest and transit. They also usually have a way to restrict access.

#### Using S3 as remote backend

S3 is your best bet for backend.  Its managed, has great availability, supports encryption, supports locking via DynamoDB, supports versioning so every revision is stored and you can roll back to an older version if something goes wrong and its cheap to use.

First you need to create an S3 bucket. You can use Terraform for this but keep it as a separate directory.

```hcl
resource "aws_s3_bucket" "terraform_state" {
  bucket = "terraform-up-and-running-state"
 
  # Prevent accidental deletion of this S3 bucket
  lifecycle {
    prevent_destroy = true
  }
}

# Enable versioning on the bucket
resource "aws_s3_bucket_versioning" "enabled" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

# Enable encryption on the bucket
resource "aws_s3_bucket_server_side_encryption_configuration" "default" {
  bucket = aws_s3_bucket.terraform_state.id

  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

# Block public access to the bucket
resource "aws_s3_bucket_public_access_block" "public_access" {
  bucket                  = aws_s3_bucket.terraform_state.id
  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}

# Create DynamoDB with LockID primary key.
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-up-and-running-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}
```

Use `terraform init` and `terraform apply` to deploy the S3 bucket and DynamoDB.  Now you need to configure Terraform to use the backend storage.

You need to add a backend configuration to your Terraform code. This is a configuration for Terraform itself so it resides in a `terraform` block and has the syntax below with `BACKEND_NAME` is the backend you want to use (for example s3) and `CONFIG` consists of arguments that are specific to that backend.

```hcl
terraform {
    backend "<BACKEND_NAME>" {
        [CONFIG...]
    }
}
```

below is the configuration for an S3 bucket.  `key` is the file path within the s3 bucket where the Terrafrom state file should be written. `encrypt` is telling Terraform to encrypt the state, so you get both s3 and Terraform encryption.

```hcl
terraform {
  backend "s3" {
    # Replace this with your bucket name!
    bucket         = "terraform-up-and-running-state"
    key            = "global/s3/terraform.tfstate"
    region         = "us-east-2"

    # Replace this with your DynamoDB table name!
    dynamodb_table = "terraform-up-and-running-locks"
    encrypt        = true
  }
}
```

Once you have the above definition in place you run `terraform init` again to configure the backend.  The init command is idempotent so you can run it multiple times. After this command you will store your config in the s3 bucket.

You can use the below to confirm things are working. Run `terraform apply` after you add them.

```hcl
output "s3_bucket_arn" {
  value       = aws_s3_bucket.terraform_state.arn
  description = "The ARN of the S3 bucket"
}

output "dynamodb_table_name" {
  value       = aws_dynamodb_table.terraform_locks.name
  description = "The name of the DynamoDB table"
}
```

#### Limitations with Terraform Backends

The fist limitation is the chicken egg situation of using Terraform to create the S3 bucket to store your Terraform state. You first write the Terraform code to create the s3 bucket and Dynamo DB table and deploy that code with a local backend. You then add the remote backend to your Terraform code and run `terraform init` to cpy your local state to S3. If you ever want to delete the S3 bucket and DynamoDB table, you'd have to do this two step process in reverse.  Fist remove the backend in your Terraform code and run `terraform init` to copy the Terraform state back to local disk. Then you run `terraform destroy` to delete the S3 bucket and DynamoDB table.

Note that you can share a single S3 bucket and DynamoDB table across all of your Terraform code.

The second limitation is that the `backend` block in Terraform does not allow you to use any variables or references. This means that you need to manually copy and past the S3 bucket name, region, and DynamoDB table name, etc into every one of your Terraform modules. You also must be very careful not to copy and paste the `key` value but unsure a unique `key` for every Terraform module you deploy so that you don't accidentally over write the state of some other module. One option for reducing the copy and paste is to use **partial configurations**  where you omit certain parameters from the backend configuration in your Terraform code and instead pass those in via `-backend-config` command-line arguments when calling `terraform init`. For example you can extract the repeated arguments such as `bucket` and `region`, into a separate file called `backend.hcl`

```hcl
# backend.hcl
bucket         = "terraform-up-and-running-state"
region         = "us-east-2"
dynamodb_table = "terraform-up-and-running-locks"
encrypt        = true
```

Only the `key` parameter remains in the Terraform code, since you still need to set a different `key` for each module.

```hcl
# Partial configuration. The other settings (e.g., bucket, region) 
# will be passed in from a file via -backend-config arguments to 
# 'terraform init'
terraform {
  backend "s3" {
    key = "example/terraform.tfstate"
  }
}
```

To put all your partial configurations together, run `terraform init` with the `-backend-config` argument as in `terraform init -backend-config=backend.hcl`. Terraform will merge the partial configuration in `backend.hcl` with the partial configuration in your Terraform code to produce the full configuration.

Another option to reduce the copy pasting is to use [Terragrunt](https://terragrunt.gruntwork.io) an open source tool that can help keep your entire `backend` configuration DRY (Don't Repeat Yourself) by defining all the basic `backend` settings in one file and automatically setting the `key` argument to the relative folder path of the module`


### Locking state files

Need to solve the issue of what happens if two team members run Terraform at the same time. (I believe the backend solves this for us need to remove section or clarify in rewrite)

### Isolating state files

If you keep all your Terraform code in a single file or directory all your Terraform state will also end up in one file which means one break can break everything. You want to separate your dev, test staging etc environments. There are two ways you can isolate environments.

* Isolation via workspaces - useful for quick isolated tests on the same configuration
* Isolation via file layout - Useful for production use cases for which you need strong separation between environments.

#### Isolation via workspaces

[Terraform workspaces](https://www.terraform.io/language/state/workspaces) allow you to store your Terraform state in multiple, separate, named workspaces. You use the default workspace by default. To create a new workspace or switch between workspaces, you use the `terraform workspace` commands. You can see which workspace you are using with the `terraform workspace show` command. To create a new workspace use a command similar to `terraform workspace new example1`. The state files in each workspace are isolated from each other. For example if your create an EC2 instance in default workspace that instance will not be there in the example workspace state file and thus a new instance will be created.  To switch workspaces you would use the `terraform workplace select example1`. On the backend in the s3 bucket Terraform will create a directory structure with a directory for each your workspaces in the `env` directory.

This is handy for when you have a Terraform module deployed and want to do some experiments.

Below is an example of using a ternary expression to set instancy type based on which workspace you are in.

```hcl
resource "aws_instance" "example" {
  ami           = "ami-0fb653ca2d3203ac1"
  instance_type = (
    terraform.workspace == "default" ? "t2.medium" : "t2.micro"
  )
}
```

Workspaces are good for quick experiments, but have some drawbacks.

* State files are stored in the same backend which means same authentication and access controls are used for all the workspaces. This makes them unsuitable for isolating environments.
* Workspaces are not visible in code or in console unless you check with `terraform workspace` command. When browsing a module the code can be deployed to one or 10 workspaces which makes it hard to track your infrastructure.
* Its easy to forget what workspace you are in and deploy to the wrong one or worse run terraform destroy in production workspace instead of staging one. As same auth is used for both there is no safety net.

#### Isolation via file layout

To achieve full isolation between environments, you need to 1. Put the Terraform configuration files for each environment into a separate folder. For example, all of the configurations for the staging environment can be in a folder called stage and all the configurations for the production environment can be in a folder called prod. 2. Configure a different backend for each environment, using different authentication mechanisms and access controls. Each environment could live in separate AWS account with separate s3 buckets as a backend.

With this approach its clear which environment you are making changes in due to the separate folders and separate authentication for each environment makes it less likely that a screw up in one environment will impact another. 

You can further isolate things into components. For example the networking configuration rarely changes so you can have the be separate from the more frequently changing web servers. Its a good idea to break these out into separate state files.

Below is a recommended file layour.

```text
stage
    vpc
    services
        frontend-app
        backend-app
            variables.tf
            outputs.tf
            main.tf
        data-storate
            mysql
            redis
prod
    vpc
    services
        frontend-app
        backend-app
    data-storage
        mysql
        redis
mgmt
    vpc
    services
    bastion-host
    jenkins
global
    iam
    s3
```

When you run Terraform it simply looks for files in the current directory with the `.tf` extension so filenames are up to you.

Some options beyond the conventions above

* `dependencies.tf` - its common to put all your data sources in a `dependencies.tf` file to make it easier to see what external things the code depends on.
* `providers.tf` - You may want to put your providers block into `providers.tf` so you can see at a glance what providers the coded talks to and what authentication you have to provide.
* `main-xxx.tf` - break up the main file into some smaller logical resources. For example main-iam.tf main-s3.tf. 

A drawback of the breaking things into directory is you can no longer run one single file which will bring up the entire environment. You can work around this with `Terragrunt` using the `run-all` command.

#### The terraform_remote_state data source

Similar to the `aws_subnets` data source we used before to get a list of subnets in our VPC, there is another data source that is particularly useful when working with state. The `terraform_remote_state` can be used to fetch the Terraform state file stored by another set of Terraform configurations.

Lets say we have the following content in the file `stage/data-stores/mysql/main.tf` in our directory structure.

```terraform
provider "aws" {
    region = "us-east-2"
}

resource "aws_db_instance" "example" {
  identifier_prefix   = "terraform-up-and-running"
  engine              = "mysql"
  allocated_storage   = 10
  instance_class      = "db.t2.micro"
  skip_final_snapshot = true
  db_name             = "example_database"
  # How should we set the username and password?
  username = var.db_username
  password = var.db_password
}
```

Note that above we are not specifying the username and password.  In a later section we will cover how to manage secretes when working with Terraform, but for now we will just keep the secrets in a separate file without default values so that we are prompted for them.

```terraform
# stage/data-stores/mysql/variables.tf
variable "db_username" {
  description = "The username for the database"
  type        = string
  sensitive   = true
}

variable "db_password" {
  description = "The password for the database"
  type        = string
  sensitive   = true
}
```

Now we configure the module to store its state in S3 bucket we created earlier.

```terraform
terraform {
  backend "s3" {
    # Replace this with your bucket name!
    bucket         = "terraform-up-and-running-state"
    key            = "stage/data-stores/mysql/terraform.tfstate"
    region         = "us-east-2"

    # Replace this with your DynamoDB table name!
    dynamodb_table = "terraform-up-and-running-locks"
    encrypt        = true
  }
}
```

Finally we add two output variables

```terraform
# stage/data-stores/mysql/output.tf
output "address" {
  value       = aws_db_instance.example.address
  description = "Connect to the database at this endpoint"
}

output "port" {
  value       = aws_db_instance.example.port
  description = "The port the database is listening on"
}
```

You can pass the values in when you run the code at the prompt or you can set them in your shell environment.

```bash
$ export TF_VAR_db_username="(YOUR_DB_USERNAME)"
$ export TF_VAR_db_password="(YOUR_DB_PASSWORD)"
```

Now you can run `terraform init` and `terraform apply` to create the database.

Now if we go back to our cluster code, we can get the web server to read those outputs (for the database) from the state file using `terraform_remote_state`

```terraform
data "terraform_remote_state" "db" {
  backend = "s3"

  config = {
    bucket = "(YOUR_BUCKET_NAME)"
    key    = "stage/data-stores/mysql/terraform.tfstate"
    region = "us-east-2"
  }
}
```

Like all Terraform data sources, the data returned by `terraform_remote_state` is read only.   Your read the data with the syntax `data.terraform_remote_state.<NAME>.outputs.<ATTRIBUTE>` for example ...

```bash
user_data = <<EOF
#!/bin/bash
echo "Hello, World" >> index.html
echo "${data.terraform_remote_state.db.outputs.address}">>index.html
echo "${data.terraform_remote_state.db.outputs.port}">>index.html
nohup busybox httpd -f -p ${var.server_port} &
EOF
```

A side not here its kind of a pain to have your bash script inside of your terraform code.  You can sue the `template` built in functionality to externalize the script.

You would use the Terraform function (there are other functions you can explore) `templatefile(<PATH>, <VARS>)` to do this. 

We can put our bash script int ot he file `stage/services/webserver-cluster/user-data.sh` as in the example below. Not that `${...}` is used where we need to substitute variables. The script will only have access to variables passed in via the second function parameter.  The script below also added some HTML style syntax to make the output look nicer.

```bash
#!/bin/bash

cat > index.html <<EOF
<h1>Hello, World</h1>
<p>DB address: ${db_address}</p>
<p>DB port: ${db_port}</p>
EOF

nohup busybox httpd -f -p ${server_port} &
```

Once we have script created, we need to update the `user_data` parameter of the `aws_launch_configuration` resource to call the `templatefile` function and pass in the variables it needs as a map.

```terraform
resource "aws_launch_configuration" "example" {
  image_id        = "ami-0fb653ca2d3203ac1"
  instance_type   = "t2.micro"
  security_groups = [aws_security_group.instance.id]

  # Render the User Data script as a template
  user_data = templatefile("user-data.sh", {
    server_port = var.server_port
    db_address  = data.terraform_remote_state.db.outputs.address
    db_port     = data.terraform_remote_state.db.outputs.port
  })

  # Required when using a launch configuration with an ASG.
  lifecycle {
    create_before_destroy = true
  }
}
```

You can experiment with Terraform functions using the `terraform console` command. This will give you an interactive console where you can experiment. The Terraform console is read only so you don't need to worry about accidentally changing infrastructure state.