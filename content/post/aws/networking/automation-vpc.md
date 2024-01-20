+++
title = "Create a VPC - Automation"
description = "Walkthrough of a VPC creation in AWS"
date = "2023-03-11"

weight = 3

tags = [
    "AWS",
    "VPC"
]

categories = [
    "Cloud Networking",
]
+++


In a previous blogpost we managed to manually create a simple VPC that could host several services or traditionnal [EC2 instances](https://aws.amazon.com/pm/ec2/).
This process was quite simple using the AWS console as it would deploy all the network construct within a VPC (Subnets, Route Tables, Internet Gateways etc...)

I have been deploying network infrastructure (NSX, Cumulus, Cisco ...) for the past few years using [Infrastructure as Code](https://www.ipspace.net/kb/tag/network-infrastructure-as-code.html) and naturally I will show you how to deploy networking cloud constructs related to AWS (not only !) using [Terraform](https://cloud.hashicorp.com/products/terraform).

First, Terraform will need a way to authenticate with AWS (covered in this [documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs), this [AWS documentation](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html) is also very valuable). 

There are plenty of examples on the Internet for this but the official terraform aws module is quite [complete](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest), it points to a [GitHub repo](https://github.com/terraform-aws-modules/terraform-aws-vpc/tree/master/examples/complete-vpc) that is very well presented and easy to follow.

My code for this particular deployment is very simple:


``` bash

provider "aws" {
  shared_config_files      = ["/Users/nmichel/.aws/config"]
  shared_credentials_files = ["/Users/nmichel/.aws/credentials"]
  #profile                  = "default"
  region = "us-east-1"
}

locals {
  name   = "AWS-UE1-VPC-MGMT"
  region = "us-east-1"
  tags = {
    Owner       = "Nicolas MICHEL"
    Environment = "IT Services"
    #Name        = "AWS-UE1-VPC-MGMT"
  }
}


module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  version = "3.14.2"
  name = "AWS-UE1-VPC-MGMT"
  cidr = "10.0.10.0/24"

  azs             = ["${local.region}a", "${local.region}b"]
  private_subnets = ["10.0.10.96/28", "10.0.10.112/28"]
  public_subnets  = ["10.0.10.0/28", "10.0.10.16/28"]

  enable_nat_gateway = false
  enable_vpn_gateway = false

  manage_default_route_table = true
  default_route_table_tags   = { Name = "${local.name}-default" }


  tags = local.tags
}

resource "aws_security_group" "allow_ssh" {
  name        = "SG-vPackets-MGMT"
  description = "Allow SSH/HTTPS inbound traffic"
  vpc_id      = module.vpc.vpc_id
  #vpc_id      = aws_vpc.main.id

  ingress {
    description      = "SSH from VPC"
    from_port        = 22
    to_port          = 22
    protocol         = "tcp"
    cidr_blocks      = ["YOUR_IP"]
    #ipv6_cidr_blocks = [aws_vpc.main.ipv6_cidr_block]
  }
  ingress {
    description      = "TLS from VPC"
    from_port        = 443
    to_port          = 443
    protocol         = "tcp"
    cidr_blocks      = ["YOUR_IP"]
    #ipv6_cidr_blocks = [aws_vpc.main.ipv6_cidr_block]
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name = "SG-vPackets-MGMT"
  }
}

```


Same as previously it will create a simple VPC in 10 seconds and will allow me to install all my EC2 instances workloads. The only difference here is that I have included a Security Group that will allow inbound SSH and TLS traffic into EC2 instances linked with that Security Group. (please include your IP in the security group).

``` bash 
$ terraform plan         

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
symbols:
  + create

Terraform will perform the following actions:

  # module.vpc.aws_default_route_table.default[0] will be created
  + resource "aws_default_route_table" "default" {
      + arn                    = (known after apply)
      + default_route_table_id = (known after apply)
      + id                     = (known after apply)
      + owner_id               = (known after apply)
      + route                  = (known after apply)
      + tags                   = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-default"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all               = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-default"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id                 = (known after apply)

      + timeouts {
          + create = "5m"
          + update = "5m"
        }
    }

  # module.vpc.aws_internet_gateway.this[0] will be created
  + resource "aws_internet_gateway" "this" {
      + arn      = (known after apply)
      + id       = (known after apply)
      + owner_id = (known after apply)
      + tags     = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id   = (known after apply)
    }

  # module.vpc.aws_route.public_internet_gateway[0] will be created
  + resource "aws_route" "public_internet_gateway" {
      + destination_cidr_block = "0.0.0.0/0"
      + gateway_id             = (known after apply)
      + id                     = (known after apply)
      + instance_id            = (known after apply)
      + instance_owner_id      = (known after apply)
      + network_interface_id   = (known after apply)
      + origin                 = (known after apply)
      + route_table_id         = (known after apply)
      + state                  = (known after apply)

      + timeouts {
          + create = "5m"
        }
    }

  # module.vpc.aws_route_table.private[0] will be created
  + resource "aws_route_table" "private" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = (known after apply)
      + tags             = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1a"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all         = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1a"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id           = (known after apply)
    }

  # module.vpc.aws_route_table.private[1] will be created
  + resource "aws_route_table" "private" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = (known after apply)
      + tags             = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1b"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all         = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1b"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id           = (known after apply)
    }

  # module.vpc.aws_route_table.public[0] will be created
  + resource "aws_route_table" "public" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = (known after apply)
      + tags             = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-public"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all         = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-public"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id           = (known after apply)
    }

  # module.vpc.aws_route_table_association.private[0] will be created
  + resource "aws_route_table_association" "private" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.vpc.aws_route_table_association.private[1] will be created
  + resource "aws_route_table_association" "private" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.vpc.aws_route_table_association.public[0] will be created
  + resource "aws_route_table_association" "public" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.vpc.aws_route_table_association.public[1] will be created
  + resource "aws_route_table_association" "public" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.vpc.aws_subnet.private[0] will be created
  + resource "aws_subnet" "private" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1a"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.10.96/28"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = false
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1a"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all                                       = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1a"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.vpc.aws_subnet.private[1] will be created
  + resource "aws_subnet" "private" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1b"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.10.112/28"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = false
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1b"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all                                       = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1b"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.vpc.aws_subnet.public[0] will be created
  + resource "aws_subnet" "public" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1a"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.10.0/28"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-public-us-east-1a"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all                                       = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-public-us-east-1a"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.vpc.aws_subnet.public[1] will be created
  + resource "aws_subnet" "public" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1b"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.10.16/28"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-public-us-east-1b"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all                                       = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-public-us-east-1b"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.vpc.aws_vpc.this[0] will be created
  + resource "aws_vpc" "this" {
      + arn                                  = (known after apply)
      + assign_generated_ipv6_cidr_block     = false
      + cidr_block                           = "10.0.10.0/24"
      + default_network_acl_id               = (known after apply)
      + default_route_table_id               = (known after apply)
      + default_security_group_id            = (known after apply)
      + dhcp_options_id                      = (known after apply)
      + enable_classiclink                   = (known after apply)
      + enable_classiclink_dns_support       = (known after apply)
      + enable_dns_hostnames                 = false
      + enable_dns_support                   = true
      + id                                   = (known after apply)
      + instance_tenancy                     = "default"
      + ipv6_association_id                  = (known after apply)
      + ipv6_cidr_block                      = (known after apply)
      + ipv6_cidr_block_network_border_group = (known after apply)
      + main_route_table_id                  = (known after apply)
      + owner_id                             = (known after apply)
      + tags                                 = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all                             = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT"
          + "Owner"       = "Nicolas MICHEL"
        }
    }

Plan: 15 to add, 0 to change, 0 to destroy.

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run
"terraform apply" now.

```

Once you are ready to commit your configuration:

``` bash 
$ terraform apply --auto-approve

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following
symbols:
  + create

Terraform will perform the following actions:

  # module.vpc.aws_default_route_table.default[0] will be created
  + resource "aws_default_route_table" "default" {
      + arn                    = (known after apply)
      + default_route_table_id = (known after apply)
      + id                     = (known after apply)
      + owner_id               = (known after apply)
      + route                  = (known after apply)
      + tags                   = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-default"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all               = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-default"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id                 = (known after apply)

      + timeouts {
          + create = "5m"
          + update = "5m"
        }
    }

  # module.vpc.aws_internet_gateway.this[0] will be created
  + resource "aws_internet_gateway" "this" {
      + arn      = (known after apply)
      + id       = (known after apply)
      + owner_id = (known after apply)
      + tags     = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id   = (known after apply)
    }

  # module.vpc.aws_route.public_internet_gateway[0] will be created
  + resource "aws_route" "public_internet_gateway" {
      + destination_cidr_block = "0.0.0.0/0"
      + gateway_id             = (known after apply)
      + id                     = (known after apply)
      + instance_id            = (known after apply)
      + instance_owner_id      = (known after apply)
      + network_interface_id   = (known after apply)
      + origin                 = (known after apply)
      + route_table_id         = (known after apply)
      + state                  = (known after apply)

      + timeouts {
          + create = "5m"
        }
    }

  # module.vpc.aws_route_table.private[0] will be created
  + resource "aws_route_table" "private" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = (known after apply)
      + tags             = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1a"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all         = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1a"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id           = (known after apply)
    }

  # module.vpc.aws_route_table.private[1] will be created
  + resource "aws_route_table" "private" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = (known after apply)
      + tags             = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1b"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all         = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1b"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id           = (known after apply)
    }

  # module.vpc.aws_route_table.public[0] will be created
  + resource "aws_route_table" "public" {
      + arn              = (known after apply)
      + id               = (known after apply)
      + owner_id         = (known after apply)
      + propagating_vgws = (known after apply)
      + route            = (known after apply)
      + tags             = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-public"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all         = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-public"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id           = (known after apply)
    }

  # module.vpc.aws_route_table_association.private[0] will be created
  + resource "aws_route_table_association" "private" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.vpc.aws_route_table_association.private[1] will be created
  + resource "aws_route_table_association" "private" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.vpc.aws_route_table_association.public[0] will be created
  + resource "aws_route_table_association" "public" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.vpc.aws_route_table_association.public[1] will be created
  + resource "aws_route_table_association" "public" {
      + id             = (known after apply)
      + route_table_id = (known after apply)
      + subnet_id      = (known after apply)
    }

  # module.vpc.aws_subnet.private[0] will be created
  + resource "aws_subnet" "private" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1a"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.10.96/28"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = false
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1a"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all                                       = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1a"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.vpc.aws_subnet.private[1] will be created
  + resource "aws_subnet" "private" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1b"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.10.112/28"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = false
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1b"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all                                       = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-private-us-east-1b"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.vpc.aws_subnet.public[0] will be created
  + resource "aws_subnet" "public" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1a"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.10.0/28"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-public-us-east-1a"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all                                       = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-public-us-east-1a"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.vpc.aws_subnet.public[1] will be created
  + resource "aws_subnet" "public" {
      + arn                                            = (known after apply)
      + assign_ipv6_address_on_creation                = false
      + availability_zone                              = "us-east-1b"
      + availability_zone_id                           = (known after apply)
      + cidr_block                                     = "10.0.10.16/28"
      + enable_dns64                                   = false
      + enable_resource_name_dns_a_record_on_launch    = false
      + enable_resource_name_dns_aaaa_record_on_launch = false
      + id                                             = (known after apply)
      + ipv6_cidr_block_association_id                 = (known after apply)
      + ipv6_native                                    = false
      + map_public_ip_on_launch                        = true
      + owner_id                                       = (known after apply)
      + private_dns_hostname_type_on_launch            = (known after apply)
      + tags                                           = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-public-us-east-1b"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all                                       = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT-public-us-east-1b"
          + "Owner"       = "Nicolas MICHEL"
        }
      + vpc_id                                         = (known after apply)
    }

  # module.vpc.aws_vpc.this[0] will be created
  + resource "aws_vpc" "this" {
      + arn                                  = (known after apply)
      + assign_generated_ipv6_cidr_block     = false
      + cidr_block                           = "10.0.10.0/24"
      + default_network_acl_id               = (known after apply)
      + default_route_table_id               = (known after apply)
      + default_security_group_id            = (known after apply)
      + dhcp_options_id                      = (known after apply)
      + enable_classiclink                   = (known after apply)
      + enable_classiclink_dns_support       = (known after apply)
      + enable_dns_hostnames                 = false
      + enable_dns_support                   = true
      + id                                   = (known after apply)
      + instance_tenancy                     = "default"
      + ipv6_association_id                  = (known after apply)
      + ipv6_cidr_block                      = (known after apply)
      + ipv6_cidr_block_network_border_group = (known after apply)
      + main_route_table_id                  = (known after apply)
      + owner_id                             = (known after apply)
      + tags                                 = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT"
          + "Owner"       = "Nicolas MICHEL"
        }
      + tags_all                             = {
          + "Environment" = "IT Services"
          + "Name"        = "AWS-UE1-VPC-MGMT"
          + "Owner"       = "Nicolas MICHEL"
        }
    }

Plan: 15 to add, 0 to change, 0 to destroy.
module.vpc.aws_vpc.this[0]: Creating...
module.vpc.aws_vpc.this[0]: Creation complete after 2s [id=vpc-0d69b9d9d035a3211]
module.vpc.aws_internet_gateway.this[0]: Creating...
module.vpc.aws_route_table.private[1]: Creating...
module.vpc.aws_default_route_table.default[0]: Creating...
module.vpc.aws_subnet.private[0]: Creating...
module.vpc.aws_subnet.private[1]: Creating...
module.vpc.aws_route_table.private[0]: Creating...
module.vpc.aws_subnet.public[1]: Creating...
module.vpc.aws_subnet.public[0]: Creating...
module.vpc.aws_route_table.public[0]: Creating...
module.vpc.aws_default_route_table.default[0]: Creation complete after 0s [id=rtb-013e6bb525838db79]
module.vpc.aws_internet_gateway.this[0]: Creation complete after 1s [id=igw-0b0e542cb990236bb]
module.vpc.aws_route_table.private[1]: Creation complete after 1s [id=rtb-04bbfcd5698ba09b1]
module.vpc.aws_subnet.private[1]: Creation complete after 1s [id=subnet-045c21dd73cb6b884]
module.vpc.aws_route_table.public[0]: Creation complete after 1s [id=rtb-0ea1ec6dfcda17cfb]
module.vpc.aws_route.public_internet_gateway[0]: Creating...
module.vpc.aws_route_table.private[0]: Creation complete after 1s [id=rtb-0621af979022ced9e]
module.vpc.aws_subnet.private[0]: Creation complete after 1s [id=subnet-0ea8623065f7d1ad8]
module.vpc.aws_route_table_association.private[1]: Creating...
module.vpc.aws_route_table_association.private[0]: Creating...
module.vpc.aws_route.public_internet_gateway[0]: Creation complete after 0s [id=r-rtb-0ea1ec6dfcda17cfb1080289494]
module.vpc.aws_route_table_association.private[1]: Creation complete after 0s [id=rtbassoc-0004ba062692cf374]
module.vpc.aws_route_table_association.private[0]: Creation complete after 0s [id=rtbassoc-000104d5c187f0447]
module.vpc.aws_subnet.public[1]: Still creating... [10s elapsed]
module.vpc.aws_subnet.public[0]: Still creating... [10s elapsed]
module.vpc.aws_subnet.public[0]: Creation complete after 11s [id=subnet-091fc97a306565841]
module.vpc.aws_subnet.public[1]: Creation complete after 11s [id=subnet-028328f18f4e7abca]
module.vpc.aws_route_table_association.public[0]: Creating...
module.vpc.aws_route_table_association.public[1]: Creating...
module.vpc.aws_route_table_association.public[0]: Creation complete after 1s [id=rtbassoc-070ed95b691a0a6dc]
module.vpc.aws_route_table_association.public[1]: Creation complete after 1s [id=rtbassoc-026c58be638f03aaa]

Apply complete! Resources: 15 added, 0 changed, 0 destroyed.
```

And voila ! The last lines in the outputs are the most important and will show you the networks constructs created !
