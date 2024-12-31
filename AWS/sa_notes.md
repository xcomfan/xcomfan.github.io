---
layout: page
title: "AWS SA Notes Monolith"
permalink: /aws/sa_notes
---

## Intro And Course Details

[GitHub repo](https://github.com/acantril/aws-sa-pro) for the course should have some good examples, and resources for the course.

## AWS Accounts

An AWS account is a container for identities (users you log in with) and AWS resources. Each AWS account needs to have a unique email address and a payment card assigned to it (card can be same for many accounts). The unique email of the account is tied to the **root user**. Root user is always created with an AWS account, they have full authority over that account and cannot be restricted.

Its a good idea to keep separate accounts for dev, prod, etc. because if a credential leaks or someone make a mistake the damage is contained to just that account.

A trick you can use to quickly get multiple emails is something like `catguy@gmail.com` being your email, gmail lets you use `+` to make alternate emails for example `catguy+AWSAccount1@gmail.com` and `catguy+AWSAccount2@gmail.com` will go to the same gmail address. This is a dynamic alias.

When logged in as root user you should go to account and under IAM User and Role Access to Billing Information check the checkbox that enables it. Without this even if you provide IAM access to billing information and IAM account would not be able to see it.

### IAM Access Keys

IAM Keys do not expire or rotate unless you explicitly decide to rotate them.

An IAM User has **1** username and **1** password. He can have no more than **two access keys**. Rationale for two is for rotating keys.

Access keys can be created, deleted, made inactive or made active.

IAM keys are only used by IAM users. Nothing else uses IAM access keys.

## AWS CLI Configuration

`aws configure --profile <name_of_profile>` this will prompt your for access key and secret as well as region (you can just hit enter to choose the default output format).

## Organizations

You start with an standard AWS account and use it to create an organization. The organization you create does not belong to this account, but the account is the **management account** for the organization. This is an important concept to understand. Management account may be referred to as master account in some older documentation.

Other accounts can be invited into the organization. Each account will need to approve the invite. Once accept the invitation they go from being standard accounts to being **member accounts**. Member accounts will have their billing passed up to the management account. This works with consolidation of reservations and volume discounts. You can also create a new account directly within an organization.

You can arrange AWS accounts within an organization in a hierarchial manner. All of the accounts will be in an **Organizational Root** container (don't confuse this with root user its just a container) and within that container you can have a hierarchy of **Organizations Units (OUs)**.

A common set up is to have one account that you log into via either regular AWS login or via federation and you role switch from that account into other accounts. The way you would do this is choose Another AWS account as the entity when creating a Role. You will also see this relationship in the trust relationships of the account.

### Service Control Policies

Service Control Policies (SCPs) are a feature of AWS Organizations which allow restriction to be placed on member accounts in the form of boundaries.

An SCP is a JSON policy document which can be attached to an organization as a whole, or they can be attached to one or more Organizational Units (OUs), or to an individual AWS account.

SCPs inherit down the organization tree. So for example if its attached to an organization it affects all accounts in the organization if its attached to an OU it affects all accounts in the OU. If attached to an account, they only impact that account. Note that a management account is not effected by SCPs. For this reason its a good practice to not have resources in your management account as that account cannot be restricted.

SCPs are **account permission boundaries**, they limit what the account (including the account root user) can do.

You can use SCPs to limit the size of EC2 instances that an account can create or the region in which an account can create resources.

SCPs **do not** grant any permissions they define a limit of what is and isn's allowed in an account.

You can create SCPs in an allow list manner (where you have to specifically identify what is allowed) or a Deny list manner (where you specifically say what cannot be done). AWS creates a FullAWSAccess policy by default when you enable SCPs. Without the FullAWSAccess default policy access would just be denied. Any time there is a conflict between an allow statement and a deny statement the deny will always take precedence. If you want to use an explicit allow you have to remove the FullAWSAccess policy first.

Below is an example so an SCP policy which allows everything but denies S3 access.

```json
"Version": "202-10-17",
"Statement": [
    {
        "Effect": "Allow",
        "Action": "*",
        "Resource": "*"
    },
    {
        "Effect": "Deny",
        "Action": "s3:*",
        "Resource": "*"
    }
]
```

## Security Token Service STS

STS generates temporary credentials. These credentials expire and they don't belong to the identity.

STS credentials do not need to have access to the full range of permissions a role being assumed provides. You can configure it to grant a subset of permissions.

Temporary credentials are requests by another identity. This can be an AWS identity (such as an IAM role) or an external identify via web identity federation. The entity making the STS request does not necessarily need to be a user. It can be another AWS service.

Trust policy on a role is what specifies who can assume that role.

STS generates temporary credentials consisting of:

* AccessKeyId - Unique ID of the credentials
* Expiration - Date and time of credential expiration
* SecretAccessKey - Used to sign requests
* SessionToken - Unique token which must be passed with all requests.

These temporary credentials authorize access based on the **permission policy**.

## Revoking Temporary Credentials

You cannot expire a temporary credential. It si valid until it expires.

If fore example a temporary credential is leaked, you don't want to change the permissions of the policy of the role that credentials are assuming as that would impact anyone trying to use that role. What you want to do is modify the permission policy to revoke older sessions (all sessions generate before a certain time). This will force everyone to have to re assume the role via STS and get new credentials and make the leaked credentials invalid.

Belows is an example of the change you would make to the IAM role to make this kind of invalidation. (This policy gets created and attached to the role if you click revoke active sessions in the console)

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": [
                "*"
            ],
            "Resource": [
                "*"
            ],
            "Condition": {
                "DateLessThan": {
                    "aws:TokenIssueTime": "2022-05-14T05:34:15.058Z"
                }
            }
        }
    ]
}
```

## IAM Policies

`"NotAction"` within a policy is a way of saying instead of a list of actions use/apply everything except these. Kind of an inverse of `"Action"`

CloudFront, IAM, Route 53 and Support are global services. For Global services; event though they are global you interact with them as though they are in `us-east-1` (unless the docs say otherwise). This is important to consider when writing certain IAM policy.  Below is an example of this. It denies the use of these services unless you are making the call from the two approved regions.

```json
{
    "Version": "2012=10=17",
    "Statement": [
        {
            "Sid": "DenyNonApprovedRegions",
            "Effect": "Deny",
            "NotAction": [
                "cloudfront:*",
                "iam:*",
                "route53:*",
                "support:*"
            ],
            "Resource": "*",
            "Condition": {
                "StringNotEquals": {
                    "aws:RequestedRegion": [
                        "ap-southeast-2",
                        "eu-west-1"
                    ]
                }
            }
        }
    ]
}
```

Certain actions must have a `*` as their resource. Below is one such example that applies to getting s3 bucket information. Not that these permission let you list but not look into buckets. Those permissions would need a different statement.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListAllMyBuckets",
                "s3:GetBucketLocation"
            ],
            "Resource": "*"
        },
        ...
    ]
}
```

You can use IAM policy variables. Below is an example of getting the username within a policy evaluation.

```json
{
    "Effect": "Allow",
    "Action": "s3:*",
    "Resource": [
        "arn:aws:s3:::xyz-user-content/home/${aws:username}",
        "arn:aws:s3:::xyz-user-content/home/${aws:username}/*"
    ]
    
}
```

You can find more details on IAM variables [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_variables.html)

## Permission boundaries

Permission boundaries don't grant any policies, they limit what permissions an entity can receive. This is very handy when you need to create a user with certain admin permissions, but don't want to give him all the permissions. Below we go though an example configuration.

The scenario:

* Julie is a `AdministratorAccess`
* She wants Bob to be an `IAM administrator`
* She give bob `iam:*` to manage identities
* With this permission nothing stops Bob from changing his own permissions (this is why we need boundaries)
* Even if we block Bob from modifying his account he still can just create another Admin account and use that.

Step 1: Create the user boundary IAM policy. The example policy below limits the users created by bob to a certain set of services and lets them manage their own user/password information.

```json
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Sid": "ServicesLimitViaBoundaries",
          "Effect": "Allow",
          "Action": [
              "s3:*",
              "cloudwatch:*",
              "ec2:*"
          ],
          "Resource": "*"
      },
      {
          "Sid": "AllowIAMConsoleForCredentials",
          "Effect": "Allow",
          "Action": [
              "iam:ListUsers","iam:GetAccountPasswordPolicy"
          ],
          "Resource": "*"
      },
      {
          "Sid": "AllowManageOwnPasswordAndAccessKeys",
          "Effect": "Allow",
          "Action": [
              "iam:*AccessKey*",
              "iam:ChangePassword",
              "iam:GetUser",
              "iam:*ServiceSpecificCredential*",
              "iam:*SigningCertificate*"
          ],
          "Resource": ["arn:aws:iam::*:user/${aws:username}"]
      }
  ]
}
```

Step2: We create the admin boundary that we will be applying to Bob. This is also an IAM policy. This makes it so that Bob can only create users with the user boundary policy applied. We give Bob full admin permissions but attach an admin boundary. The policy below would be attached via permissions to the Bob user.

```json
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Sid": "CreateOrChangeOnlyWithBoundary",
          "Effect": "Allow",
          "Action": [
              "iam:CreateUser",
              "iam:DeleteUserPolicy",
              "iam:AttachUserPolicy",
              "iam:DetachUserPolicy",
              "iam:PutUserPermissionsBoundary",
              "iam:PutUserPolicy"
          ],
          "Resource": "*",
          "Condition": {"StringEquals": 
              {"iam:PermissionsBoundary": "arn:aws:iam::MANAGEMENTACCOUNTNUMBER:policy/a4luserboundary"}}
      },
      {
          "Sid": "CloudWatchAndOtherIAMTasks",
          "Effect": "Allow",
          "Action": [
              "cloudwatch:*",
              "iam:GetUser",
              "iam:ListUsers",
              "iam:DeleteUser",
              "iam:UpdateUser",
              "iam:CreateAccessKey",
              "iam:CreateLoginProfile",
              "iam:GetAccountPasswordPolicy",
              "iam:GetLoginProfile",
              "iam:ListGroups",
              "iam:ListGroupsForUser",
              "iam:CreateGroup",
              "iam:GetGroup",
              "iam:DeleteGroup",
              "iam:UpdateGroup",
              "iam:CreatePolicy",
              "iam:DeletePolicy",
              "iam:DeletePolicyVersion",
              "iam:GetPolicy",
              "iam:GetPolicyVersion",
              "iam:GetUserPolicy",
              "iam:GetRolePolicy",
              "iam:ListPolicies",
              "iam:ListPolicyVersions",
              "iam:ListEntitiesForPolicy",
              "iam:ListUserPolicies",
              "iam:ListAttachedUserPolicies",
              "iam:ListRolePolicies",
              "iam:ListAttachedRolePolicies",
              "iam:SetDefaultPolicyVersion",
              "iam:SimulatePrincipalPolicy",
              "iam:SimulateCustomPolicy" 
          ],
          "NotResource": "arn:aws:iam::MANAGEMENTACCOUNTNUMBER:user/bob"
      },
      {
          "Sid": "NoBoundaryPolicyEdit",
          "Effect": "Deny",
          "Action": [
              "iam:CreatePolicyVersion",
              "iam:DeletePolicy",
              "iam:DeletePolicyVersion",
              "iam:SetDefaultPolicyVersion"
          ],
          "Resource": [
              "arn:aws:iam::MANAGEMENTACCOUNTNUMBER:policy/a4luserboundary",
              "arn:aws:iam::MANAGEMENTACCOUNTNUMBER:policy/a4ladminboundary"
          ]
      },
      {
          "Sid": "NoBoundaryUserDelete",
          "Effect": "Deny",
          "Action": "iam:DeleteUserPermissionsBoundary",
          "Resource": "*"
      }
  ]
}
```

Now when Bob creates a user he must attache the boundary in Step1 or he will be blocked from creating the user.

## Policy Evaluation Logic

### Single Account Flow

Below is the flow which AWS goes through any time its evaluating permissions in a single account.

* Explicit Deny - If there is an explicit deny, access is denied and processing stops.
* Service Control Policies (SCPs) - Check if there are any SCPs on the identity account. The only SCP that is evaluated is the one that is in the account that contains the identity. If the SCP has a deny the processing stops otherwise we move on to next step.
* Resource Policies - If the resource policy has an allow the action will be allowed and processing stops here. If it does not have an allow processing continues to permission boundaries.
* Permission Boundaries - If there is an applicable permission boundry we check if it has a deny. If it does we deny and stop if it does not we move on to Session Policies.
* Session Policies - If we get to this step that means that we are using an IAM role. If session policy does not allow the action you get a deny. If there is no session policy or if session policy allows the action we move on to Identify Policies.
* Identity Policies - If allowed; allow the action otherwise deny.

### Multi Account Flow

You need an allow from both accounts. So account B needs to allow the access from account A and account A has to allow the access from account B. The flow of checks is same as single account flow otherwise.

## S3 Cross Account Access

### With ACLs

ACLs are kind of a legacy thing so you may not need them, but its important to know if you give another account access to a bucket ant hey upload something that account will own the object and the bucket owner account won't have access to the uploaded object even though it owns the bucket.

### With Bucket Policy

When you use Bucket Policy you have an option to set object ownership to either object writer or bucket owner.

### With Role

Because the same account owns the role and the bucket you don't run into the ownership of object issue that you get with ACLs and Bucket Policies. This is the preferred method.

## Resource Access Manager (RAM)

RAM allows you to share resources between AWS accounts without VPC peering. Some things can be shared with any AWS account and only with accounts in the same organization. Products need to support RAM. A shared resources can be accessed natively as though it belongs to the account.

One thing to note when working with RAM is the `us-east-1a` in one AWS account may be a different physical data center than `us-east-1a` in a different account (AWS does this so that users don't swamp the 1a datac enter). Due to this inconsistency you need to use availability zone ids such as `use-az1`. Ids will be consistent across accounts.

VPCs can be created which provide shared infrastructure services to other AWS accounts. For example one account in an org can create a subnet and another account can create resources such as EC2 instances into those subnets. So one account owns and controls the VPS while the other account owns and controls the compute.

## AWS Service Quotas

These are just the AWS applied limits so you don't use up a bunch of services. Each service has specific limits you need to operate withing. Just check the docs or AWS console "service quotas" for the limits. You can create a Quota request template that can be applied to your organization accounts so that you are not manually adjusting quotas per account. You can also create a cloudwatch alarm that will trigger when you are using a certain percentage of the quota for the service. There was a legacy way of tracking this via support but now you should be using the Service Quotas console.

