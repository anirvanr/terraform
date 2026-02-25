### What language does Terraform use?
HashiCorp Configuration Language (HCL)
### What are providers in Terraform? Give examples.
Providers are plugins to interact with cloud platforms. Example: AWS, Azure, Google Cloud.
### What is a Terraform state file?
It’s a file (terraform.tfstate) that keeps track of the infrastructure resources managed by Terraform, mapping real-world resources to your configuration.
### Explain local vs remote state.
Local state is on your machine; remote state is stored in S3, Terraform Cloud, or Consul.
### What is the purpose of terraform state list ?
Lists all resources tracked in the state file.
### How do you lock a remote state?
Use backend with locking, e.g., S3 with DynamoDB.
### What is state locking, and why is it important?
Prevents simultaneous updates that could corrupt state.
### What backends does Terraform support?
S3, Consul, Terraform Cloud, Azure Storage, GCS.
### How do you migrate Terraform state from local to remote?
Configure backend and run terraform init -migrate-state .
### Explain terraform refresh .
Updates state file with real-world resource values.
### What is drift in Terraform state?
Differences between real infrastructure and state file.
### How do you resolve state file corruption?
Restore from backup or use terraform state commands to fix.
### What is a Terraform resource?
A resource represents infrastructure components like EC2 instances, S3 buckets, or security groups.
### What is a Terraform module?
A container of multiple resources used together.
### What is the difference between terraform plan and terraform apply ? 
plan previews changes; apply applies changes to infrastructure.
### What is terraform init used for?
Initializes a working directory, downloads providers, and sets up backend.
### How does terraform destroy work?
Deletes all resources defined in the configuration.
### What is the Terraform registry?
A repository for public and private modules and providers.
### What is the terraform validate command used for?
Checks syntax and configuration for correctness.
### What is terraform.tfvars?
A file to define variable values.
### What is a Terraform workspace?
A Terraform workspace is a feature that allows you to maintain multiple state files within the same Terraform configuration.

List workspaces: `terraform workspace list`

Create a new workspace: `terraform workspace new dev`

Switch to a workspace: `terraform workspace select dev`

Show the current workspace: `terraform workspace show`
### What is a backend in Terraform?
A backend defines where Terraform’s state is stored, such as locally or remotely (e.g., in AWS S3, Azure Blob Storage).
### What is the purpose of the terraform import command?
It brings existing infrastructure into Terraform management by importing resources into the state file.
### What is a data source?
A data source in Terraform is a way to query and retrieve information from outside Terraform—such as from a cloud provider, another Terraform state, or an external service—without creating or modifying any resources.
### Is every Terraform configuration a module?
Yes. Even if you write one .tf file, it is still a root module.
### Difference between root module and child module.
Root module is main config; child modules are called by root.
```
my-project/                  ← Root Module (run terraform here)
├── main.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
└── modules/
    └── vpc/                 ← Child Module
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```
Root Module (main.tf)
```
module "vpc" {                  # ← This makes "vpc" a child module
  source = "./modules/vpc"      # Local path
```
### What is the public Terraform Module Registry?
Repository for ready-made modules maintained by HashiCorp or community.
### What can be the source of a Terraform module?
A module can come from: Local folder,GitHub,Terraform Registry,S3 bucket or storage.
### What files are commonly used inside a module?

A module usually includes:
- main.tf
- variables.tf
- outputs.tf
- providers.tf (optional)
### What is the advantage of using modules for multiple environments?
Modules let you create dev, test, and prod environments easily by reusing the same code and only changing input variables.
### What are outputs in modules?
Outputs expose module values for root module or other modules.

Inside the module (modules/vpc/outputs.tf):
```
output "vpc_id" {
  value = aws_vpc.main.id
}
```
This exposes the VPC ID to whoever calls the module. Using that output in the root module:
```
module "vpc" {
  source = "./modules/vpc"
}
```
Now you can access the output like this:
```
output "my_vpc_id" {
  value = module.vpc.vpc_id
}
```
### Where is the state stored?
- By default: In a local file called terraform.tfstate
- In teams: In remote backends like S3, Azure Storage, GCS, Terraform Cloud, etc.


# **Common Terraform State Commands**

## **1. `terraform state list`**

Shows all resources tracked in the Terraform state file.

```sh
terraform state list
```

---

## **2. `terraform state show`**

Shows detailed information about a specific resource in the state.

```sh
terraform state show aws_instance.web
```

---

## **3. `terraform state mv`**

Moves (renames) a resource inside the state file.

```sh
terraform state mv aws_s3_bucket.old aws_s3_bucket.new
```

---

## **4. `terraform state rm`**

Removes a resource from the state file **without deleting it in the cloud**.

```sh
terraform state rm aws_s3_bucket.demo
```

---

## **5. `terraform state pull`**

Downloads/prints the current state from the backend.

```sh
terraform state pull
```

---

## **6. `terraform state push`**

Uploads a local state file to the remote backend.
*(Use very carefully.)*

```sh
terraform state push terraform.tfstate
```

---

## **7. `terraform refresh`** *(older versions)*

Updates the state with real infrastructure values.

```sh
terraform refresh
```
### What is provisioner?
A Terraform provisioner is used to run scripts or commands on a resource after it is created.
Terraform has mainly two types:
- local-exec → runs commands on your local machine
```
resource "aws_instance" "web" {
  ami           = "ami-123456"
  instance_type = "t2.micro"

  provisioner "local-exec" {
    command = "echo Instance Created!"
  }
}
```
- remote-exec → runs commands on the remote server / VM
```
resource "aws_instance" "web" {
  ami           = "ami-123456"
  instance_type = "t2.micro"

  provisioner "remote-exec" {
    inline = [
      "sudo apt update",
      "sudo apt install -y nginx"
    ]
  }

  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("key.pem")
    host        = self.public_ip
  }
}
```
### What is the file provisioner?
It is used to copy files or folders from your local machine to the remote server.
```
provisioner "file" {
  source      = "app.conf"
  destination = "/etc/app.conf"
}
```
### What is the function of terraform refresh?
It updates the state file with the real infrastructure, detecting any changes made outside of Terraform.
### What is Terraform Cloud?
Terraform Cloud is a SaaS platform by HashiCorp for managing Terraform runs, state, and collaboration.
### What are Terraform Variables?
Terraform variables allow you to store values in a clean, reusable, and flexible way. Instead of hardcoding values, you put them in variables so your configuration becomes easy to change.

Variable types:

- String
- Number
- Bool
- List
- Map
- Object (advanced)
```
#################################
# 1. STRING VARIABLE
#################################
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

#################################
# 2. NUMBER VARIABLE
#################################
variable "ebs_volume_size" {
  description = "Size of EBS volume in GB"
  type        = number
  default     = 20
}

#################################
# 3. BOOLEAN VARIABLE
#################################
variable "enable_monitoring" {
  description = "Enable detailed monitoring for EC2"
  type        = bool
  default     = false
}

#################################
# 4. LIST (list of string)
#################################
variable "security_groups" {
  description = "List of Security Group IDs for EC2"
  type        = list(string)
  default     = [
    "sg-0a1b2c3d4e5f67890",
    "sg-0123456789abcdef0"
  ]
}

#################################
# 5. MAP (map of string)
#################################
variable "instance_tags" {
  description = "Tags to apply to the EC2 instance"
  type        = map(string)
  default = {
    Name        = "web-server"
    Environment = "dev"
  }
}

#################################
# 6. OBJECT VARIABLE
#################################
variable "ec2_network_config" {
  description = "EC2 network configuration"
  type = object({
    subnet_id         = string
    associate_eip     = bool
    private_ip        = string
  })

  default = {
    subnet_id     = "subnet-0abc1234def567890"
    associate_eip = true
    private_ip    = "10.0.1.10"
  }
}

#################################
# 7. LIST OF OBJECTS
#################################
variable "ebs_block_devices" {
  description = "List of EBS volumes attached to EC2"
  type = list(object({
    device_name = string
    volume_size = number
    encrypted   = bool
  }))

  default = [
    {
      device_name = "/dev/sda1"
      volume_size = 20
      encrypted   = true
    },
    {
      device_name = "/dev/sdb"
      volume_size = 30
      encrypted   = false
    }
  ]
}
```
## Difference Between list(string) and map(string)
- List (list(string))

A list is an ordered collection of values.
You access items using index numbers (0, 1, 2...).

```
variable "security_groups" {
  type = list(string)
  default = [
    "sg-1111",
    "sg-2222",
    "sg-3333"
  ]
}
```
Access example `var.security_groups[0]   # sg-1111`

- Map (map(string))

A map is a key-value pair collection.
You access values using keys.

```
variable "instance_tags" {
  type = map(string)
  default = {
    Name        = "web-server"
    Environment = "dev"
    Owner       = "teamA"
  }
}
```
Access example `var.instance_tags["Name"]      # web-server`
### How do you handle secrets in Terraform?
By using external secrets management systems like HashiCorp Vault or AWS Secrets Manager, and avoiding hardcoding secrets in code.
### What is Sentinel in Terraform?
Sentinel is a policy-as-code framework created by HashiCorp. It allows organizations to control what Terraform can and cannot do before resources are created.

Sentinel lets you write rules like:

- “Don't allow EC2 instances without tags”
- “Don't allow public S3 buckets”
- “Ensure all resources have encryption enabled”

Terraform will evaluate these rules before apply, and stop the apply if the rules fail.
Sentinel works only with: Terraform Cloud and Terraform Enterprise. It is not available in open-source Terraform CLI.
