EKS Module – Code Explanation
1. Creating IAM Role for EKS Cluster
resource "aws_iam_role" "cluster" {
  name = "${var.cluster_name}-cluster-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "eks.amazonaws.com"
      }
    }]
  })
}


Creates an IAM role for the EKS cluster itself.

Allows Amazon EKS to assume this role to manage cluster operations.

2. Attaching IAM Policy to Cluster Role
resource "aws_iam_role_policy_attachment" "cluster_policy" {
  policy_arn = "arn:aws:iam::aws:policy/AmazonEKSClusterPolicy"
  role       = aws_iam_role.cluster.name
}


Attaches the AmazonEKSClusterPolicy to the cluster role.

Grants necessary permissions for EKS to create and manage resources within AWS.

3. Creating the EKS Cluster
resource "aws_eks_cluster" "main" {
  name     = var.cluster_name
  version  = var.cluster_version
  role_arn = aws_iam_role.cluster.arn

  vpc_config {
    subnet_ids = var.subnet_ids
  }

  depends_on = [
    aws_iam_role_policy_attachment.cluster_policy
  ]
}


Deploys the EKS cluster with the specified:

cluster_name (from variables)

cluster_version (Kubernetes version)

role_arn (IAM role for the cluster)

Associates the cluster with VPC subnets for networking.

depends_on ensures the IAM policy is attached before cluster creation.

4. Creating IAM Role for Node Group
resource "aws_iam_role" "node" {
  name = "${var.cluster_name}-node-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
  })
}


Creates an IAM role for EKS worker nodes.

Allows EC2 instances to assume this role.

5. Attaching IAM Policies to Node Role
resource "aws_iam_role_policy_attachment" "node_policy" {
  for_each = toset([
    "arn:aws:iam::aws:policy/AmazonEKSWorkerNodePolicy",
    "arn:aws:iam::aws:policy/AmazonEKS_CNI_Policy",
    "arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryReadOnly"
  ])

  policy_arn = each.value
  role       = aws_iam_role.node.name
}


Attaches three AWS managed policies to the worker node role:

AmazonEKSWorkerNodePolicy – Allows nodes to join the EKS cluster.

AmazonEKS_CNI_Policy – Ensures container networking works correctly.

AmazonEC2ContainerRegistryReadOnly – Grants read access to pull container images from ECR.

6. Creating EKS Node Group
resource "aws_eks_node_group" "main" {
  for_each = var.node_groups

  cluster_name    = aws_eks_cluster.main.name
  node_group_name = each.key
  node_role_arn   = aws_iam_role.node.arn
  subnet_ids      = var.subnet_ids

  instance_types = each.value.instance_types
  capacity_type  = each.value.capacity_type

  scaling_config {
    desired_size = each.value.scaling_config.desired_size
    max_size     = each.value.scaling_config.max_size
    min_size     = each.value.scaling_config.min_size
  }

  depends_on = [
    aws_iam_role_policy_attachment.node_policy
  ]
}


Creates EKS worker nodes within the cluster.

Uses the node IAM role for permissions.

Assigns nodes to the specified subnets.

Configurable instance types and capacity type (On-Demand or Spot).

Auto-scaling is configured with desired_size, min_size, and max_size.

depends_on ensures IAM policies are applied before node creation.