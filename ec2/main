## AWS Auth
provider "aws" {
  version = "~>3.0"
  region  = "ap-south-1"
}

## Remote S3 Data Block
terraform {
  backend "s3" {
    bucket = "ec2-bucket-777"
    key    = "terraform/atlantisterraform/state"
    region = "ap-south-1"
  }
}

## Create an EC2 Instance
resource "aws_instance" "web" {
  ami               = "ami-0449c34f967dbf18a"
  instance_type     = "t2.medium"
  availability_zone = "ap-south-1b"
  tags = {
    Name = "atlantis-medium"
  }
}
