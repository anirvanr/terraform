**1. What is Terraform, and why is it used?**

Terraform is an open-source Infrastructure as Code (IaC) tool that allows you to provision, manage, and version cloud and on-prem resources declaratively.

**2. Explain the difference between Terraform and CloudFormation.**

Terraform is multi-cloud and open-source; CloudFormation is AWS-specific. Terraform uses HCL; CloudFormation uses JSON/YAML.

**3. What are providers in Terraform? Give examples.**

Providers are plugins to interact with cloud platforms. Example: AWS, Azure, Google Cloud.

**4. What is a Terraform resource?**

A resource represents infrastructure components like EC2 instances, S3 buckets, or security groups.

**5. What is a Terraform module?**

A module is a container for multiple resources and configurations, used for reusability.

**6. What are variables in Terraform? How do you define them?**

Variables allow dynamic configuration. Defined with variable "name" {} block.

**7. What are outputs in Terraform?**

Outputs expose information about resources after deployment.

**8. Explain the Terraform state file. Why is it important?**

Stores resource mappings and metadata. Needed to track infrastructure and plan changes.

**9. What are data sources in Terraform?**

Data sources allow reading external information from providers without creating resources.

**10. What is the difference between terraform plan and terraform apply?**

plan previews changes; apply applies changes to infrastructure.

**11. What is terraform init used for?**

Initializes a working directory, downloads providers, and sets up backend.

**12. How does terraform destroy work?**

Deletes all resources defined in the configuration.

**13. Explain the purpose of terraform validate.**

Checks syntax and configuration for correctness.

**14. What is the Terraform registry?**

A repository for public and private modules and providers.

**15. How do you pass environment-specific variables in Terraform?**

Use .tfvars files or environment variables.

**16. Explain Terraform's configuration language (HCL).**

HashiCorp Configuration Language (HCL) is human-readable and used for writing Terraform configs.

**17. What is interpolation in Terraform?**

Syntax to reference variables, attributes, and outputs: ${var.name}.

**18. Can Terraform manage existing resources not created by Terraform?**

Yes, using terraform import.

**19. What is the difference between a resource and a data block?**

Resource creates infrastructure; data block reads existing information.

**20. How does Terraform handle dependencies between resources?**

Terraform automatically detects dependencies or can use depends_on explicitly.


**21. What is Terraform state?**

A JSON file storing resource metadata and mapping between code and deployed infrastructure.

**22. Explain local vs remote state.**

Local state is on your machine; remote state is stored in S3, Terraform Cloud, or Consul.

**23. What is the purpose of terraform state list?**

Lists all resources tracked in the state file.

**24. How do you lock a remote state?**

Use backend with locking, e.g., S3 with DynamoDB.

**25. What is state locking, and why is it important?**

Prevents simultaneous updates that could corrupt state.

**26. What backends does Terraform support?**

S3, Consul, Terraform Cloud, Azure Storage, GCS.

**27. How do you migrate Terraform state from local to remote?**

Configure backend and run terraform init -migrate-state.

**28. Explain terraform refresh.**

Updates state file with real-world resource values.

**29. What is drift in Terraform state?**

Differences between real infrastructure and state file.

**30. How do you resolve state file corruption?**

Restore from backup or use terraform state commands to fix.

**31. What is a workspace in Terraform?**

Workspace allows multiple states in the same configuration for environments.

**32. How do you use workspaces for multi-environment deployments?**

Use separate workspaces (dev, staging, prod) with environment-specific variables.

**33. What is terraform import and when would you use it?**

Imports existing resources into Terraform state.

**34. Can Terraform handle multiple environments with a single state file?**

Not recommended; use separate workspaces or backends.

**35. Explain the risks of manually editing a state file.**

Can corrupt state and cause resource mismanagement.


**36. What are Terraform modules, and why use them?**

Reusable containers for resources; simplify code management.

**37. Difference between root module and child module.**

Root module is main config; child modules are called by root.

**38. How do you call a module in Terraform?**

Using module "name" { source = "path_or_registry" }.

**39. What is the public Terraform Module Registry?**

Repository for reusable modules maintained by HashiCorp or community.

**40. How do you pass variables to modules?**

Use variables block and assign values in module call.

**41. What are outputs in modules?**

Outputs expose module values for root module or other modules.

**42. How can you share Terraform modules across projects?**

Store in Git repositories or private registry.

**43. How do you version modules?**

Use Git tags or registry version constraints.

**44. Explain nested modules.**

Modules calling other modules.

**45. What are some best practices for writing reusable modules?**

Use variables, outputs, proper naming, documentation.

**46. How do you use conditional logic in modules?**

Use count, for_each, or ternary expressions.

**47. How do you handle module dependencies?**

Implicit via resource references; can use depends_on if needed.

**48. Can modules be sourced from Git repositories?**

Yes, use source = "git::https://repo.git".

**49. How do you override default module variables?**

Provide values in module call or .tfvars files.

**50. Explain how modules improve collaboration in large teams.**

Encapsulate code, enforce standards, reduce conflicts, easier reviews.

**51. Explain dynamic blocks in Terraform.**

Dynamic blocks allow creating multiple nested blocks dynamically using variables or loops.

**52. What are for_each and count in Terraform?**

Both are used to create multiple resources. count is index-based; for_each uses key/value mappings.

**53. Difference between count and for_each.**

count creates resources based on a numeric value; for_each allows mapping resources to unique keys.

**54. What is a Terraform provider alias?**

Alias lets you use multiple configurations of the same provider in one configuration.

**55. How do you use locals in Terraform?**

Locals store reusable expressions or values for cleaner code.

**56. What is the purpose of depends_on?**

Forces explicit resource dependency when Terraform can't infer automatically.

**57. How do you handle secrets in Terraform?**

Use sensitive variables, AWS Secrets Manager, or SSM Parameter Store; do not store in code.

**58. Explain the terraform taint command.**

Marks a resource for destruction and recreation on the next apply.

**59. What is the terraform graph command used for?**

Generates a visual representation of resource dependencies.

**60. How do you use external data sources?**

Use external provider to read JSON output from scripts.

**61. What is Terraform interpolation syntax?**

Syntax ${} to reference variables, attributes, or outputs in strings.

**62. What is the difference between Terraform 0.11 and 0.12?**

0.12 introduced richer expressions, for_each, improved type system, and HCL2 support.

**63. How do you implement conditional resource creation?**

Use count or for_each with ternary logic based on variables.

**64. What are lifecycle blocks?**

Control resource behavior with create_before_destroy, prevent_destroy, and ignore_changes.

**65. Explain create_before_destroy and prevent_destroy.**

create_before_destroy: ensures new resource exists before destroying old. prevent_destroy: prevents accidental deletion.

**66. How can Terraform be used to manage multi-cloud infrastructure?**

Configure multiple providers, modularize code, and maintain separate states per cloud.

**67. What is remote execution in Terraform Cloud?**

Terraform Cloud runs plans and applies in a managed environment rather than locally.

**68. How do you manage sensitive outputs?**

Mark outputs as sensitive = true to hide them in CLI and logs.

**69. How can Terraform be integrated with CI/CD pipelines?**

Run terraform fmt, validate, plan, and apply in pipelines like GitHub Actions, GitLab CI, or Jenkins.

**70. Explain the terraform fmt and terraform validate commands.**

fmt formats code; validate checks for syntax correctness and logical errors.

**71. What are some common Terraform providers?**

AWS, Azure, Google Cloud, Kubernetes, Helm, Consul.

**72. How do you configure AWS provider in Terraform?**

Define provider "aws" { region = "us-east-1" } with credentials.

**73. How do you manage multiple providers in a single Terraform configuration?**

Use provider aliases and specify provider = aws.alias in resources.

**74. Explain provider versioning in Terraform.**

Specify required_providers block with version constraints to ensure compatibility.

**75. How do you use Terraform to manage Kubernetes clusters?**

Use AWS provider to create EKS and Kubernetes provider to deploy resources on the cluster.

**76. How do you integrate Terraform with AWS IAM?**

Create IAM roles/policies and assign them to resources; use provider credentials with permissions.

**77. How do you manage Azure or GCP resources with Terraform?**

Use Azure or Google Cloud providers with required credentials and resource definitions.

**78. How does Terraform interact with third-party APIs?**

Through external provider, provider plugins, or custom providers.

**79. Can Terraform manage on-prem infrastructure?**

Yes, via providers like VMware vSphere, OpenStack, or Ansible integration.

**80. How do you handle provider authentication securely?**

Use environment variables, credentials files, or secret managers; avoid storing secrets in code.

**81. What are best practices for managing Terraform state?**

Use remote state with locking, versioning, and workspace separation.

**82. How should you organize Terraform code in a repository?**

Use modules, separate environments, logical folder structure, and version control.

**83. Why should you avoid hardcoding values in Terraform?**

Hardcoding reduces flexibility; use variables for environment-specific configuration.

**84. How do you structure Terraform for multi-environment deployments?**

Use workspaces, separate state files, and environment-specific variables.

**85. What is the importance of version controlling Terraform configurations?**

Tracks changes, enables collaboration, and rollbacks.

**86. How do you test Terraform modules?**

Use terraform plan, validate, and testing frameworks like Terratest.

**87. How do you handle sensitive data in Terraform configs?**

Use sensitive flag, external secret managers, and avoid storing in VCS.

**88. What is the difference between immutable and mutable infrastructure in Terraform?**

Immutable: replace resources on change; Mutable: modify resources in place.

**89. How do you ensure Terraform deployments are idempotent?**

Terraform plans should always converge; avoid dynamic resource names causing unnecessary changes.

**90. How do you prevent accidental resource deletion?**

Use prevent_destroy lifecycle block and manual approval in pipelines.

**91. You need to deploy multiple environments (dev, staging, prod) with the same Terraform code. How do you approach it?**

Use workspaces or environment-specific .tfvars files to parameterize deployments.

**92. You notice Terraform is trying to replace a resource unnecessarily. What could be the reason and how to fix it?**

Immutable attributes changed or drift detected; fix with ignore_changes in lifecycle or correct the variable.

**93. How would you manage secrets (e.g., DB passwords) in Terraform for production environments?**

Use AWS Secrets Manager, SSM Parameter Store, or encrypted variables.

**94. You need to deploy a VPC, subnets, and EC2 instances in multiple regions. How would you structure Terraform?**

Use reusable modules for VPC, subnets, EC2; pass region-specific variables; maintain separate state per region.

**95. Your Terraform state is corrupted. How do you recover it?**

Restore from S3/DynamoDB remote state backup or manually repair state using terraform state commands.

**96. You want to deploy a Kubernetes cluster on AWS with Terraform. What provider and resources will you use?**

AWS provider for EKS cluster and node groups; Kubernetes provider for pods and services.

**97. How do you handle multi-cloud infrastructure using Terraform?**

Use multiple providers, modularize code, separate states, and manage credentials securely.

**98. You need to ensure zero-downtime deployment for infrastructure updates. How?**

Use create_before_destroy, blue/green deployments, and health-checked load balancers.

**99. Terraform is failing during a plan because of provider version conflicts. How would you resolve it?**

Specify correct required_providers versions and run terraform init -upgrade.

**100. You want to implement CI/CD for Terraform changes. How would you set it up?**

Git repo for code, CI pipeline with fmt, validate, plan, manual approval, remote state in S3/DynamoDB, apply to prod with controlled approvals.
