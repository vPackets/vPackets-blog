+++
title = "Create a VPC - Manual"
description = "Walkthrough of a VPC creation in AWS"
date = "2023-03-12"

weight = 1

tags = [
    "AWS",
    "VPC"
]

categories = [
    "Cloud Networking",
]
+++


In this blogpost we are going to create a [Virtual Private Cloud](https://docs.aws.amazon.com/vpc/) in [Amazon Web Services](https://aws.amazon.com/) that will host all the shared services generally deployed by an IT team.
Since networking is fundamental to allow communication between servers and computers, every cloud service providers use a dedicated network construct. AWS and Google Cloud call it a VPC, while Azure for example calls it a VNET (virtual network).

Network constructs are different between cloud providers. For example, a VPC is global in Google Cloud while it is a regional construct in AWS. This blogpost is not targeted to highlight the networking differences between AWS and GCP (more on that in a dedicated blogpost).

As AWS mentions, "A [region](https://aws.amazon.com/about-aws/global-infrastructure/regions_az/) is a physical location around the world where we cluster data centers. We call each group of logical data centers an Availability Zone. Each AWS Region consists of multiple, isolated, and physically separate AZs within a geographic area. Unlike other cloud providers, who often define a region as a single data center, the multiple AZ design of every AWS Region offers advantages for customers. Each AZ has independent power, cooling, and physical security and is connected via redundant, ultra-low-latency networks. AWS customers focused on high availability can design their applications to run in multiple AZs to achieve even greater fault-tolerance. AWS infrastructure Regions meet the highest levels of security, compliance, and data protection.

What we are trying to achieve is the following:

![AWS VPC](/images/aws/vpc/manual-vpc/Fig1-aws-vpc.png)


That's the diagram of a very general VPC that can created using the AWS Console. We will also cover how to create a VPC using several automation tools like the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) and [Hashicorp Terraform](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/vpc) and here is an [example](https://github.com/terraform-aws-modules/terraform-aws-vpc)

The AWS console offers the choice to create juste a simple VPC without any other additionnal network construct or the entiore VPC with all the objects needed to allow communication between your entities (Route table, Security Group, NAT and Internet gateways). In our example we will be using the options that allows us to create all networking construct at once.

+ CIDR:
  - This VPC has a CIDR or 10.0.10.0/24 and AWS will automatically create 2 publics subnets and 2 private subnets.
  - These public and private subnets will be shared between the availability chosen during the creation process.

+ No IPv6 is required for now.
+ Number of Availability zones: 2 is sufficient
+ Number of public and private subnets: 2 each (even though only 1 public is sufficient, our VPC might need additional private and public subnets later on.)
+ NAT Gateway: None.

The following picture shows how to create your VPC:

![AWS VPC Console](/images/aws/vpc/manual-vpc/Fig2-aws-vpc-console.png)

For now we do not have to take care of any [Elastic IP Address](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html) as we will ask AWS to assign a public IP to our controller when we deploy it.

Finally the AWS Console will confirm that all the objects within the VPC have been created:

![AWS VPC Created](/images/aws/vpc/manual-vpc/Fig3-aws-vpc-created.png)

The next picture will demonstrate the automatically created subnets and route tables for the specifically mentionned availability zones.

![AWS VPC Subnets](/images/aws/vpc/manual-vpc/Fig4-aws-vpc-subnets.png)

If we have a look at the route tables associated with that VPC, we can see that each private subnet has its own routing table. The public subnets share the same public route table:

![AWS VPC Subnets](/images/aws/vpc/manual-vpc/Fig5-aws-vpc-routetable.png)

When you dig into the **"AWS-UE1-VPC-MGMT-rtb-public"** routing table, we can see that it has two routes for now :
+ **10.0.10.0/24**, that is local 
+ **0.0.0.0/0**, the default route that points to an internet gateway.

Now if we deploy an EC2 instance into any public subnet that belongs to the Region/AZ created previously, we should have connectivity to reach the internet. If we want to authorize inbound traffic to that EC2 instance, we need to allow that traffic using the stateful AWS [Security Groups](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html).

Our VPC should be ready to host our [Aviatrix Controller](https://docs.aviatrix.com/HowTos/controller_config.html) to start out multicloud journey.



## Bonus 

If you want to verify using the CLI (some of us are still network engineers right ?) you can configure your computer following this [doc](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html).


(I will reduce the amount of information in the outputs below to focus on what's important. For example, this output should also list the default VPCs but I will not list it)
``` bash

$ aws ec2 describe-vpcs | jq

```
``` json
{
  "Vpcs": [
    {
      "CidrBlock": "10.0.10.0/24",
      "DhcpOptionsId": "dopt-0a943645c8bc81085",
      "State": "available",
      "VpcId": "vpc-08aee6f377637f9ae",
      "OwnerId": "",
      "InstanceTenancy": "default",
      "CidrBlockAssociationSet": [
        {
          "AssociationId": "vpc-cidr-assoc-04f586cab46d8b000",
          "CidrBlock": "10.0.10.0/24",
          "CidrBlockState": {
            "State": "associated"
          }
        }
      ],
      "IsDefault": false,
      "Tags": [
        {
          "Key": "Name",
          "Value": "AWS-UE1-VPC-MGMT"
        }
      ]
    },
  ]
}
```

We can also check for the subnets created.

```
$ aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-08aee6f377637f9ae" | jq
```

``` json

{
  "Subnets": [
    {
      "AvailabilityZone": "us-east-1b",
      "AvailabilityZoneId": "use1-az1",
      "AvailableIpAddressCount": 11,
      "CidrBlock": "10.0.10.16/28",
      "DefaultForAz": false,
      "MapPublicIpOnLaunch": false,
      "MapCustomerOwnedIpOnLaunch": false,
      "State": "available",
      "SubnetId": "subnet-07c81d6d42836a8f9",
      "VpcId": "vpc-08aee6f377637f9ae",
      "OwnerId": " ",
      "AssignIpv6AddressOnCreation": false,
      "Ipv6CidrBlockAssociationSet": [],
      "Tags": [
        {
          "Key": "Name",
          "Value": "AWS-UE1-VPC-MGMT -subnet-public2-us-east-1b"
        }
      ],
      "SubnetArn": "arn:aws:ec2:us-east-1: :subnet/subnet-07c81d6d42836a8f9",
      "EnableDns64": false,
      "Ipv6Native": false,
      "PrivateDnsNameOptionsOnLaunch": {
        "HostnameType": "ip-name",
        "EnableResourceNameDnsARecord": false,
        "EnableResourceNameDnsAAAARecord": false
      }
    },
    {
      "AvailabilityZone": "us-east-1b",
      "AvailabilityZoneId": "use1-az1",
      "AvailableIpAddressCount": 11,
      "CidrBlock": "10.0.10.144/28",
      "DefaultForAz": false,
      "MapPublicIpOnLaunch": false,
      "MapCustomerOwnedIpOnLaunch": false,
      "State": "available",
      "SubnetId": "subnet-0f75d35ae1da469cc",
      "VpcId": "vpc-08aee6f377637f9ae",
      "OwnerId": " ",
      "AssignIpv6AddressOnCreation": false,
      "Ipv6CidrBlockAssociationSet": [],
      "Tags": [
        {
          "Key": "Name",
          "Value": "AWS-UE1-VPC-MGMT -subnet-private2-us-east-1b"
        }
      ],
      "SubnetArn": "arn:aws:ec2:us-east-1: :subnet/subnet-0f75d35ae1da469cc",
      "EnableDns64": false,
      "Ipv6Native": false,
      "PrivateDnsNameOptionsOnLaunch": {
        "HostnameType": "ip-name",
        "EnableResourceNameDnsARecord": false,
        "EnableResourceNameDnsAAAARecord": false
      }
    },
    {
      "AvailabilityZone": "us-east-1a",
      "AvailabilityZoneId": "use1-az6",
      "AvailableIpAddressCount": 11,
      "CidrBlock": "10.0.10.128/28",
      "DefaultForAz": false,
      "MapPublicIpOnLaunch": false,
      "MapCustomerOwnedIpOnLaunch": false,
      "State": "available",
      "SubnetId": "subnet-0c363777a7f301bc2",
      "VpcId": "vpc-08aee6f377637f9ae",
      "OwnerId": " ",
      "AssignIpv6AddressOnCreation": false,
      "Ipv6CidrBlockAssociationSet": [],
      "Tags": [
        {
          "Key": "Name",
          "Value": "AWS-UE1-VPC-MGMT -subnet-private1-us-east-1a"
        }
      ],
      "SubnetArn": "arn:aws:ec2:us-east-1: :subnet/subnet-0c363777a7f301bc2",
      "EnableDns64": false,
      "Ipv6Native": false,
      "PrivateDnsNameOptionsOnLaunch": {
        "HostnameType": "ip-name",
        "EnableResourceNameDnsARecord": false,
        "EnableResourceNameDnsAAAARecord": false
      }
    },
    {
      "AvailabilityZone": "us-east-1a",
      "AvailabilityZoneId": "use1-az6",
      "AvailableIpAddressCount": 11,
      "CidrBlock": "10.0.10.0/28",
      "DefaultForAz": false,
      "MapPublicIpOnLaunch": false,
      "MapCustomerOwnedIpOnLaunch": false,
      "State": "available",
      "SubnetId": "subnet-0f2351620e76a54d7",
      "VpcId": "vpc-08aee6f377637f9ae",
      "OwnerId": " ",
      "AssignIpv6AddressOnCreation": false,
      "Ipv6CidrBlockAssociationSet": [],
      "Tags": [
        {
          "Key": "Name",
          "Value": "AWS-UE1-VPC-MGMT -subnet-public1-us-east-1a"
        }
      ],
      "SubnetArn": "arn:aws:ec2:us-east-1: :subnet/subnet-0f2351620e76a54d7",
      "EnableDns64": false,
      "Ipv6Native": false,
      "PrivateDnsNameOptionsOnLaunch": {
        "HostnameType": "ip-name",
        "EnableResourceNameDnsARecord": false,
        "EnableResourceNameDnsAAAARecord": false
      }
    }
  ]
}
```

Then we can have a look at the route tables that are associated with the VPC created previously.

```

$ aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-08aee6f377637f9ae" | jq
```

``` json
{
  "RouteTables": [
    {
      "Associations": [
        {
          "Main": false,
          "RouteTableAssociationId": "rtbassoc-02429aa65c6ca4696",
          "RouteTableId": "rtb-04083da70542085d7",
          "SubnetId": "subnet-0f75d35ae1da469cc",
          "AssociationState": {
            "State": "associated"
          }
        }
      ],
      "PropagatingVgws": [],
      "RouteTableId": "rtb-04083da70542085d7",
      "Routes": [
        {
          "DestinationCidrBlock": "10.0.10.0/24",
          "GatewayId": "local",
          "Origin": "CreateRouteTable",
          "State": "active"
        },
        {
          "DestinationPrefixListId": "pl-63a5400a",
          "GatewayId": "vpce-07c5489eece54ac5c",
          "Origin": "CreateRoute",
          "State": "active"
        }
      ],
      "Tags": [
        {
          "Key": "Name",
          "Value": "AWS-UE1-VPC-MGMT -rtb-private2-us-east-1b"
        }
      ],
      "VpcId": "vpc-08aee6f377637f9ae",
      "OwnerId": "   "
    },
    {
      "Associations": [
        {
          "Main": true,
          "RouteTableAssociationId": "rtbassoc-032f087a7405940dd",
          "RouteTableId": "rtb-050a2ad500f057f3b",
          "AssociationState": {
            "State": "associated"
          }
        }
      ],
      "PropagatingVgws": [],
      "RouteTableId": "rtb-050a2ad500f057f3b",
      "Routes": [
        {
          "DestinationCidrBlock": "10.0.10.0/24",
          "GatewayId": "local",
          "Origin": "CreateRouteTable",
          "State": "active"
        }
      ],
      "Tags": [],
      "VpcId": "vpc-08aee6f377637f9ae",
      "OwnerId": "   "
    },
    {
      "Associations": [
        {
          "Main": false,
          "RouteTableAssociationId": "rtbassoc-049270142f38cc317",
          "RouteTableId": "rtb-0726a61879b6886c8",
          "SubnetId": "subnet-07c81d6d42836a8f9",
          "AssociationState": {
            "State": "associated"
          }
        },
        {
          "Main": false,
          "RouteTableAssociationId": "rtbassoc-0f4a7bb9b742f8840",
          "RouteTableId": "rtb-0726a61879b6886c8",
          "SubnetId": "subnet-0f2351620e76a54d7",
          "AssociationState": {
            "State": "associated"
          }
        }
      ],
      "PropagatingVgws": [],
      "RouteTableId": "rtb-0726a61879b6886c8",
      "Routes": [
        {
          "DestinationCidrBlock": "10.0.10.0/24",
          "GatewayId": "local",
          "Origin": "CreateRouteTable",
          "State": "active"
        },
        {
          "DestinationCidrBlock": "0.0.0.0/0",
          "GatewayId": "igw-08cb7b3514b815ae0",
          "Origin": "CreateRoute",
          "State": "active"
        }
      ],
      "Tags": [
        {
          "Key": "Name",
          "Value": "AWS-UE1-VPC-MGMT -rtb-public"
        }
      ],
      "VpcId": "vpc-08aee6f377637f9ae",
      "OwnerId": "   "
    },
    {
      "Associations": [
        {
          "Main": false,
          "RouteTableAssociationId": "rtbassoc-0a9141e0861dbca9c",
          "RouteTableId": "rtb-0bbeb2d8088d29033",
          "SubnetId": "subnet-0c363777a7f301bc2",
          "AssociationState": {
            "State": "associated"
          }
        }
      ],
      "PropagatingVgws": [],
      "RouteTableId": "rtb-0bbeb2d8088d29033",
      "Routes": [
        {
          "DestinationCidrBlock": "10.0.10.0/24",
          "GatewayId": "local",
          "Origin": "CreateRouteTable",
          "State": "active"
        },
        {
          "DestinationPrefixListId": "pl-63a5400a",
          "GatewayId": "vpce-07c5489eece54ac5c",
          "Origin": "CreateRoute",
          "State": "active"
        }
      ],
      "Tags": [dzoo
        {
          "Key": "Name",
          "Value": "AWS-UE1-VPC-MGMT -rtb-private1-us-east-1a"
        }
      ],
      "VpcId": "vpc-08aee6f377637f9ae",
      "OwnerId": "   "
    }
  ]
}
```

Finally, let's dig deeper and have a look at the public routing tables entries and how it is associated.

We can see that this routing table is shared with 2 subnets: subnet-07c81d6d42836a8f9 and subnet-0f2351620e76a54d7 (both public subnets in the VPC on different AZ).

The routing table has the 2 entries mentionned above:
+ 10.0.10.0/24 for the local subnet
+ 0.0.0.0/0 that points to the internet gateway (igw-08cb7b3514b815ae0) used by the public subnet in the VPC


```
aws ec2 describe-route-tables --filters "Name=tag:Name,Values=AWS-UE1-VPC-MGMT -rtb-public" | jq
```

``` json
{
  "RouteTables": [
    {
      "Associations": [
        {
          "Main": false,
          "RouteTableAssociationId": "rtbassoc-049270142f38cc317",
          "RouteTableId": "rtb-0726a61879b6886c8",
          "SubnetId": "subnet-07c81d6d42836a8f9",
          "AssociationState": {
            "State": "associated"
          }
        },
        {
          "Main": false,
          "RouteTableAssociationId": "rtbassoc-0f4a7bb9b742f8840",
          "RouteTableId": "rtb-0726a61879b6886c8",
          "SubnetId": "subnet-0f2351620e76a54d7",
          "AssociationState": {
            "State": "associated"
          }
        }
      ],
      "PropagatingVgws": [],
      "RouteTableId": "rtb-0726a61879b6886c8",
      "Routes": [
        {
          "DestinationCidrBlock": "10.0.10.0/24",
          "GatewayId": "local",
          "Origin": "CreateRouteTable",
          "State": "active"
        },
        {
          "DestinationCidrBlock": "0.0.0.0/0",
          "GatewayId": "igw-08cb7b3514b815ae0",
          "Origin": "CreateRoute",
          "State": "active"
        }
      ],
      "Tags": [
        {
          "Key": "Name",
          "Value": "AWS-UE1-VPC-MGMT -rtb-public"
        }
      ],
      "VpcId": "vpc-08aee6f377637f9ae",
      "OwnerId": "   "
    }
  ]
}
```

last edited: October 28th 2022