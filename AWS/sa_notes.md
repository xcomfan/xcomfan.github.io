---
layout: page
title: "AWS SA Notes Monolith"
permalink: /aws/sa_notes
---

[comment]: <> (TODO: Once done with course videos do a review and organization of this so it makes sense to you and you can dive deep on some of the ambiguous concepts)

## AWS Public vs Private Services

Public Service in AWS is one that is accessed over public internet. Private services run within a VPC and can only be accesses from the VPC. You can set up access from a VPC to public services so that access will not go over public internet but will stay in the AWS network. S3 is one such global service.

## AWS Global Infrastructure

Check out [AWS Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) site for visualization of AWS global infrastructure

An Availability Zone is not necessarily one data center. It could be numerous data centers. The Availability Zone designation means that it is isolated from other Availability Zones within a Region.

**Globally Resilient** service means that a region can fail and the service will continue to operate.

**Region Resilient** services operate as separate services within each region and generally replicated data to multiple availability zones within that region.

**AZ Resilient** services run from a single AZ. If that AZ fails the service will fail.

## Intro And Course Details

[GitHub repo](https://github.com/acantril/aws-sa-pro) for the course should have some good examples, and resources for the course.

## Virtual Private Cloud (VPC)

A VPC is a virtual network inside AWS.

A VPC is within 1 account and 1 region.

By default a VPC is private and isolated (you decide if you want to remove isolation or privacy)

There are two types of VPC the default VPC and Custom VPCs. You can only have one default VPC per region but many custom VPCs.

You will typically work with custom VPCs for implementations as they allow for more control.

Default VPCs have all the networking configuration configured by AWS on your behalf and are less flexible.

Every VPC is allocated a range of VPC address known as a VPC CIDR (for example `172.31.0.0/16`) which defines the start and end range of IP addresses that can be used in a VPC.

Custom VPCs can have multiple CIDR ranges, but default VPC only gets one and it is always the same. Its `172.31.0.0/16`.

To make a VPC function across Availability Zones you divide the VPC into subnets. Default VPC will have one subnet in each availability zone.

Each subnet uses a part of the CIDR range and they cannot overlap. For example 172.31.0.0/20 for us-east0-2a, 172.31.16.0/20 for us-east0-2b and 172.31.32.0/20 for us-east0-2c

A default VPC can be deleted an recreated so you can force a 0 VPC situation.

Default VPC carves up the /16 CIDR ranges into /20 ranges for each AZ.

Default VPC comes with an Internet Gateway (IGW), Security Group (SG) and a Network ACL (NACL)

By default anything placed into a default VPC will get a public IPv4 address.

## S3

On object in S3 can be from 0 bytes to 5TB in size.

Buckets are created in a specific AWS region. This means that your data has a primary home region and unless you configure it to do so your data will not leave that region. This also means that in case of disaster teh blast radius is that region.

A bucket name needs to be globally unique (unique across all Regions and AZs)

A bucket is not a file system. Its flat so all objects in a bucket are at the same level. If you use a name such as `/images/my_picture.jpg` The UI will present that as a directory. Folders in S3 are often called prefixes.

Bucket names must be between 3 and 63 characters, start with a lowercase letter or a number and cannot be formatted like IP addresses (`1.1.1.1`).

There is a soft limit of 100 buckets and a hard limit of 1000 buckets per account.

You have an unlimited number of objects in a bucket.

Key for an object is the name and value is the data.

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

## Advanced VPC Routing

Routing table can be associated with an IGW or VGW.

IPv4 and IPv6 are handled separately within a Routing Table.

Routes send traffic based on destinations to a target.

Route table has a limit of 50 static routes and 100 dynamic (propagated) routes.

With two conflicting routes in an RT the longest prefix wins so `/32` will win over a `/24` or `/16` or `/0` (the higher the slash number the more specific a route is).

Static routes have higher priority than dynamic routes as static routes are likely to be ones you manually input.

Any routes learned have the below priority order (only applies if conflicting route)

* Routes learned via Direct Connect (highest)
* Routes learned via Static VPN
* Routes learned via BGP based VPN
* AS_PATH (This is the BGP path between two autonomous systems) (lowest)

Communications between VPCs with CIDR overlaps is really hard and has many limitations. Avoid CIDR overlaps.

[comment]: <> (TODO: Review the Advanced VPC routing content and update notes. Not sure why the flow of the videos jumped to advanced VPC stuff before covering setting up a regular VPC.)

## AWS PrivateLink

PrivateLink lets you connect securely to services hosted in other AWS accounts. Services are presented into your account as private IP addresses and elastic network interfaces. Providing a service this way is one way of using a service without your data going over the public internet.

If setting up a PrivateLink service for it to be highly available you need to have multiple endpoints.

PrivateLink support IPv4 and TCP only. IPV6 is not supported yet.

Private DNS is supported (verified domains) so users of the service can make an DNS entry and override in their VPC the name used for that service.

Direct Connect, Site to Site VPN and VPC Peering are supported.

## VPC Endpoints (Gateway Endpoints)

At time of writing they provide access to S3 and DynamoDB.

Allows you to access these services without setting up internet gateway or public IPs for your resources and to not traverse the public internet.

Prefix list is added to route table with the Gateway Endpoint as a target. Prefix list is just like any entry in a routing table, but the list of IPs is managed by AWS.

The provided endpoint is HA across all AZs in a region by default.

An endpoint policy is used to control what it can access. For example you can allow for connecting to only a particular set of s3 buckets.

Gateway Endpoints can only be used to access services in the same region. You cannot access cross region services.

You can configure S3 buckets to only accept connections from a specific gateway endpoint. This helps with preventing leaky buckets.

These are a logical gateway object and thus can only be accessed from the VPC they exist in.

## Interface Endpoints

Also supports just S3 and DynamoDB like VPC Endpoints.

Not HA by default. For HA add one endpoint, to one subnet per AZ used in the VPC.

You can use Security Groups to control access to the endpoint which you can't do with VPC endpoints.

Endpoints policies restrict what can be done with the endpoint.

Only support TCP and IPv4.

Behind the scenes use PrivateLink

Provides service endpoints in DNS, but can be overridden with your own DNS entry.

[comment]: <> (TODO: Go over this section again you kind of watched it in a hurry)

## Route53

Route 53 provides two services:

1. Register Domains
2. Host Zone files on managed nameservers

S3 is a Global service and thus no region picking is needed. Its also globally resilient.

For registering domains S3 has relationships with all the major domain registries.

When you register a domain with Route 53:

1. Route 53 checks with the registry if the domain is available.
2. If the domain is available Route53 creates a zone file (a database as a flat file with all the DNS information for a domain). In addition to creating a zone file Route53 creates a nameservice for the zone. This domain having a zone file is referred to as **hosted zone** in AWS terminology. Route53 places the zone file onto the managed nameservers and communicates with the registry to add these managed nameserver records as name server (NS) records with the registry.

Hosted zone can be public (find the DNS record on internet) or private (linked and usable within a VPC not publicly accessible).

## Domain Registration with Route53

Process is just standard domain registration from at least via UI.

Make sure you check spam folder for any verification steps needed.

Each domain needs a hosted zone which there will be a small charge for.

The nameservers of a domain should be the same as the ones in the hosted zone for the domain. If you ever delete the hosted zone for some reason and recreate it you will get 4 new nameserver hosts for the hosted zone. You will need to update your registered domain with those servers.

## DNS Record Types

* NS - Name Server records specify what the name server is for a domain
* A and AAAA Records - Map host-names to IP addresses. A for IPv4 and AAAA for IPv6. Generally you create both for the same name and the OS or client can figure out which one they want to use.
* CNAME - Canonical name is an alias or shortcut. You can have multiple CNAME records point to the same A record so that you can use a host for multiple purposes.
* MX - Used for email. MX record has two parts a priority and a value. Lower values are higher priority. If priority values are the same than any of them can be selected. MX record can pointe to inside the zone or if you use a fqnd (. at end) it points to outside the zone.
* TXT Record - Allows you to add arbitrary text to a domain. One common use for txt record is to prove domain ownership. Also can be used to fight spam to provide information about authorized entities.

All DNS records have at TTL (Time to Live time)

It may be a good idea to lower the TTL value some time before you are going to be making changes to reduce chances of caching issues.

## IAM Identity Policies

An Identity is an IAM user, group or role.

An IAM policy is a set of statements that allow or deny access to AWS resources.

We will use the below IAM policy document as an example.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "FullAccess",
            "Effect": "Allow",
            "Action": ["s3:*"],
            "Restore": ["*"]
        },
        {
            "Sid": "DenyCatBucket",
            "Action": ["s3:*"],
            "Effect": "Deny",
            "Resource": ["arn:aws:s3:::catgifs", "arn:aws:s3:::catgifs/*"]
        }
    ]
}
```

`Sid` is an optional field that lets you identity a statement and what it does.

A statement only applies if the action an identity is taking on a resource math the action and resource in the statement.

If specifying a specific resource you need to use the ARN format.

If you have an explicit allow and an explicit deny; the deny will always win. By default everything is denied so you need to explicitly allow and not have it bee denied in any other policy to provide the access.

There are two types of policies **inline** and **managed**. An inline policy is one that you apply directly to an account. A managed policy is one that you create the policy and then apply that policy to users. The difference is with inline you would have a copy per user and managed you would have a single copy associated to multiple users. A managed policy is its own object.

Inline policies are typically used for exceptions to the typical access provided by a managed policy.

There are AWS managed policies which are created by AWS and your own which you create.

## IAM Users and ARNs

IAM users are an identity used for anything requiring long-term AWS access. For example humans, applications or service accounts.

IAM starts with a principal (people, computers, services or group of any of these). First a principal has to identify themselves to IAM. You authenticate with Access Keys or a username and password which are both long term credentials. Once logged in the principle is now known as an "authenticated identity".

ARNs are Amazon Resource Name and they uniquely identify a resource in AWS. Let you identify a single resource or a group of resources using wildcards.

General format is:

```text
arn:partition:service:region:account-id:resource-id
arn:partition:service:region:account-id:resource-type/resource-id
arn:partition:service:region:account-id:resource-type:resource-id
```

`arn:aws:s3:::catgifs` references a bucket while `arn:aws:s3:::catgifs/*` references all items in a bucket. These two ARNs DO NOT overlap. Certain actions are bucket level action so you need to use the first while some are objects based actions so you need to use the second. If you want to provide access for a bucket and objects in the bucket you would need to use both.

When you see `:::` in an ARN it means those fields are not needed. For example in the ARNs above for S3 because S3 is globally unique just the bucket name is enough to identify the resource. The `::` syntax is for when something does not need to be specified.

`partition` is usually just `aws` but for some scenarios such as if you are in China region you would have `aws-cn` for partition.

You can only have up to 5000 users per account.

IAM user can be a member of 10 IAM groups.

These limits have system design impacts. IAM Roles or identity federation can be used to work around these limits.

## IAM Groups

IAM Groups are simply containers for IAM Users. You cannot log into a group and groups have no credentials of their own. Groups can however have policies attached to them both inline and managed. A user that is in a group will get teh policies attached to that group along with any policies attached to the user themselves. This makes Groups an effective admin/management tool. To re-iterate Groups are not a true identity. They can't be referenced as a principal in a policy.

## IAM Roles

A roles are also identities within IAM. They are intended to be used by multiple users in same AWS account or users and services. A role does not represent "you" or a user it represents a level of access.

IAM roles have two types of policies that can be attached to them. Trust Policies which specify what identities can assume that role and Permission Policies which specify what AWS resources the role has access to.

### Service Linked Roles

Service Linked Roles are IAM roles linked to a specific service with permissions predefined by that service. You cannot delete this role until it is no longer required. When creating a policy that allows for the creation of service linked roles you need to be exact in specifying the service. [This link](https://docs.aws.amazon.com/IAM/latest/UserGuide/using-service-linked-roles.html) has details on which services have which service name string.

## AWS Control Tower

AWS Control Tower offers a straightforward way to set up and govern an AWS multi-account environment, following prescriptive best practices. AWS Control Tower orchestrates the capabilities of several other AWS services, including AWS Organizations, AWS Service Catalog, and AWS IAM Identity Center (successor to AWS Single Sign-On), to build a landing zone in less than an hour. Resources are set up and managed on your behalf.

AWS Control Tower orchestration extends the capabilities of AWS Organizations. To help keep your organizations and accounts from drift, which is divergence from best practices, AWS Control Tower applies preventive and detective controls (guardrails). For example, you can use guardrails to help ensure that security logs and necessary cross-account access permissions are created, and not altered.

[comment]: <> (TODO: These notes are from the overview lesson this topic warrants a deeper dive.)

## S3 Security

S3 is private by default. Only user that has access by default is the account root user.

First way to grant access to an S3 bucket is with a **bucket policy**. A bucket policy is a **resource policy** which is just like an identify policy but attached to a resource instead of to an identity. With an identity policy you control what that identity can access. With a resource policy you are controlling who can access that resources. Identity policies can only be attached to identities in your own account thus identity policies can only control security within your account. Resource policies can allow/deny same or different accounts. Resource policies are also able to allow/deny anonymous principals.

Below is an example of a resource policy. Notice that it has a `Principal` component. You would not see this in an identity policy because the identify itself is the principal.

```json
{
    "Version": "2012-10-17",
    "Statement":[
        {
            "Sid": "PublicRead",
            "Effect": "Allow",
            "Principal": "*",
            "Action": ["s3:GetObject"],
            "Resource":["arn:aws:s3:::secretcatproject/*"]
        }
    ]
}
```

Bucket polices should be your default though when it comes to granting anonymous access to an S3 bucket.

Bucket polices can control who can access objects and even block specific IP addresses. Example below:

```json
{
    "Version": "2012-10-17",
    "Id": "BlockUnLeet",
    "Statement": [
        {
            "Sid": "IPAllow",
            "Effect": "Deny",
            "Principal": "*",
            "Action": "s3:*",
            "Resource": "arn:aws:s3:::secretcatproject/*",
            "Condition": {
                "NotIpAddress": {"aws:SourceIp": "1.3.3.7/32"}
            }
        }
    ]
}
```

More bucket policy examples can be found [here](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html)

Another approach to provide access to a bucket is less used now but its ACLs (Access Control Lists). ACLs are very limited and AWS recommends using bucket policies. They essentially allow for read, write, read and write of the ACL anf full control. Nothing outside of that.

## S3 Static Hosting

S3 can generate a URL for you which will be based on region and bucket name, or you can use a custom domain but then the bucket name has to match the custom domain. 

This feature is useful not just for a whole static site, but to have static content in S3 that a dynamic app can reference as part of the page the client sees.

Its also good to use as status or other type of out of bound management pages.

Even if you select host website in S3 you still need to use a bucket policy to allow unauthenticated reads of objects in the s3 bucket.

example of such as policy:

```json
{
    "Version": "2012-10-17",
    "Id": "BlockUnLeet",
    "Statement": [
        {
            "Sid": "PublicRead",
            "Effect": "Allow",
            "Principal": "*",
            "Action": ["s3:GetObject"],
            "Resource": "arn:aws:s3:::example_bucket/*",
        }
    ]
}
```

When making the Route53 entry for the static hosted website, use simple routing record type would be A but for the Value/Route traffic to option you would pickk Alias to S3 website endpoint and enter the bucket endpoint URL as well as appropriate region.

## S3 Object Versioning and MFA Delete

Once you enable bucket versioning on a bucket you cannot disable it. You can however suspend the versioning.

Each object in a bucket has an `id` metadata which if versioning is disabled is just set to `null`. If versioning is enabled the `id` is used to track versions. If you request an object you are given the latest one by default, but if you request an object and provide `id` you can get to the prior version.

If you delete on object with versioning enabled nothing actually gets deleted. A delete marker is used to indicate that an object has been deleted and it is then effectively hidden but still there. Delete marker is a version of that object that is 0 sized. You can "un-delete" an object by removing the delete marker. If you delete an object and specify the `id` that specific version of the object will be permanently deleted.

All versions of the object take up space (and will be billed for)

Within versioning configuration options you can enable MFA delete. If enabled MFA will be required to change bucket versioning state and to delete versions of objects. You will need to pass the MFA token value with any API Calls.

## S3 Performance Optimization

By default data is uploaded to S3 as a blob using a single stream. If a stream fails the whole upload needs to start over. This has issues as for larger file chances of failing and having to start over are high. To address this there is a multi part upload option. 

Minimum size to use multi part upload is 100MB. Most AWS tooling will switch to multi part upload if the option is available. Each part of upload can fail and be restarted. Also since you are using multiple streams you end up uploading faster and more reliably.

S3 Transfer Acceleration is a feature you can enable where instead of uploads going to a specific region they will go to an edge location and be moved to the region via the AWS network. This way your upload goes to the closest location to you and then goes over the fast AWS network to get to the actual region of the bucket.

When you enable transfer acceleration you are provided with a new endpoint that you should be using to upload data to S3. AWS has a S3 Transfer Acceleration comparison tool so that you can see the performance benefits.

## KMS (Key Management Service)

KMS is a regional and public service (occupies AWS public zone).

Like any other AWS service you need permission to access it.

Supports both symmetric and asymmetric keys.

Supports cryptographic operations (encrype, decretyp, etc.)

Keys never leave the product

KMS primarily uses KMS keys. This is a logical containers for the actual key and this containers has an ID, date, policy, description and state.

Each key is backed by physical key material which is either generated by KMS or imported into KMS.

A high level flow of KMS use would be:

1. CreateKey to create a KMS key. KMS will store that key on disk in encrypted format (nothing in KMS is ever no encrypted on disk possibly in memory).
2. Encypt call where you provide what you want to be encrypted along with the key to use and KMS does the encryption for you.
3. Decrypte call and give the data to decrypt. No need to tell KMS which key to use that in encoded into the data cypher text.

Encrypt, Decrypt and generating keys are all separate permissions.

KMS can only operate on data that is 4kb in size. The way KMS works around this limitation with with DEKs (Data Encryption Keys). KMS key is used to generate a DEK which is linked to the KMS key and the DEK is used to encrypt data that is > 4kb in size. KMS DOES NOT store the DEK in any way. It generates it and provides it to you then discards it. KMS does not actually do the encryption and decryption of data. You do using the DEK. This works as follows:

1. When a DEK is generated KMS provides you with two versions of that key. A plaintext version that can be used to encrypt/decrypt immediately and a ciphertext version that is encrypted. The encrypted one can be given back to KMS for it be decrypted. So you would encrypt your content, discard the clear text DEK key and store the encrypted version with your data.

2. You can use one DEK on all of your content or generate a new DEK for each one. The choice is yours.

3. You pass the encrypted DEK key to KMS along with the KMS key used to generate it to decrypt the DEK.

By default KMS keys are stored within a specific region and never leave the region.

KMS does support multi region keys where the keys are replicated to another region.

There are AWS managed and customer manged keys in KMS. AWS managed ones are created automatically behind the scenes for AWS services which leverage KMS such as S3.

There are key policies (a resource policy) which allow you to specify permissions specific to a key itself. This can also be used to give another account access to the key.

Unlike other AWS services KMS does not implicitly trust the account its contained within. You need to use a key policy to provide that trust. Example of that below. Without that on the key policy IAM policies will not work and you will need to add any permission on the key policy itself. Rational for this is that in high security environments you can have the access controlled specifically on a key.

```json
"Sid": "Enable IAM User Permissions",
"Effect": "Allow",
"Principal": {"AWS":"arn:aws:iam::111222333:root"}
"Action": "kms",
"Resource": "*"
```

Permissions for KMS are very granular and you can split who can create keys and who can use them to encrypt and/or decrypt.

Example of using KMS to encrypt and decrypt a plain text file

```bash
echo "find all the doggos, distract them with the yumz" > battleplans.txt

aws kms encrypt \
    --key-id alias/catrobot \
    --plaintext fileb://battleplans.txt \
    --output text \
    --query CiphertextBlob \
    | base64 --decode > not_battleplans.enc 
    
aws kms decrypt \
    --ciphertext-blob fileb://not_battleplans.enc \
    --output text \
    --query Plaintext | base64 --decode > decryptedplans.txt
```

## S3 Bucket Encryption

Buckets are not encrypted, objects are. This means you set/apply the encryption policy when uploading the document. You can set the default choice on the bucket, but it does not prevent someone from uploading with different options.

S3 supports client side and server side encryption.  Client side encrypted is S3 never sees the un encrypted data it arrives to S3 encrypted while server side (SSE) is data is not encrypted (not taking the https tunnel into account) and S3 encrypts it as it writes to disk. 

SSE has been made mandatory so you have to use encryption at rest.

Data in transit to S3 is usually encrypted (there are a few exceptions)

There are 3 types of SSE available to S3 objects.

* SSE-C Server side encryption with customer provided keys. You provide the object to be stored and the key (in plain text but over https) and S3 will do the encryption compute then store a one way hash of the key and discard the key.If you try to get object out you provide the key again and the hash is used to verify that ist the right key then it uses the key you provide to decrypt. Decision here is your requirements. If you trust Amazon you can offload the encryption compute to them and just manage the keys, if you don't trust them you can locally encrypt.

* SSE-S3 Server side encryption with Amazon S3 managed keys (this is the default). Each object in encrypted with a key that is generated for each object. There is another key fully managed by S3 that is used to encrypt the key which is used to encrypt the object. Essentially this is same as SSE-C but Amazon managed the key. If you are in environment where you need to control keys or key rotation orr be able to separate the roles of who administers S3 and who is able to view the data than this process does not work.

* SSE-KMS Service side encryption with KMS Keys stored in AWS KMS. KMS key is created by you, managed by you and can have isolated permissions. This approach is similar to SSE-S3 but you control the key and have role separation.

Client side and server side encryption can be combined should you want that.

## S3 Bucket Keys

If you are using SSE-KMS each upload to S3 needs to make a call to KMS. KMS has cost and throttling so that may be an issue. With bucket keys instead of KMS being called to generate Data Enryption Key (DEK) for each object it generates a time limited key for the bucket which is used for that period of time. This is not retroactive takes affect after its enabled.

After being enabled CloudTrail will show bucket instead of object. 

This will also work with replication. The object encryption will be maintained.

If replicating plaintext to a bucket using bucket keys the object in encrypted at the destination side (ETAG changes between source and destination) more details [here](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucket-key.html)

## S3 Object Storage Classes

With the default S3 Standard objects are replicated to 3 AZs in a region. You are billed per GB of data stored, per GB of transfer out and price per 1000 requests. There are not minimum object amounts or storage sizes. Standard has a first byte access time of milliseconds. Standard should be used for frequently accessed data that is non replaceable.

S3 Standard IA (Infrequent Access) is just like standard in features but there is a retrieval fee in addition to the transfer fee. The storage cost is cheaper however. There is also a minimum of 30 days storage charge (no matter how long object is stored you get billed for 30 days). You also get charged for 128kb per object (or more if object is bigger) even if object is smaller than 128kb.  S3 Standard IA should be used for long lived data which is irreplaceable but where access is infrequent.

S3 One Zone-IA Similar to IA but not replicated to 3 regions in an AZ. Data is still replicated withing the availability zone, but not across availability zones. Should be used for long lived but non critical and replaceable data that is infrequently accessed.

S3 Glacier Instant - Higher extraction fees and minimum storage of 90 days but even cheaper per GB cost. Good for long lived data that is accessed once a quarter.

S3 Glacier Flexible - Think of these as cold objects. They can't be made public and any access requires a retrieval process. You have 3 options for retrieval, expedited takes 1 to 5 minutes but is expensive, standard which takes 3 to 5 hours and bulk which takes 5 to 12 hours. Has a 40kb minimum billable size and 90 Day min Duration.

S3 Glacier Deep Archive - The cheapest to store. 40kb min billable size and 180 day minimum billable duration. Objects cannot be made public. Take much longer to retrieve 12 hours standard and 48 hours for bulk. Good for archival data.

S3 Intelligent-Tiering - Contains 5 tiers within it. Frequent access, Infrequent Access, Archive Instant Access, Archive Access and Deep Archive. Usage of Archive and Deep Archive is optional. The other tiers S3 will monitor and move infrequently accesses objects to. Good for long lived data where usage patterns are unknown.

## S3 Lifecycle Configuration

Lets you transition objects between storage classes based on rules or delete objects after a certain time. You can't configure based on how frequently accessed an object is for that use intelligent tiering. In Lifecycle configuration an object needs to be in standard for 30 days before it can be moved to a cheaper class. You also can't move to Standard IA or One Zone IA and Glacier classes in a single rule. Need to combine two rules for that.

## S3 Replication

There are two types of replications Cross Region Replication (CRR) and Same Region Replication (SRR). SRR is when you want to replicate between two accounts in same region.

Configuration is done on source bucket. A role needs to be configured via Trust Policy so that you can assume that role and write to the destination bucket. If replicating to a different account you need to have a bucket policy which trusts the account that is sending the data.

You can replicate all objects or filter based on prefix or tags.

You can also select storage class in the destination bucket. 

You can also configure ownership in the destination account.

RTC (Replication Time Control) can be used to add a 15 minute SLA to the replication process. Without this its a best effort thing. You can also use monitoring to see what is queued for replication.

Replication is not retroactive. If a bucket has objects when you enable replication, those objects will not be replicated. Only the new objects will be replicated.

A bucket cannot be enabled for replication without versioning being enabled.

Batch replication can be used to replicate existing objects.

Replication is not bi-directional. It goes one way from source to destination. An option for bi directional replication has been recently added but that is a separate feature you need to enable.

With some extra configuration SSE-S3, SSE-KMS and SSE-C encryption is supported for replication.  SSE-C is a recent addition.

No system events, Glacier or Glacier Deep Archive can be replicated. (Glacier should be though of as a separate storage product)

By default deletes are not replicated but this can be enabled.

Uses for replication are:

* Log aggregation
* Prod and test account/environment sync
* Resilience with strict sovereignty requirements
* Global Resilience Improvements
* Latency Reduction

## S3 Pre Signed URLs

Pre signed URLs allow you to give another person access to an object in an S3 bucket using your credentials in a safe and secure way.

Without using pre signed URLs the only way to give an anonymous/un-authenticated user access to an object is s3 is:

* Give the anonymous user an AWS Identify
* Give the anonymous user AWS Credentials
* Make the bucket public

None of these options are ideal.

When you ask s3 to create a pre signed URL the pre signed url has embedded in it credentials with an expiration time. It will also have the details of which object and bucket the pre signed URL is for.

Pre signed URLs can be used for GET and PUT (download and upload operations)

When using the URL, the permissions match the identity which generated it at the moment the link the being used. So if you get an access denied that means that either the creator of the link did not have access initially or it does not have access at the time you are using the link.

DO NOT generate signed URLs with an IAM role. the URL will stop working when the temporary credentials of the role expire.

One way to generate a pre signed link is via aws cli `aws s3 presign s3://some_prefix/file.txt --expires-in 180`.

## S3 Select and Glacier Select

The problem this feature solves is if you have a large 5TB object and you download that 5TB object you are going to be billed for the 5TB of transfer.

S3/Glacier Select lets you use SQL-LIke statements to extract a part of the object. 

No demo for this in the course just making you aware it exists.

## S3 Events

Allows you to configure event notifications on a bucket.

Can be delivered to SNS, SQS and Lambda Functions

Events supporter are of create category (put, post, copy, completeMultiPartUpload), delete (*, Delete, DeleteMarkerCreated), restore (Post(initiated), completed) and replication (OperationMissedThreshold, OperationReplicatedAfterThreshold, OperationNotTracked, OperationFailedReplication)

Notifications can be send to Lambda, SQS Queue and SNS Topics. You will need to add resource policies to those services so that they are allowed to interact with S3.

## S3 Access Logs

Logs for a source bucket will be sent to a target bucket. Works using an S3 Log Delivery Group. You need to give the Log Delivery Group write access to the target bucket. Logs are delivered as files and are newline delimited.

If you use this you need to manage the storage lifecycle of the log files in the target bucket.

## S3 Object Lock

Can be enabled on a new bucket or existing but with help of AWS support.

Implements WORM (Write Once Read Many) - No delete no overwrite

Requires versioning and individual versions of the object are locked.

A lock has a Retention period and a Legal Hold. These can be defined at the object or at the bucket level.

If you use compliance mode that means that an object cannot be adjusted, deleted or overwritten during the retention period. It also means the retention period itself cannot be adjusted or removed. This even included the account root user. This is the most strict form of object lock.

Governance mode allows for the lock setting to be adjusted.

Governance mode is good for preventing accidental deletions or even for testing policy before turning on compliance mode.

There is also the option of S3 Legal Hold. This is an on or off toggle. The object will not be deletable or changeable until the legal hold is removed. The `s3:PutObjectLegalHold` permission is required to add or remove the hold. Used to prevent accidental deletions or to flag an object as critical for a project.

These options can be used in conjunction with each other.

## S3 Access Points

If you have an S3 bucket that is very widely used your bucket policy can become very complicated (think many different teams needing different access permissions). S3 Access points addresses this by allowing you to create multiple access points to the same bucket each with different policies and different network access controls. Each endpoint has its own address.

You can create these both via UI or via CLI `aws s3control create-access-point --name secretcats --account-id 123456789012 --bucket catpics`

You can restrict an access point to have a specific VPC as the request origin.

Any permissions defined on an access point also need to be defined on the bucket policy. You can have wide open access on a bucket policy to a specific access point and then use the access point permissions to fine tune the access. This is a commonly used pattern.

## VPC Sizing and Structure

Before creating a custom VPC take these considerations into account.

* What size should the VPC be?
* Are there any networks we can't use?
* Other VPC's, Cloud, On-premises, Partners & Vendors
* Try to predict the future
* VPC Structure - Tiers & Resiliency (Availability Zones)

In AWS smallest VPC you can make is `/28` (16 IP addresses) and maximum size is a `/16` (65536 IP addresses).

Its good to avoid common ranges so avoid using  `10.0.X.X` and `10.1.X.X` because `10.0` is the default and many people pick the next one as the non default. Generally good to avoid 10.0.X.X to 10.10.X.X so good starting point is 10.16.

Aim to have at least 2 networks per region being used per account and add some buffer.

For example if you will be using 5 Regions (3 in US and Europe and Australia), and 4 accounts that would make it `5 * 2 * 4` and bring us to 40 IP ranges.

10.128 to 10.255 is a Google default so avoid that.

VPC Sizing:

| VPC Size | Netmask | Subnet Size | Hosts/Subnet* | Subnet/VPC | Total IPs* |
| -------- | ------- | ----------- | ------------- | ---------- | ---------- |
| Micro| /24 | /27 | 27 | 8 | 216 |
| Small | /21 | /24 | 251 | 8 | 2008 |
| Medium | /19 | /22 | 1019 | 8 | 8152 |
| Large | /18 | /21 | 2043 | 8 | 16344 |
| Extra Large | /16 | /20 | 4091 | 16 | 65456 |

You want to plan to use 4 availability zones in your VPC with one being a spare in case you have growth.

Within each availability zone you want to have "tiers" a good default for number of tiers is 4 (Web tier, app tier, db tier and a spare). This can vary depending on what you are doing, but its a good start.

Each tier would have its own subnet in each AZ.

Some useful links:

https://github.com/acantril/aws-sa-associate-saac02/tree/master/07-VPC-Basics/01_vpc_sizing_and_structure

https://cloud.google.com/vpc/docs/vpc

https://aws.amazon.com/answers/networking/aws-single-vpc-design/

## Custom VPCs Theory and Demo

If you choose Dedicated Tenancy at the VPC level then you will be locked in to using dedicated tenancy in that VPC. If you pick default options than you can choose at resource creation time what tenancy you need.

In AWS diagrams blue means private subnets and green means public by convention.

A subnet can never be in more than one availability zone.

Some IPs in every VPC subnet are reserved 5 in total:

If for example we are talking about a VPC with subnet 10.16.16.0/20 (10.16.16.0 to 10.16.31.255) the following addresses are reserved/not usable.

* Network address 10.16.16.0
* Network + 1 10.16.16.1 - VPC Router
* Network + 2 10.16.15.2 - Reserved for DNS
* Network + 3 10.16.16.3 - Reserved for future use
* Last IP in Subnet 10.16.31.255 - The broadcast address

DHCP Options set controls DHCP behavior for the subnet. You can create your own option sets, but you cannot edit them.

## VPC Routing and Internet Gateway

VPC Router is a highly available device that exists in every subnet at the network + 1 IP address. By default it will route traffic between the subnets of a VPC. This routing is controlled by a Routing table and each VPC by default has a "Main" route table which you can override with your own. As you recall a subnet can only have one subnet associated with it, but a route table can be associated with many subnets.

All routing tables have at least one entry with a local target so that VPC router knows where local traffic should be sent.

Internet Gateway (IGW) is a region resilient(mean that one IGW can cover all AZs in a region) gateway attached to a VPC.

VPC can have none or 1 IGW, and IGW can be attached to only 1 VPC at a time.  

IGW Gateways traffic between the VPC and the Internet or AWS Public Zone services (S3, SQS, SNS, etc.)

To use an IGW:

1. Create IGW
2. Attache IGW to VPC
3. Create custom Routing Table
4. Associate Route Table to VPC
5. Default Routes go to IGW
6. Subnet allocate IPv4 and IPv6 automatically

Side note about public IPv4 addresses. The public IP address is never seen by the instance and its OS. The public IP is associated with the private IP in an internet gateway so that the gateway can send the traffic meant for that IP to the correct private address.

## NAT (Network Address Translation) Gateway

A set of processes that remaps source or destination IPs. This is sometimes referred to as IP masquerading (hiding CIDR blocks behind one IP address).

Useful because public IP addresses are running out.

Gives a private CIDR range **outgoing** internet access. You just use the routing table in a private subnet and point the default IPv4 route to the NAT Gateway.

Nat Gateways are resilient to an AZ (HA in that AZ)

NAT Gateways run from a public subnet and utilizes an Elastic IP address.

You need one Gateway and one RT in each AZ that you use.

Scale to 45 Gbps and are charged by time and data volume. You can route to multiple NAT Gateways if you need more bandwidth. Partial hours are billed as full hours.

You can create a NAT instance. If you do this make sure you disable source and destination checks as by default an EC2 instance will drop any data on its network work where the network card is not either the source or destination. If we are using an EC2 instance as a NAT Gateway, we need to disable that feature.

For maximum availability you would have a NAT Gateway in each AZ that you use.

EC2 as NAT instance is a good idea if:

* Cost is a matter
* Test or low volume VPCs
* Can save money in situation where you have a lot of data going over the gateway.
* Provides predictable cost
* Can be used for port forwarding and as bastion hosts.
* Can use Network ACLs and Security Groups. NAT Gateway only supports NACL.

NAT is not needed for IPv6. In AWS all IPv6 addresses are publicly routable.

Internet Gateway works with ALL IPv6 IPs directly. If an instance in a private subnet has a default IPv6 route to an Internet Gateway it will become a public instance. If you want outgoing only IPv6 access look at the Egress Only Internet Gateway which will get covered later.