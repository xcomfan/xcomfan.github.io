---
layout: page
title: "Terraform"
permalink: /terraform
---

## Set these in your environment to use Terraform

```bash
$ export AWS_ACCESS_KEY_ID=(your access key id)
$ export AWS_SECRET_ACCESS_KEY=(your secret access key)
```

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
* `depends_on` - Normally Terraform automatically figures out your dependency graph, but in rare situations, you have to give it extra hings. For example you have a variable that depends on IP address of a server, but that IP address won't be accessible until a security group is properly configured. This lets you explicitly tell Terraform there is a dependency between the IP address output variable and the security group.

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

Once you have the above definition in place you run `terraform init` again to configure the backend.  The init command is idempotent so you can run it multiple times.

### Locking state files

Need to solve the issue of what happens if two team members run Terraform at the same time.

### Isolating state files

How can you isolate QA/Staging/prod if all your state is in one file?