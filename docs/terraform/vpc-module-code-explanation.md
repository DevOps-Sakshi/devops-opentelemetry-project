# VPC Module – Code Explanation

This module creates a VPC, subnets, Internet Gateway, NAT gateways, and route tables for deploying EKS or other AWS workloads.

## 1. Creating the VPC
''' hcl
  resource "aws_vpc" "main" {
    cidr_block           = var.vpc_cidr
    enable_dns_hostnames = true
    enable_dns_support   = true
    
    tags = {
      Name = "${var.cluster_name}-vpc"
      "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    }
  }

- Creates a VPC with the CIDR block specified in vpc_cidr.
- Enables DNS hostnames and DNS support for Kubernetes and other services.
- Tags the VPC with a name and Kubernetes cluster reference.

## 2. Creating Private Subnets
resource "aws_subnet" "private" {
  count             = length(var.private_subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.private_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]
  
  tags = {
    Name                                        = "${var.cluster_name}-private-${count.index + 1}"
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    "kubernetes.io/role/internal-elb"          = "1"
  }
}

- Creates private subnets based on private_subnet_cidrs.
- Places subnets in specified availability zones.
- Tags subnets for Kubernetes internal use (internal-elb).

## 3. Creating Public Subnets
resource "aws_subnet" "public" {
  count             = length(var.public_subnet_cidrs)
  vpc_id            = aws_vpc.main.id
  cidr_block        = var.public_subnet_cidrs[count.index]
  availability_zone = var.availability_zones[count.index]

  map_public_ip_on_launch = true

  tags = {
    Name                                        = "${var.cluster_name}-public-${count.index + 1}"
    "kubernetes.io/cluster/${var.cluster_name}" = "shared"
    "kubernetes.io/role/internal-elb"          = "1"
  }
}

- Creates public subnets for internet-facing resources.
- Automatically assigns public IPs to instances.
- Tags subnets for Kubernetes and external load balancer usage.

## 4. Creating Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "${var.cluster_name}-igw"
  }
}

- Attaches an Internet Gateway to the VPC.
- Provides internet access for public subnets.

## 5. Creating Elastic IPs for NAT Gateways
resource "aws_eip" "nat" {
  count  = length(var.public_subnet_cidrs)
  domain = "vpc"

  tags = {
    Name = "${var.cluster_name}-nat-${count.index + 1}"
  }
}

- Allocates Elastic IPs for NAT gateways in public subnets.
- Allows private subnets to access the internet securely.

## 6. Creating NAT Gateways
resource "aws_nat_gateway" "main" {
  count         = length(var.public_subnet_cidrs)
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id

  tags = {
    Name = "${var.cluster_name}-nat-${count.index + 1}"
  }  
}

- Creates NAT Gateways in each public subnet.
- Enables private subnet traffic to reach the internet.

## 7. Creating Public Route Table
resource "aws_route_table" "public" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main.id
  }

  tags = {
    Name = "${var.cluster_name}-public"
  }
}

- Creates a route table for public subnets.
- Routes all traffic (0.0.0.0/0) through the Internet Gateway.

## 8. Creating Private Route Tables
resource "aws_route_table" "private" {
  count  = length(var.private_subnet_cidrs)
  vpc_id = aws_vpc.main.id

  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_internet_gateway.main[count.index].id
  }

  tags = {
    Name = "${var.cluster_name}-private-${count.index + 1}"
  }
}

- Creates route tables for private subnets.
- Routes outbound traffic through NAT gateways for internet access.

## 9. Associating Route Tables with Subnets
resource "aws_route_table_association" "private" {
  count          = length(var.private_subnet_cidrs)
  subnet_id      = aws_subnet.private[count.index].id
  route_table_id = aws_route_table.private[count.index].id
}

resource "aws_route_table_association" "public" {
  count          = length(var.public_subnet_cidrs)
  subnet_id      = aws_subnet.public[count.index].id
  route_table_id = aws_route_table.public.id
}

- Associates private route tables with private subnets.
- Associates public route table with all public subnets.
- Ensures proper routing for internet and NAT access.