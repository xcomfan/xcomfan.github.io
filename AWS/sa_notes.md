---
layout: page
title: "AWS SA Notes Monolith"
permalink: /aws/sa_notes
---

[comment]: <> (TODO: Once done with course videos do a review and organization of this so it makes sense to you and you can dive deep on some of the ambiguous concepts)

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

Note: Groups are not real identities which can be accessed by ARNs in resource policies.

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

## SAML 2.0 Identity federation

SAML means Security Assertion Markup Language which is an open standard used by many identify providers. This functionality lets you use an identify provided used in your enterprise to **indirectly** access AWS CLI or console. The identify provider must be SAML 2.0 compatible to work with AWS. Web identity provided such as Google, or Facebook will not work here.

At a high level the process is you set up a trust between the identify provided and IAM. Once identity provider verifies a user it will use `STS:AssumeRoleWithSAML` call to get STS credentials for the user which grand the access based on the role assumed. For access to AWS console its the same process, but instead of a trust between your identify provider and IAM you set up a trust between identity provider and a SAML/SSO endpoint on AWS.

## AWS Single Sign-on (SSO)

This should replace SAML 2.0 federation and is not the recommended way. It allows you to manage SSO access to AWS accounts. There is a built in identity service or you can link it to an existing one that you have.

## DHCP in a VPC

You can either use Amazon provided DNS (Route 53 resolver running within VPC) or provide your own DNS host.

Configuration is done via DHCP option sets. Each options set is immutable and can be associated with 0 or more VPCs. Each VPC can have 0 or at most 1 Option Set associated with it. You can change the associated option set, but changes will only occur when DHCP renews occur.

## VPC Router Deep Dive

Virtual Router within a VPC. Is created when a VPC is created and his highly available across all AZx in that region. Its scalable without any involvement from the user. Responsible for routing traffic between subnets. Also routes from external network into VPC and from VPC into external networks. Has in interface in every subnet at the subnet+1 address (default gateway via DHCP Options Set). Controlled using route tables.

Every VPC is created with a Main Route Table (RT). This main route table is the default for every subnet in the VPC. Custom route tables can be created and associated with subnets in the VPC removing the Main RT. Subnets can be associated with one Route Table only either a custom one or the default Main RT.

Its best practice to leave the Main RT blank and set your routes in the custom route tables. That way if custom route table is removed you don't get surprise/unexpected behavior. 

Route tables can contain multiple routes, and the most specific route wins (`/32` is most specific with just one ip address) when deciding which route to use. Route tables can also be associated with Gateways. Target of a route can be `local` which means the local VPC.

## Stateful vs Stateless Firewalls

As a fundamental concept when you establish a connection for example via https on port 443, what happens under the hood is the client is making on request on port 443 and the server is responding to an ephemeral port chosen by the clients OS. Thus a connection is identified by the unique combination of request IP, request port and response IP and response port.

The primary difference between a stateful and stateless firewall is that with a stateless firewall each request is independent. So as in the example above, you need to create a rule for the inbound request and for the outbound response (this is from perspective of the servers it reverses for the client.) You also have to allow the range of ephemeral ports since you don't know which one will be selected. A stateful firewall on the other hand is able to match up a request with a response so if the request is allowed the corresponding response is also allowed without having to create the extra rules.

## Network Access Controls Lists (NACL)

NACLs are associated with subnets. Every subnet has an associated network ACL. The NACL comes into play when data crosses the boundary of the subnet. NACL is not evaluated for traffic within a subnet; just data coming into subnet and leaving subnet. NACLs are stateless so you need to have rules for both outbound and inbound requests. 

Rules of a NACL are processed in order till a match is found. Once a match is found processing stops so the first match rule wins if you have two rules with same parameters. There is also a catch all indicated by `*` which effectively says deny if none of the rules match.

Below is an example of a NACL for https.  `0.0.0.0/0` mean from anywhere. There is an inbound rule for port 443 and outbound for the response.

| Rule number | Type | Protocol | Port range | Source | Allow/Deny |
| ----------- | ---- | -------- | ---------- | ------ | ---------- |
| 110 | HTTPS(443) | TCP | 443 | 0.0.0.0/0 | Allow |
| * | All traffic | All | All | 0.0.0.0/0 | Deny |

| Rule number | Type | Protocol | Port range | Source | Allow/Deny |
| ----------- | ---- | -------- | ---------- | ------ | ---------- |
| 120 | Custom TCP | TCP | 1024-65535 | 0.0.0.0/0 | Allow |
| * | All traffic | All | All | 0.0.0.0/0 | Deny |

Default NACL created with a VPC will allow all traffic.

NACLS allow you to explicitly deny access so it may be a good tool to block bad actors. 

There is no logic that NACLs are capable of. They just look at source and destination IPs and ports. They also can only be applied to subnets only, not individual resources.

Custom NACLs you create just deny traffic by default. Just something to be aware of if you are creating one as it differs from the allow all traffic behavior you get with the default NACL created with a subnet.

## Security Groups

Security groups are stateful. Thus if you allow the request the response is automatically allowed.

Security groups do not have an explicit deny. You can only allow or implicitly deny. In other words you cannot block a specific bad actor. This is why NACLs are used in combination with security groups.

Security groups support IP/CIDR and logical resources including other security groups and itself. Itself is useful because you can use it to allow all traffic between resources in the same security group.

Security groups are not attached to instances but actually attached to ENIs. The UI sometimes makes it look like you are attached a SG to an instances but its actually being attached to ENI.

## AWS Local Zones

Local Zones are kind of like AZs but more local. Useful if you have latency sensitive applications. The AZ in the region is a parent to these Local Zones and the subnets from the parent region is extended to the local zones so you can create resources in the local Zones which leverage the same subnets.

## Border Gateway Protocol (BPG) 101

Autonomous Systems (AS) - Routers controlled by one entity ... a network in BGP

ASNs are uniq and allocated by IANA (0-65535) 64512 - 65534 are private

BGP Operates over tcp port 179 and is reliable

BGP is not automatic - peering is manually configured

BGP is a path-vector protocol. It exchanges the best path to a destination between peers. This path is called **ASPATH**

**iBGP** = Internal BGP - Routing within an AS

**eBGP** = External BGP - Routing between ASs

## AWS Global Accelerator

Global Accelerator helps with the issue of your application being hosted in one region but having users access it from another (thing global application).

It starts with 2 anycast IP addresses. Anycast IP addresses are a special type of IP addresses (normal IP addresses are unicast and refer to one network device). Anycast are IPs that are advertised onto the internet but multiple devices can use. Core routers will route the traffic to the device closest to the source.

With Global Accelerator if you use one of these two IP addresses you are routed to an edge location closest to you over public internet. Global accelerator takes the traffic and sends it to the target location, but now its going over the AWS private network not public internet.

Global Accelerator is very similar to Cloud Front. It moves the AWS network closer to your customers. Global accelerator can route the traffic either to one region or to the closest region.

Main difference between Global Accelerator and CloudFront is Global Accelerator is a network product and is used on TCP/UDP. CloudFront only works on HTTP.

## IPSEC VPN Fundamentals

As you know symmetric encryption is fast, but you need to have a way to share the key. Asymmetric is slow, but you don't need to send a key (it can be negotiated during session establishment). For this reason IPSEC has two main phases **IKE Phase 1** the slow part where the key to be used is is established. this is when you authenticate via a password (pre shared key) or certificate, using asymmetric encryption agree on and create a symmetric key and set up the encrypted tunnel. **IKE Phase 2** uses the keys from phase 1 for bulk data transfer.

There are policy based and route based VPNs Route based is simple and based on destination. Policy based checks rules and applies different security configurations depending on what rules apply to the traffic.

## AWS Site to Site VPN

AWS Site to Site VPN is a logical connections between a VPC and on-premises network encrypted using IPSec, running over the public internet.

Can be fully HA if you design and implement it correctly and can be set up in about an hour. It is comprised of ..

* Virtual Private Gateway (VGW) this can be a target on route tables.
* Customer Gateway (CGW) - either a logical configuration of a physical devices in an AWS customer data center.
* VPN Connection between the VGW and CGW.

[comment]: <> (TODO: This is a good candidate to re watch "Site2Site VPN Refresher" and fill out the notes as when I viewed it it was not relevant to what I am working on.)

## Transit Gateway

[comment]: <> (TODO: Come back and write up notes on "Transit Gateway Refresher" and the deep dive lesson)

## 