# terraform
An introductory project on terraform with aws

Tutorial Followed - https://www.youtube.com/watch?v=SLB_c_ayRMo&ab_channel=freeCodeCamp.org

## Application of Terraform in this project:

1. Creating an aws provider
   
    ```
    provider "aws" {
    region = "us-east-1"
    access_key = "your aws access key"
    secret_key = "your aws secret key"
    }
    ```
2. Creating aws vpc

    ```
        resource "aws_vpc" "prod-vpc" {
            cidr_block = "10.0.0.0/16"
            tags = {
                Name = "production"
            }
        }
    ```
3. Creating aws subnet

    ```
        variable "subnet_prefix" {
            description = "cidr block ip address declaration"
            type = string
        }

        resource "aws_subnet" "subnet-1" {
            vpc_id     = aws_vpc.prod-vpc.id
            cidr_block = var.subnet_prefix
            availability_zone = "us-east-1a"

            tags = {
                Name = "prod-subnet"
            }
        }
    ```

4. Creating aws internet gateway

    ```
        resource "aws_internet_gateway" "gw" {
            vpc_id = aws_vpc.prod-vpc.id
        }
    ```
5. Creating aws route table
   
   ```
    resource "aws_route_table" "prod-route-table" {
        vpc_id = aws_vpc.prod-vpc.id

        route {
            cidr_block = "0.0.0.0/0"
            gateway_id = aws_internet_gateway.gw.id
        }

        route {
            ipv6_cidr_block        = "::/0"
            gateway_id = aws_internet_gateway.gw.id
        }

        tags = {
            Name = "Prod"
        }
    }

    resource "aws_route_table_association" "a" {
        subnet_id      = aws_subnet.subnet-1.id
    route_table_id = aws_route_table.prod-route-table.id
    }
   ```
6. Creating aws security group

    ```
        resource "aws_security_group" "allow_web" {
            name        = "allow_web_traffic"
            description = "Allow Web inbound traffic"
            vpc_id      = aws_vpc.prod-vpc.id

            ingress {
                description      = "HTTPS from VPC"
                from_port        = 443
                to_port          = 443
                protocol         = "tcp"
                cidr_blocks      = ["0.0.0.0/0"]
                ipv6_cidr_blocks = ["::/0"]
            }

            ingress {
                description      = "HTTP from VPC"
                from_port        = 80
                to_port          = 80
                protocol         = "tcp"
                cidr_blocks      = ["0.0.0.0/0"]
                ipv6_cidr_blocks = ["::/0"]
            }

            ingress {
                description      = "SSH from VPC"
                from_port        = 22
                to_port          = 22
                protocol         = "tcp"
                cidr_blocks      = ["0.0.0.0/0"]
                ipv6_cidr_blocks = ["::/0"]
            }

            egress {
                from_port        = 0
                to_port          = 0
                protocol         = "-1"
                cidr_blocks      = ["0.0.0.0/0"]
                ipv6_cidr_blocks = ["::/0"]
            }

            tags = {
                Name = "allow_web"
            }
        }
    ```
7. Creating aws network interface

    ```
        resource "aws_network_interface" "web-server-nic" {
            subnet_id       = aws_subnet.subnet-1.id
            private_ips     = ["10.0.1.50"]
            security_groups = [aws_security_group.allow_web .id]
        }

        resource "aws_eip" "one" {
            vpc                       = true
            network_interface         = aws_network_interface.web-server-nic.id
            associate_with_private_ip = "10.0.1.50"
            depends_on = [
                aws_internet_gateway.gw
            ]
        }
    ```
8. Creating an ec2 instance

    ```
        resource "aws_instance" "web-server-instance" {
            ami = "ami-042e8287309f5df03"
            instance_type = "t2.micro"
            availability_zone = "us-east-1a"
            key_name = "main-key"
            network_interface {
                device_index = 0
                network_interface_id = aws_network_interface.web-server-nic.id
            }
    ```


## Common Terraform Commands
1. terraform init
2. terraform plan
3. terraform apply
4. terraform apply --auto-approve
5. terraform apply -target aws_item
6. terraform destroy
7. terraform destroy -target aws_item
8. terraform refresh
9. terraform state list
10. terraform state show your_aws_instance_item
11. terraform output