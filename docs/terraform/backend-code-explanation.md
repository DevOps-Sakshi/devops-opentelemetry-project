Terraform Backend – Explanation

This Terraform configuration sets up state file management for a project, using AWS S3 for storing the state and DynamoDB for locking. Proper state management ensures safe, consistent, and concurrent Terraform deployments.

🧱 Provider Configuration
provider "aws" {
  region = "us-west-2"
}


Configures the AWS provider for Terraform.

All resources created will be in the us-west-2 region.

Essential to allow Terraform to interact with AWS services.

🧱 S3 Bucket for Terraform State
resource "aws_s3_bucket" "terraform_state" {
  bucket = "demo-terraform-eks-state-bucket"

  lifecycle {
    prevent_destroy = false
  }
}


Creates an S3 bucket to store Terraform state files.

Bucket name: demo-terraform-eks-state-bucket.

Lifecycle block:

prevent_destroy = false allows the bucket to be destroyed if needed (default is safe to destroy).

Purpose: Centralized and durable storage for Terraform state.

Benefits:

Multiple team members can work on the same infrastructure.

Keeps state safe from local machine loss.

🧱 DynamoDB Table for Terraform Locking
resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-eks-state-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }
}


Creates a DynamoDB table for state locking.

Table name: terraform-eks-state-locks.

Billing mode: PAY_PER_REQUEST → you pay only for what you use.

Hash key: LockID is the primary key for locking entries.

Purpose: Prevents concurrent Terraform operations on the same state file.

Benefits:

Avoids race conditions or conflicts in multi-user setups.

Ensures safe, atomic Terraform runs.