
1. **You need to secure Terraform state because it contains sensitive data. What do you do?**  
**Answer:**  
I will use:  
* S3 backend with server-side encryption (AES-256 or KMS-managed)  
* DynamoDB table for state locking  
* IAM roles/policies with least-privilege + bucket policy restricting to specific roles  
* S3 bucket versioning + lifecycle for backups  
* Optional: Terraform Cloud/Enterprise or AWS Backup for advanced features  
This prevents corruption, unauthorized access, drift, and accidental deletion.

2. **Your state file is corrupted or accidentally deleted (local or remote). What do you do?**  
**Answer:**  
I will:  
* Restore latest version from S3 versioning (`aws s3api get-object --version-id ...`)  
* Use `terraform state pull > backup.tfstate` for future safety  
* Re-import every resource with `terraform import` + matching .tf code  
* Run `terraform refresh && terraform apply` to reconcile  
Prevention: Always use remote backend + versioning + daily backups.

3. **You have a monolithic Terraform configuration with thousands of resources causing slow plans. What do you do?**  
**Answer:**  
I will:  
* Split into smaller modules/layers (networking → data → compute) with separate state files  
* Use `terraform_remote_state` data source for cross-layer dependencies  
* Adopt Terragrunt or Terraform Cloud workspaces for orchestration  
* Set `-parallelism=10` in CI and use targeted plans (`-target=module.xxx`)  
This reduces blast radius and plan time from minutes to seconds.

4. **You need state locking in a team environment. What do you do?**  
**Answer:**  
I will configure S3 backend with DynamoDB locking table (partition key = LockID). Grant IAM least-privilege to assume role only. Use Terraform Cloud for automatic locking + history. This prevents concurrent applies and race conditions.

5. **You want to migrate state from local to S3 backend without losing resources. What do you do?**  
**Answer:**  
I will:  
* `terraform state pull > old.tfstate`  
* Update backend block  
* `terraform init -migrate-state` (confirm when prompted)  
* Verify with `terraform plan` (no changes)  
Always snapshot first.

6. **State shows drift after manual AWS Console change. What do you do?**  
**Answer:**  
I will run `terraform plan` to detect, then `terraform apply` to reconcile. For recurring drift, add drift detection in CI (weekly `terraform plan` + Slack alert) and enforce "no manual changes" via IAM SCPs or resource policies.

7. **You need to share state across teams without exposing secrets. What do you do?**  
**Answer:**  
I will use Terraform Cloud/Enterprise with team access controls or S3 + KMS encryption + fine-grained IAM. Mark outputs `sensitive = true`. Never commit state to Git.

8. **You accidentally committed state file to Git. What do you do?**  
**Answer:**  
I will: `git rm --cached terraform.tfstate`, add to .gitignore, force-push, rotate all exposed credentials (IAM keys, DB passwords), scan history with git-secrets/trufflehog, restore from S3 backup.

9. **You need to handle large state files (>10 MB) efficiently. What do you do?**  
**Answer:**  
I will split into multiple smaller states (per environment or per layer), use remote state data sources, enable S3 intelligent-tiering, and compress where possible. Consider Terraform Cloud paid tier for large-state optimization.

10. **You want to version Terraform configurations and state together. What do you do?**  
**Answer:**  
I will use Git for code (tag releases), S3 versioning + object locking for state, and Terraform Cloud run history. Pin provider versions in required_providers.

11. **State locking is stuck (orphan lock). What do you do?**  
**Answer:**  
I will use `terraform force-unlock <LOCK_ID>` after verifying no concurrent run (check CloudWatch or CI logs). Prevent with proper CI job termination handlers.

12. **You need to move resources between modules without recreation. What do you do?**  
**Answer:**  
I will use `terraform state mv old.address new.address` (or `terraform state mv` for entire modules). Verify with plan.

13. **You want to remove a resource from state without destroying it. What do you do?**  
**Answer:**  
I will use `terraform state rm aws_xxx.resource`, then manage outside Terraform or re-import later.

14. **You need to import an existing AWS resource (e.g., manually created VPC) into Terraform. What do you do?**  
**Answer:**  
I will write matching .tf code, run `terraform import aws_vpc.main vpc-12345678`, then `terraform plan` to fix attributes, apply if needed.

15. **You are migrating from Terraform 0.12 to 1.x. What do you do?**  
**Answer:**  
I will follow official upgrade guide, use `terraform state replace-provider`, test in non-prod workspace, then migrate state with `-migrate-state`.

16. **You need to manage DB passwords or API keys securely in Terraform. What do you do?**  
**Answer:**  
I will fetch from AWS Secrets Manager (`data "aws_secretsmanager_secret_version"`) or SSM, mark variables `sensitive = true`, never store in tfvars/Git. Rotate via AWS managed rotation.

17. **You want to enforce tagging on all resources. What do you do?**  
**Answer:**  
I will use `default_tags` in provider block + `merge()` in modules for required tags (Environment, Owner, CostCenter). Add tfsec/checkov in CI to fail on missing tags.

18. **You need least-privilege IAM roles for Terraform execution. What do you do?**  
**Answer:**  
I will create dedicated IAM role with assume_role policy, attach minimal policies (e.g., EC2 full only for specific accounts), use OIDC with GitHub Actions for no long-term keys.

19. **You want to prevent accidental deletion of critical resources (e.g., RDS, S3). What do you do?**  
**Answer:**  
I will add `lifecycle { prevent_destroy = true }`, combine with S3 object lock or RDS deletion protection.

20. **You need to scan Terraform code for security issues before apply. What do you do?**  
**Answer:**  
I will run tfsec, Checkov, tfvalidate in CI pipeline; block merge on high/critical findings. Use OPA/Sentinel policies in Terraform Cloud.

21. **You need reusable, versioned modules across teams. What do you do?**  
**Answer:**  
I will publish to private Terraform Registry or Git with semantic tags, document inputs/outputs, use `for_each`/`dynamic` blocks, pin versions.

22. **You want to pass dynamic values between modules (VPC subnets → EC2). What do you do?**  
**Answer:**  
I will use module outputs + inputs; `terraform_remote_state` for cross-state; avoid data sources when possible for consistency.

23. **You need to deploy a highly available, multi-AZ VPC with private subnets. What do you do?**  
**Answer:**  
I will use VPC module (official or custom) with public/private/isolated subnets, NAT gateways, VPC endpoints, flow logs to CloudWatch. Enable DNS hostnames.

24. **You are deploying EC2 autoscaling group with zero-downtime updates. What do you do?**  
**Answer:**  
I will use `create_before_destroy = true` + lifecycle, health checks on LB target group, blue-green via two ASGs + weighted routing.

25. **You need to deploy Lambda with VPC, secrets, and layers. What do you do?**  
**Answer:**  
I will attach IAM role with least privilege, use Secrets Manager data source, VPC config with security groups, enable X-Ray tracing.

26. **You are deploying RDS Aurora with encryption and multi-AZ. What do you do?**  
**Answer:**  
I will enable storage_encrypted = true (KMS), multi_az = true, deletion_protection, automated backups, read replicas, and use Secrets Manager for master password.

27. **You manage 100+ AWS accounts (AWS Organizations). What do you do for Terraform?**  
**Answer:**  
I will use provider `assume_role` with alias per account, centralized S3 state bucket in management account, Terragrunt or Terraform Cloud for orchestration, SCPs to enforce baselines.

28. **You need dev/staging/prod environments with minimal duplication. What do you do?**  
**Answer:**  
I will use workspaces (`terraform workspace new prod`) + tfvars files, or directory-per-env with shared modules.

29. **You want zero-downtime blue-green deployment for production. What do you do?**  
**Answer:**  
I will deploy two environments, switch Route 53/ALB listener, destroy old after validation.

30. **You integrate Terraform in CI/CD (CodePipeline/GitHub Actions). What do you do?**  
**Answer:**  
I will run fmt → validate → plan (with approval gate) → apply; store plan artifact; use OIDC for auth.

31. **You see unexpected changes in `terraform plan`. How do you troubleshoot?**  
**Answer:**  
I will run `terraform refresh`, `terraform state show`, check data sources/variables, provider version drift, then fix config.

32. **You hit AWS API rate limits during apply. What do you do?**  
**Answer:**  
I will set provider retry_mode = "exponential", add time_sleep resources, reduce parallelism, split into smaller applies.

33. **You need to test Terraform modules. What do you do?**  
**Answer:**  
I will use Terratest (Go) or terraform-exec for integration tests, tflint/tfsec/Checkov for static, run in disposable accounts.

34. **You want to implement custom variable validation. What do you do?**  
**Answer:**  
I will use validation blocks with conditions/regex (Terraform 0.13+), or custom null_resource preconditions.

35. **How do you handle or prevent manual modification done outside your Terraform code?**

**Answer:**
- Restrict Console & CLI Write Access:
  - Developers get ReadOnlyAccess for troubleshooting.
  - Use Service Control Policies (SCPs) in AWS Organizations to enforce guardrails.
  - Use AWS Config or IAM policies to deny resource creation unless it has a tag like ManagedBy: Terraform
- Terraform Cloud/Enterprise Features:
  - Use remote state locking to prevent concurrent modifications
  - Implement policy-as-code with Sentinel or OPA
  - Set up workspace permissions and approval workflows
- Terraform Drift Detection:
  - Run terraform plan regularly to detect differences between your state and actual infrastructure
  - Use terraform refresh to update state file with current resource status
  - Implement automated drift detection in CI/CD pipelines (daily or after each manual change window)
- AWS Config Rules:
  - Set up AWS Config to track resource configuration changes
  - Create custom rules to alert on modifications to Terraform-managed resources
  - Use Config's timeline feature to see who changed what and when

36. **How do you manage multiple environments (dev, staging, QA, prod, etc.)**

**Answer:**

Separate folders for each environment, shared modules.
```
infra/
├── modules/
│   └── vpc/
│       ├── main.tf
│       ├── variables.tf
│   ├── ec2/
│   └── rds/
├── envs/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   ├── staging/
│   │   └── ...
│   └── prod/
│       └── ...
```
Each env calls the same modules with different values.

**modules/vpc/variables.tf**
```
variable "vpc_cidr" {
  type = string
}
```

**modules/vpc/main.tf**
```
resource "aws_vpc" "this" {
  cidr_block = var.vpc_cidr

  tags = {
    Name = "example-vpc"
  }
}
```

**envs/dev/main.tf**
```
provider "aws" {
  region = "us-east-1"
}

module "vpc" {
  source    = "../../modules/vpc"
  vpc_cidr  = var.vpc_cidr
}
```

**envs/dev/terraform.tfvars**
```
vpc_cidr = "10.0.0.0/16"
```

Each environment (dev, staging, prod) should have its own isolated state, usually stored in an S3 backend with a DynamoDB lock table.
Example: **envs/dev/backend.tf**
```
terraform {
  backend "s3" {
    bucket         = "my-tf-state-bucket"
    key            = "dev/terraform.tfstate"
    region          = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```
For staging (**envs/staging/backend.tf**):
```
terraform {
  backend "s3" {
    bucket         = "my-tf-state-bucket"
    key            = "staging/terraform.tfstate"
    region          = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```
From each env folder:

**DEV:**
```
cd infra/envs/dev
terraform init
terraform apply
```
**STAGING:**
```
cd infra/envs/staging
terraform init
terraform apply
```

Each terraform init downloads its own remote state file.

37. **How do you manage Terraform code for multi-account or multi-environment deployments?**

I use a modular architecture with reusable modules and separate environment folders (e.g., dev, stage, prod).
Each environment has its own backend state and variables.
Access to accounts is handled via assume-role, and state is stored in remote backends like S3 + DynamoDB for locking.
CI/CD pipelines deploy per environment.

38. **What strategies do you use to avoid Terraform state file corruption?**

- Always use a remote backend (S3, GCS, Terraform Cloud).
- Enable state locking (DynamoDB table, GCS locking, etc.).
- Avoid manual edits to state.
- Use terraform import, not manual state hacking.
- Restrict state access with IAM and encryption.

39. **How do you handle Terraform drift detection and remediation?**
`terraform plan -detailed-exitcode`
in CI.
`Exit code 2` indicates drift.
Auto-remediation options:
- Push desired state from Git (preferred GitOps approach).
- Sometimes I fix the differences by hand. I check what changed in the cloud compared to my Terraform code, and then update either the code or the actual resources so that both match again.

40. **How do you manage secrets in Terraform?**

Best practices:
- Never store secrets in terraform.tfvars or code
- Use data sources such as:
  - AWS SSM Parameter Store (secure string)
  - AWS Secrets Manager
  - Vault
- CI/CD injects runtime secrets
- Enable encryption at rest in backends
- Use `sensitive = true` on variables + outputs

41. **What is the purpose of the lifecycle block in a Terraform resource?**

It controls how Terraform manages resources during creation, updates, and deletion.
```
resource "aws_instance" "example" {
  ami           = "ami-123456"
  instance_type = "t2.micro"

  lifecycle {
    create_before_destroy = true
    prevent_destroy       = true
    ignore_changes        = [tags]
  }
}
```
- `create_before_destroy`: Creates the new resource first, then deletes the old one (Helps achieve zero downtime)
- `prevent_destroy`: Prevents accidental deletion of a resource. Terraform will throw an error if destruction is attempted (Databases, production resources)
- `ignore_changes`: Tells Terraform to ignore specific attribute changes - useful when attributes are modified outside Terraform
  - Problem: Urgent manual changes made in production
  - Example: Changing instance size or config temporarily
```
lifecycle {
  ignore_changes = [instance_type]
}
```
👉 Keeps Terraform from reverting emergency fixes

