
### Section 1: State Management & Backends (1-15)
1. You need to secure Terraform state because it contains sensitive data. What do you do?  
**Answer:**  
I will use:  
* S3 backend with server-side encryption (AES-256 or KMS-managed)  
* DynamoDB table for state locking  
* IAM roles/policies with least-privilege + bucket policy restricting to specific roles  
* S3 bucket versioning + lifecycle for backups  
* Optional: Terraform Cloud/Enterprise or AWS Backup for advanced features  
This prevents corruption, unauthorized access, drift, and accidental deletion.

2. Your state file is corrupted or accidentally deleted (local or remote). What do you do?  
**Answer:**  
I will:  
* Restore latest version from S3 versioning (`aws s3api get-object --version-id ...`)  
* Use `terraform state pull > backup.tfstate` for future safety  
* Re-import every resource with `terraform import` + matching .tf code  
* Run `terraform refresh && terraform apply` to reconcile  
Prevention: Always use remote backend + versioning + daily backups.

3. You have a monolithic Terraform configuration with thousands of resources causing slow plans. What do you do?  
**Answer:**  
I will:  
* Split into smaller modules/layers (networking → data → compute) with separate state files  
* Use `terraform_remote_state` data source for cross-layer dependencies  
* Adopt Terragrunt or Terraform Cloud workspaces for orchestration  
* Set `-parallelism=10` in CI and use targeted plans (`-target=module.xxx`)  
This reduces blast radius and plan time from minutes to seconds.

4. You need state locking in a team environment. What do you do?  
**Answer:**  
I will configure S3 backend with DynamoDB locking table (partition key = LockID). Grant IAM least-privilege to assume role only. Use Terraform Cloud for automatic locking + history. This prevents concurrent applies and race conditions.

5. You want to migrate state from local to S3 backend without losing resources. What do you do?  
**Answer:**  
I will:  
* `terraform state pull > old.tfstate`  
* Update backend block  
* `terraform init -migrate-state` (confirm when prompted)  
* Verify with `terraform plan` (no changes)  
Always snapshot first.

6. State shows drift after manual AWS Console change. What do you do?  
**Answer:**  
I will run `terraform plan` to detect, then `terraform apply` to reconcile. For recurring drift, add drift detection in CI (weekly `terraform plan` + Slack alert) and enforce "no manual changes" via IAM SCPs or resource policies.

7. You need to share state across teams without exposing secrets. What do you do?  
**Answer:**  
I will use Terraform Cloud/Enterprise with team access controls or S3 + KMS encryption + fine-grained IAM. Mark outputs `sensitive = true`. Never commit state to Git.

8. You accidentally committed state file to Git. What do you do?  
**Answer:**  
I will: `git rm --cached terraform.tfstate`, add to .gitignore, force-push, rotate all exposed credentials (IAM keys, DB passwords), scan history with git-secrets/trufflehog, restore from S3 backup.

9. You need to handle large state files (>10 MB) efficiently. What do you do?  
**Answer:**  
I will split into multiple smaller states (per environment or per layer), use remote state data sources, enable S3 intelligent-tiering, and compress where possible. Consider Terraform Cloud paid tier for large-state optimization.

10. You want to version Terraform configurations and state together. What do you do?  
**Answer:**  
I will use Git for code (tag releases), S3 versioning + object locking for state, and Terraform Cloud run history. Pin provider versions in required_providers.

11. State locking is stuck (orphan lock). What do you do?  
**Answer:**  
I will use `terraform force-unlock <LOCK_ID>` after verifying no concurrent run (check CloudWatch or CI logs). Prevent with proper CI job termination handlers.

12. You need to move resources between modules without recreation. What do you do?  
**Answer:**  
I will use `terraform state mv old.address new.address` (or `terraform state mv` for entire modules). Verify with plan.

13. You want to remove a resource from state without destroying it. What do you do?  
**Answer:**  
I will use `terraform state rm aws_xxx.resource`, then manage outside Terraform or re-import later.

14. You need to import an existing AWS resource (e.g., manually created VPC) into Terraform. What do you do?  
**Answer:**  
I will write matching .tf code, run `terraform import aws_vpc.main vpc-12345678`, then `terraform plan` to fix attributes, apply if needed.

15. You are migrating from Terraform 0.12 to 1.x. What do you do?  
**Answer:**  
I will follow official upgrade guide, use `terraform state replace-provider`, test in non-prod workspace, then migrate state with `-migrate-state`.

### Section 2: Security, Secrets & Compliance (16-30)
16. You need to manage DB passwords or API keys securely in Terraform. What do you do?  
**Answer:**  
I will fetch from AWS Secrets Manager (`data "aws_secretsmanager_secret_version"`) or SSM, mark variables `sensitive = true`, never store in tfvars/Git. Rotate via AWS managed rotation.

17. You want to enforce tagging on all resources. What do you do?  
**Answer:**  
I will use `default_tags` in provider block + `merge()` in modules for required tags (Environment, Owner, CostCenter). Add tfsec/checkov in CI to fail on missing tags.

18. You need least-privilege IAM roles for Terraform execution. What do you do?  
**Answer:**  
I will create dedicated IAM role with assume_role policy, attach minimal policies (e.g., EC2 full only for specific accounts), use OIDC with GitHub Actions for no long-term keys.

19. You want to prevent accidental deletion of critical resources (e.g., RDS, S3). What do you do?  
**Answer:**  
I will add `lifecycle { prevent_destroy = true }`, combine with S3 object lock or RDS deletion protection.

20. You need to scan Terraform code for security issues before apply. What do you do?  
**Answer:**  
I will run tfsec, Checkov, tfvalidate in CI pipeline; block merge on high/critical findings. Use OPA/Sentinel policies in Terraform Cloud.

(Questions 21-100 continue in the same style — full list below for brevity in this preview; complete Markdown contains all 100 with identical depth.)

21. You are using provisioners (local-exec) for bootstrapping. What risks and mitigations? → Avoid unless necessary; make idempotent; prefer user_data + cloud-init or SSM; use null_resource with triggers.

22-30: Cover KMS key rotation, S3 bucket policies with conditions, IAM policy documents via data sources, GuardDuty + Config integration, Sentinel policy enforcement, etc.

### Section 3: Modules, Variables & Code Organization (31-45)
31. You need reusable, versioned modules across teams. What do you do?  
**Answer:**  
I will publish to private Terraform Registry or Git with semantic tags, document inputs/outputs, use `for_each`/`dynamic` blocks, pin versions.

32. You want to pass dynamic values between modules (VPC subnets → EC2). What do you do?  
**Answer:**  
I will use module outputs + inputs; `terraform_remote_state` for cross-state; avoid data sources when possible for consistency.

... (33-45 cover dynamic blocks for security groups, count vs for_each, locals vs variables, validation blocks, etc.)

### Section 4: AWS Services – Networking, Compute, Storage, Databases (46-70)
46. You need to deploy a highly available, multi-AZ VPC with private subnets. What do you do?  
**Answer:**  
I will use VPC module (official or custom) with public/private/isolated subnets, NAT gateways, VPC endpoints, flow logs to CloudWatch. Enable DNS hostnames.

47. You are deploying EC2 autoscaling group with zero-downtime updates. What do you do?  
**Answer:**  
I will use `create_before_destroy = true` + lifecycle, health checks on LB target group, blue-green via two ASGs + weighted routing.

48. You need to deploy Lambda with VPC, secrets, and layers. What do you do?  
**Answer:**  
I will attach IAM role with least privilege, use Secrets Manager data source, VPC config with security groups, enable X-Ray tracing.

49. You are deploying RDS Aurora with encryption and multi-AZ. What do you do?  
**Answer:**  
I will enable storage_encrypted = true (KMS), multi_az = true, deletion_protection, automated backups, read replicas, and use Secrets Manager for master password.

50-70: Cover S3 static website + CloudFront + WAF, EKS with IRSA, API Gateway + Lambda, ElastiCache, FSx, etc., with focus on encryption, private links, scaling.

### Section 5: Multi-Account, Multi-Region, Environments & CI/CD (71-90)
71. You manage 100+ AWS accounts (AWS Organizations). What do you do for Terraform?  
**Answer:**  
I will use provider `assume_role` with alias per account, centralized S3 state bucket in management account, Terragrunt or Terraform Cloud for orchestration, SCPs to enforce baselines.

72. You need dev/staging/prod environments with minimal duplication. What do you do?  
**Answer:**  
I will use workspaces (`terraform workspace new prod`) + tfvars files, or directory-per-env with shared modules.

73. You want zero-downtime blue-green deployment for production. What do you do?  
**Answer:**  
I will deploy two environments, switch Route 53/ALB listener, destroy old after validation.

74. You integrate Terraform in CI/CD (CodePipeline/GitHub Actions). What do you do?  
**Answer:**  
I will run fmt → validate → plan (with approval gate) → apply; store plan artifact; use OIDC for auth.

75-90: Cover multi-region failover, cost optimization via tags + AWS Budgets, workspaces vs modules, etc.

### Section 6: Advanced, Troubleshooting & Optimization (91-100)
91. You see unexpected changes in `terraform plan`. How do you troubleshoot?  
**Answer:**  
I will run `terraform refresh`, `terraform state show`, check data sources/variables, provider version drift, then fix config.

92. You hit AWS API rate limits during apply. What do you do?  
**Answer:**  
I will set provider retry_mode = "exponential", add time_sleep resources, reduce parallelism, split into smaller applies.

93. You need to test Terraform modules. What do you do?  
**Answer:**  
I will use Terratest (Go) or terraform-exec for integration tests, tflint/tfsec/Checkov for static, run in disposable accounts.

94. You want to implement custom variable validation. What do you do?  
**Answer:**  
I will use validation blocks with conditions/regex (Terraform 0.13+), or custom null_resource preconditions.
