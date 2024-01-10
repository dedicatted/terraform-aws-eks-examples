# Terraform Module: terraform-aws-eks-examples
# This module facilitates the creation of AWS EKS cluster with different options, including ec2 on demand, ec2 spot, fagrate and taints-tolerations example.

## Overview
The `terraform-aws-eks-examples` module includes examples to easily deploy AWS EKS using official Terraform module as a source. 

## Usage

Example for deploying EKS with one ec2 on demand node.

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}

module "vpc" {
  source             = "github.com/terraform-aws-modules/terraform-aws-vpc"
  name               = var.vpc_name
  cidr               = var.cidr_block
  azs                = slice(data.aws_availability_zones.available.names, 0, 3)
  public_subnets     = [cidrsubnet(var.cidr_block, 8, 3), cidrsubnet(var.cidr_block, 8, 4), cidrsubnet(var.cidr_block, 8, 5)]
  private_subnets    = [cidrsubnet(var.cidr_block, 8, 0), cidrsubnet(var.cidr_block, 8, 1), cidrsubnet(var.cidr_block, 8, 2)]
  enable_nat_gateway = true
  single_nat_gateway = true
  public_subnet_tags = {
    "kubernetes.io/cluster/demo-cluster" = "shared"
    "kubernetes.io/role/elb"             = 1
  }
  private_subnet_tags = {
    "kubernetes.io/cluster/demo-cluster" = "shared"
    "kubernetes.io/role/internal-elb"    = 1
  }
}

module "eks" {
  source                         = "github.com/terraform-aws-modules/terraform-aws-eks"
  cluster_name                   = "demo-cluster"
  vpc_id                         = module.vpc.vpc_id
  subnet_ids                     = module.vpc.private_subnets
  control_plane_subnet_ids       = module.vpc.public_subnets
  create_kms_key                 = true
  cluster_endpoint_public_access = true
  eks_managed_node_groups = {
    one = {
      name = "node-group-1"

      instance_types = ["t3.micro"]

      min_size     = 1
      max_size     = 1
      desired_size = 1
    }
  }
}
```
Example for deploying EKS with one ec2 spot node.

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}

module "vpc" {
  source             = "github.com/terraform-aws-modules/terraform-aws-vpc"
  name               = var.vpc_name
  cidr               = var.cidr_block
  azs                = slice(data.aws_availability_zones.available.names, 0, 3)
  public_subnets     = [cidrsubnet(var.cidr_block, 8, 3), cidrsubnet(var.cidr_block, 8, 4), cidrsubnet(var.cidr_block, 8, 5)]
  private_subnets    = [cidrsubnet(var.cidr_block, 8, 0), cidrsubnet(var.cidr_block, 8, 1), cidrsubnet(var.cidr_block, 8, 2)]
  enable_nat_gateway = true
  single_nat_gateway = true
  public_subnet_tags = {
    "kubernetes.io/cluster/demo-cluster" = "shared"
    "kubernetes.io/role/elb"             = 1
  }
  private_subnet_tags = {
    "kubernetes.io/cluster/demo-cluster" = "shared"
    "kubernetes.io/role/internal-elb"    = 1
  }
}

module "eks" {
  source                         = "github.com/terraform-aws-modules/terraform-aws-eks"
  cluster_name                   = "demo-cluster"
  vpc_id                         = module.vpc.vpc_id
  subnet_ids                     = module.vpc.private_subnets
  control_plane_subnet_ids       = module.vpc.public_subnets
  create_kms_key                 = true
  cluster_endpoint_public_access = true
  eks_managed_node_groups = {
    one = {
      node_group_name       = "mng-spot-4vcpu-16gb"
      capacity_type         = "SPOT"
      instance_types        = ["m4.xlarge", "m5.xlarge", "m5a.xlarge", "m5ad.xlarge", "m5d.xlarge", "t2.xlarge", "t3.xlarge", "t3a.xlarge"]
      max_size              = 1
      desired_size          = 1
      min_size              = 0
      create_iam_role       = true
      create_lauch_template = true
      subnet_ids            = module.vpc.private_subnets
    }
  }
}
```
Example for deploying EKS with fargate.

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}

module "vpc" {
  source             = "github.com/terraform-aws-modules/terraform-aws-vpc"
  name               = var.vpc_name
  cidr               = var.cidr_block
  azs                = slice(data.aws_availability_zones.available.names, 0, 3)
  public_subnets     = [cidrsubnet(var.cidr_block, 8, 3), cidrsubnet(var.cidr_block, 8, 4), cidrsubnet(var.cidr_block, 8, 5)]
  private_subnets    = [cidrsubnet(var.cidr_block, 8, 0), cidrsubnet(var.cidr_block, 8, 1), cidrsubnet(var.cidr_block, 8, 2)]
  enable_nat_gateway = true
  single_nat_gateway = true
  public_subnet_tags = {
    "kubernetes.io/cluster/demo-cluster" = "shared"
    "kubernetes.io/role/elb"             = 1
  }
  private_subnet_tags = {
    "kubernetes.io/cluster/demo-cluster" = "shared"
    "kubernetes.io/role/internal-elb"    = 1
  }
}

module "eks" {
  source                         = "github.com/terraform-aws-modules/terraform-aws-eks"
  cluster_name                   = "demo-cluster"
  vpc_id                         = module.vpc.vpc_id
  subnet_ids                     = module.vpc.private_subnets
  control_plane_subnet_ids       = module.vpc.public_subnets
  create_kms_key                 = true
  cluster_endpoint_public_access = true
  fargate_profiles = [
    {
      name = "node-group-1"
      selectors = [{
        namespace = "default"
      }]
    },
  ]
}


```
Example for deploying EKS with two ec2 on demand nodes with taints.

```hcl
data "aws_availability_zones" "available" {
  state = "available"
}

module "vpc" {
  source             = "github.com/terraform-aws-modules/terraform-aws-vpc"
  name               = var.vpc_name
  cidr               = var.cidr_block
  azs                = slice(data.aws_availability_zones.available.names, 0, 3)
  public_subnets     = [cidrsubnet(var.cidr_block, 8, 3), cidrsubnet(var.cidr_block, 8, 4), cidrsubnet(var.cidr_block, 8, 5)]
  private_subnets    = [cidrsubnet(var.cidr_block, 8, 0), cidrsubnet(var.cidr_block, 8, 1), cidrsubnet(var.cidr_block, 8, 2)]
  enable_nat_gateway = true
  single_nat_gateway = true
  public_subnet_tags = {
    "kubernetes.io/cluster/demo-cluster" = "shared"
    "kubernetes.io/role/elb"             = 1
  }
  private_subnet_tags = {
    "kubernetes.io/cluster/demo-cluster" = "shared"
    "kubernetes.io/role/internal-elb"    = 1
  }
}

module "eks" {
  source                         = "github.com/terraform-aws-modules/terraform-aws-eks"
  cluster_name                   = "demo-cluster"
  vpc_id                         = module.vpc.vpc_id
  subnet_ids                     = module.vpc.private_subnets
  control_plane_subnet_ids       = module.vpc.public_subnets
  create_kms_key                 = true
  cluster_endpoint_public_access = true
  eks_managed_node_groups = {
    one = {
      name = "node-group-1"
      taints = [
        {
          key    = "dedicatted"
          value  = "yes"
          effect = "NO_SCHEDULE"
        }
      ]
      instance_types = ["t3.micro"]

      min_size     = 1
      max_size     = 1
      desired_size = 1
    }
    two = {
      name = "node-group-2"
      taints = [
        {
          key    = "dedicatted"
          value  = "no"
          effect = "NO_EXECUTE"
        }
      ]
      instance_types = ["t3.micro"]

      min_size     = 1
      max_size     = 1
      desired_size = 1
    }
  }
}
```