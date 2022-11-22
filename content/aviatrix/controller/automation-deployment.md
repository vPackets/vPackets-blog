---
title: " Deploy a Controller - Terraform"
description: " Use Terraform to deploy an Aviatrix Controller in AWS"
date: 2022-08-23T13:57:51-05:00
draft: false
---

Now that we have seen the steps to deploy an Aviatrix controller in AWS, we are going to see how easy it when we use [Infrastructure as Code](https://en.wikipedia.org/wiki/Infrastructure_as_code)


The following [Terraform Registry](https://registry.terraform.io/namespaces/AviatrixSystems) shows the different registries and examples regarding the controller deployment in [AWS](https://aws.amazon.com/), [Azure](https://azure.microsoft.com/en-us/get-started/azure-portal), [GCP](https://cloud.google.com/) and [OCI](https://www.oracle.com/cloud/). In my example I am using AWS and will demonstrate how easy and convenient it is.

We will use the following [AVX controller in AWS registry](https://registry.terraform.io/modules/AviatrixSystems/aws-controller/aviatrix/latest).

My IAM roles have already been created previously and must be configured on the Aviatrix Controller EC2 instance.

The [request](https://pypi.org/project/requests/) library is needed to send the accurate HTTP requests for our deployment.

The documentation states that only 5 variables are needed to deploy the controller but I will use other parameters to deploy my controller.

It is important to fathom the fact that a security group and external IP address will be automatically assigned to the controller EC2 instance. You can modify that easily using the AWS CLI as we will demonstrate:

Here is my main.tf file:


``` bash

module "aviatrix_controller_aws" {
  source               = "AviatrixSystems/aws-controller/aviatrix"
  create_iam_roles     = false
  incoming_ssl_cidrs   = ["{use your own IP here}"]
  admin_password       = "{Use your own password here}"
  admin_email          = "{use your email here}"
  access_account_name  = "{use your account name/ID}"
  access_account_email = "{use your email here}"
  customer_license_id  = "{use your BYOL license here}"
  type                 = "BYOL"
  availability_zone    = "us-east-1a"
  use_existing_vpc     = true
  vpc_id               = "{use your VPC ID here}"
  subnet_cidr          = "10.0.10.0/28"
  subnet_id            = "{use your Subnet ID here}"
  use_existing_keypair = true
  key_pair_name        = "vPackets-AVX"
  termination_protection = "true"
  instance_type        = "t3.large"
  controller_name      = "AVX-Controller"
  
}

output "public_ip" {
  value = module.aviatrix_controller_aws.public_ip
}

output "private_ip" {
  value = module.aviatrix_controller_aws.private_ip
}

```


A terraform plan will allow us to assess the current state in AWS and report what objects will be created:

``` bash

nmichel@vPackets.local:/Users/nmichel/code/AVIATRIX/AVX-IAC/controller $ terraform plan                                                                              [11/22/22 - 0:36:01]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.http.avx_iam_id: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.http.avx_iam_id: Read complete after 0s [id=https://s3-us-west-2.amazonaws.com/aviatrix-download/AMI_ID/ami_id.json]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_region.current: Reading...
module.aviatrix_controller_aws.data.aws_caller_identity.current: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_availability_zones.all: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_vpc.controller_vpc: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_region.current: Read complete after 0s [id=us-east-1]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_availability_zones.all: Read complete after 0s [id=us-east-1]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1b"]: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1f"]: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1a"]: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1c"]: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1e"]: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1d"]: Reading...
module.aviatrix_controller_aws.data.aws_caller_identity.current: Read complete after 0s [id=312785526160]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1b"]: Read complete after 0s [id=t3.large]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1a"]: Read complete after 0s [id=t3.large]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1f"]: Read complete after 0s [id=t3.large]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1c"]: Read complete after 0s [id=t3.large]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1d"]: Read complete after 0s [id=t3.large]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1e"]: Read complete after 0s [id=t2.micro]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_subnet.controller_subnet: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_subnet.controller_subnet: Read complete after 0s [id=subnet-02f723bae6b659686]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_vpc.controller_vpc: Read complete after 1s [id=vpc-0939e782f90671bfb]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create
 <= read (data resources)

Terraform will perform the following actions:

  # module.aviatrix_controller_aws.module.aviatrix_controller_build.aws_eip.controller_eip will be created
  + resource "aws_eip" "controller_eip" {
      + allocation_id        = (known after apply)
      + association_id       = (known after apply)
      + carrier_ip           = (known after apply)
      + customer_owned_ip    = (known after apply)
      + domain               = (known after apply)
      + id                   = (known after apply)
      + instance             = (known after apply)
      + network_border_group = (known after apply)
      + network_interface    = (known after apply)
      + private_dns          = (known after apply)
      + private_ip           = (known after apply)
      + public_dns           = (known after apply)
      + public_ip            = (known after apply)
      + public_ipv4_pool     = (known after apply)
      + tags                 = {
          + "Createdby" = "Terraform+Aviatrix"
          + "module"    = "aviatrix-controller-build"
        }
      + tags_all             = {
          + "Createdby" = "Terraform+Aviatrix"
          + "module"    = "aviatrix-controller-build"
        }
      + vpc                  = true
    }

  # module.aviatrix_controller_aws.module.aviatrix_controller_build.aws_eip_association.eip_assoc will be created
  + resource "aws_eip_association" "eip_assoc" {
      + allocation_id        = (known after apply)
      + id                   = (known after apply)
      + instance_id          = (known after apply)
      + network_interface_id = (known after apply)
      + private_ip_address   = (known after apply)
      + public_ip            = (known after apply)
    }

  # module.aviatrix_controller_aws.module.aviatrix_controller_build.aws_instance.aviatrix_controller will be created
  + resource "aws_instance" "aviatrix_controller" {
      + ami                                  = "ami-083debf283d5fab1f"
      + arn                                  = (known after apply)
      + associate_public_ip_address          = (known after apply)
      + availability_zone                    = "us-east-1a"
      + cpu_core_count                       = (known after apply)
      + cpu_threads_per_core                 = (known after apply)
      + disable_api_stop                     = (known after apply)
      + disable_api_termination              = true
      + ebs_optimized                        = (known after apply)
      + get_password_data                    = false
      + host_id                              = (known after apply)
      + host_resource_group_arn              = (known after apply)
      + iam_instance_profile                 = "aviatrix-role-ec2"
      + id                                   = (known after apply)
      + instance_initiated_shutdown_behavior = (known after apply)
      + instance_state                       = (known after apply)
      + instance_type                        = "t3.large"
      + ipv6_address_count                   = (known after apply)
      + ipv6_addresses                       = (known after apply)
      + key_name                             = "vPackets-AVX"
      + monitoring                           = (known after apply)
      + outpost_arn                          = (known after apply)
      + password_data                        = (known after apply)
      + placement_group                      = (known after apply)
      + placement_partition_number           = (known after apply)
      + primary_network_interface_id         = (known after apply)
      + private_dns                          = (known after apply)
      + private_ip                           = (known after apply)
      + public_dns                           = (known after apply)
      + public_ip                            = (known after apply)
      + secondary_private_ips                = (known after apply)
      + security_groups                      = (known after apply)
      + subnet_id                            = (known after apply)
      + tags                                 = {
          + "Createdby" = "Terraform+Aviatrix"
          + "Name"      = "AVX-Controller"
          + "module"    = "aviatrix-controller-build"
        }
      + tags_all                             = {
          + "Createdby" = "Terraform+Aviatrix"
          + "Name"      = "AVX-Controller"
          + "module"    = "aviatrix-controller-build"
        }
      + tenancy                              = (known after apply)
      + user_data                            = (known after apply)
      + user_data_base64                     = (known after apply)
      + user_data_replace_on_change          = false
      + vpc_security_group_ids               = (known after apply)

      + capacity_reservation_specification {
          + capacity_reservation_preference = (known after apply)

          + capacity_reservation_target {
              + capacity_reservation_id                 = (known after apply)
              + capacity_reservation_resource_group_arn = (known after apply)
            }
        }

      + ebs_block_device {
          + delete_on_termination = (known after apply)
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + snapshot_id           = (known after apply)
          + tags                  = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = (known after apply)
          + volume_type           = (known after apply)
        }

      + enclave_options {
          + enabled = (known after apply)
        }

      + ephemeral_block_device {
          + device_name  = (known after apply)
          + no_device    = (known after apply)
          + virtual_name = (known after apply)
        }

      + maintenance_options {
          + auto_recovery = (known after apply)
        }

      + metadata_options {
          + http_endpoint               = (known after apply)
          + http_put_response_hop_limit = (known after apply)
          + http_tokens                 = (known after apply)
          + instance_metadata_tags      = (known after apply)
        }

      + network_interface {
          + delete_on_termination = false
          + device_index          = 0
          + network_card_index    = 0
          + network_interface_id  = (known after apply)
        }

      + private_dns_name_options {
          + enable_resource_name_dns_a_record    = (known after apply)
          + enable_resource_name_dns_aaaa_record = (known after apply)
          + hostname_type                        = (known after apply)
        }

      + root_block_device {
          + delete_on_termination = true
          + device_name           = (known after apply)
          + encrypted             = (known after apply)
          + iops                  = (known after apply)
          + kms_key_id            = (known after apply)
          + throughput            = (known after apply)
          + volume_id             = (known after apply)
          + volume_size           = 64
          + volume_type           = "gp2"
        }
    }

  # module.aviatrix_controller_aws.module.aviatrix_controller_build.aws_network_interface.eni_controller will be created
  + resource "aws_network_interface" "eni_controller" {
      + arn                       = (known after apply)
      + id                        = (known after apply)
      + interface_type            = (known after apply)
      + ipv4_prefix_count         = (known after apply)
      + ipv4_prefixes             = (known after apply)
      + ipv6_address_count        = (known after apply)
      + ipv6_address_list         = (known after apply)
      + ipv6_address_list_enabled = false
      + ipv6_addresses            = (known after apply)
      + ipv6_prefix_count         = (known after apply)
      + ipv6_prefixes             = (known after apply)
      + mac_address               = (known after apply)
      + outpost_arn               = (known after apply)
      + owner_id                  = (known after apply)
      + private_dns_name          = (known after apply)
      + private_ip                = (known after apply)
      + private_ip_list           = (known after apply)
      + private_ip_list_enabled   = false
      + private_ips               = (known after apply)
      + private_ips_count         = (known after apply)
      + security_groups           = (known after apply)
      + source_dest_check         = true
      + subnet_id                 = "subnet-02f723bae6b659686"
      + tags                      = {
          + "Createdby" = "Terraform+Aviatrix"
          + "Name"      = "controller_network_interface"
          + "module"    = "aviatrix-controller-build"
        }
      + tags_all                  = {
          + "Createdby" = "Terraform+Aviatrix"
          + "Name"      = "controller_network_interface"
          + "module"    = "aviatrix-controller-build"
        }

      + attachment {
          + attachment_id = (known after apply)
          + device_index  = (known after apply)
          + instance      = (known after apply)
        }
    }

  # module.aviatrix_controller_aws.module.aviatrix_controller_build.aws_security_group.aviatrix_security_group will be created
  + resource "aws_security_group" "aviatrix_security_group" {
      + arn                    = (known after apply)
      + description            = "Aviatrix - Controller Security Group"
      + egress                 = (known after apply)
      + id                     = (known after apply)
      + ingress                = (known after apply)
      + name                   = "AviatrixSecurityGroup"
      + name_prefix            = (known after apply)
      + owner_id               = (known after apply)
      + revoke_rules_on_delete = false
      + tags                   = {
          + "Createdby" = "Terraform+Aviatrix"
          + "Name"      = "controller_security_group"
          + "module"    = "aviatrix-controller-build"
        }
      + tags_all               = {
          + "Createdby" = "Terraform+Aviatrix"
          + "Name"      = "controller_security_group"
          + "module"    = "aviatrix-controller-build"
        }
      + vpc_id                 = "vpc-0939e782f90671bfb"
    }

  # module.aviatrix_controller_aws.module.aviatrix_controller_build.aws_security_group_rule.egress_rule will be created
  + resource "aws_security_group_rule" "egress_rule" {
      + cidr_blocks              = [
          + "0.0.0.0/0",
        ]
      + from_port                = 0
      + id                       = (known after apply)
      + protocol                 = "-1"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 0
      + type                     = "egress"
    }

  # module.aviatrix_controller_aws.module.aviatrix_controller_build.aws_security_group_rule.ingress_rule will be created
  + resource "aws_security_group_rule" "ingress_rule" {
      + cidr_blocks              = [
          + "",
        ]
      + from_port                = 443
      + id                       = (known after apply)
      + protocol                 = "tcp"
      + security_group_id        = (known after apply)
      + security_group_rule_id   = (known after apply)
      + self                     = false
      + source_security_group_id = (known after apply)
      + to_port                  = 443
      + type                     = "ingress"
    }

  # module.aviatrix_controller_aws.module.aviatrix_controller_initialize.data.aws_caller_identity.current will be read during apply
  # (depends on a resource or a module with changes pending)
 <= data "aws_caller_identity" "current" {
      + account_id = (known after apply)
      + arn        = (known after apply)
      + id         = (known after apply)
      + user_id    = (known after apply)
    }

  # module.aviatrix_controller_aws.module.aviatrix_controller_initialize.data.aws_region.current will be read during apply
  # (depends on a resource or a module with changes pending)
 <= data "aws_region" "current" {
      + description = (known after apply)
      + endpoint    = (known after apply)
      + id          = (known after apply)
      + name        = (known after apply)
    }

  # module.aviatrix_controller_aws.module.aviatrix_controller_initialize.null_resource.run_script will be created
  + resource "null_resource" "run_script" {
      + id       = (known after apply)
      + triggers = {
          + "argument_destroy" = (known after apply)
        }
    }

Plan: 8 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + private_ip = (known after apply)
  + public_ip  = (known after apply)

─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
nmichel@vPackets.local:/Users/nmichel/code/AVIATRIX/AVX-IAC/controller $                   
```


A Terraform apply will deploy the AWS EC2 instance with the default security group and external IP address assigned.

We will change that after the terraform apply. Code ommited to facilitate clarity.


``` bash

nmichel@vPackets.local:/Users/nmichel/code/AVIATRIX/AVX-IAC/controller $ terraform apply                                                                             [11/22/22 - 0:37:46]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.http.avx_iam_id: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.http.avx_iam_id: Read complete after 0s [id=https://s3-us-west-2.amazonaws.com/aviatrix-download/AMI_ID/ami_id.json]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_availability_zones.all: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_vpc.controller_vpc: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_region.current: Reading...
module.aviatrix_controller_aws.data.aws_caller_identity.current: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_region.current: Read complete after 0s [id=us-east-1]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_availability_zones.all: Read complete after 0s [id=us-east-1]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1f"]: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1e"]: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1a"]: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1d"]: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1c"]: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1b"]: Reading...
module.aviatrix_controller_aws.data.aws_caller_identity.current: Read complete after 1s [id=312785526160]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1f"]: Read complete after 1s [id=t3.large]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1c"]: Read complete after 1s [id=t3.large]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1e"]: Read complete after 1s [id=t2.micro]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1a"]: Read complete after 1s [id=t3.large]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1b"]: Read complete after 1s [id=t3.large]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_ec2_instance_type_offering.offering["us-east-1d"]: Read complete after 1s [id=t3.large]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_subnet.controller_subnet: Reading...
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_subnet.controller_subnet: Read complete after 0s [id=subnet-02f723bae6b659686]
module.aviatrix_controller_aws.module.aviatrix_controller_build.data.aws_vpc.controller_vpc: Read complete after 1s [id=vpc-0939e782f90671bfb]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create
 <= read (data resources)

Terraform will perform the following actions:

  # module.aviatrix_controller_aws.module.aviatrix_controller_build.aws_eip.controller_eip will be created
  + resource "aws_eip" "controller_eip" {
      + allocation_id        = (known after apply)
      + association_id       = (known after apply)
      + carrier_ip           = (known after apply)
      + customer_owned_ip    = (known after apply)
      + domain               = (known after apply)
      + id                   = (known after apply)
      + instance             = (known after apply)
      + network_border_group = (known after apply)
      + network_interface    = (known after apply)
      + private_dns          = (known after apply)
      + private_ip           = (known after apply)
      + public_dns           = (known after apply)
      + public_ip            = (known after apply)
      + public_ipv4_pool     = (known after apply)



......
......

Apply complete! Resources: 8 added, 0 changed, 0 destroyed.

Outputs:

private_ip = "10.0.10.14"
public_ip = "34.235.66.100"

```

There is another way to verify the parameters assigned during the creation:
Terraform informs us that the private_ip of the controller is 10.0.10.14, the public IP assigned is 34.235.66.100.

``` bash
terraform output | sed -ne 's/\(.*\) = \(.*\)/\1="\2"/p' > ../aviatrix_controller.tfvars
```

``` bash
private_ip=""10.0.10.14""
public_ip=""34.235.66.100""
```

We will assign an elastic IP to the controller. The EC2 instance has an ID of i-08e9bfdfa2e1d21bf. We will reuse that ID

``` bash
aws ec2 associate-address --instance-id i-08e9bfdfa2e1d21bf --public-ip a.b.c.d
```


``` bash
aws ec2 modify-instance-attribute --instance-id i-08e9bfdfa2e1d21bf --groups {Use your Security group ID}
```

My controller is now fully reachable from my the subnets configured in my security group.