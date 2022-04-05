# Automating Infrastructure on Google Cloud with Terraform

## Terraform Fundamentals

### What is Terraform?

Terraform is a tool for building, changing, and versioning infrastructure safely and efficiently. 

Key features

- Infrastructure as code

- Execution plans
- Resource graph
- Change automation

### Build infrastructure

The set of files used to describe infrastructure in Terraform is simply known as a `Terraform configuration`. In this section, you will write your first configuration to launch a single VM instance.

```sh
touch instance.tf
```

```hcl
resource "google_compute_instance" "terraform" {
  project      = "<PROJECT_ID>"
  name         = "terraform"
  machine_type = "n1-standard-1"
  zone         = "us-central1-a"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
    }
  }
  network_interface {
    network = "default"
    access_config {
    }
  }
}
```

```sh
# init          Prepare your working directory for other commands
# validate      Check whether the configuration is valid
# plan          Show changes required by the current configuration
# apply         Create or update infrastructure
# show          Show the current state or a saved plan
# destroy       Destroy previously-created infrastructure

terraform init
terraform validate
terraform plan
terraform apply
yes
terraform show
terraform destroy
```

- **Navigation menu** > **Compute Engine** > **VM instances** 

## Infrastructure as Code with Terraform

A simple workflow for deployment will follow closely to the steps below:

- **Scope** - Confirm what resources need to be created for a given project.
- **Author** - Create the configuration file in HCL based on the scoped parameters.
- **Initialize** - Run `terraform init` in the project directory with the configuration files. This will download the correct provider plug-ins for the project.
- **Plan & Apply** - Run `terraform plan` to verify creation process and then `terraform apply` to create real resources as well as the state file that compares future changes in your configuration files to what actually exists in your deployment environment.

In this lab you will learn how to perform the following tasks:

- Build, change, and destroy infrastructure with Terraform
- Create Resource Dependencies with Terraform
- Provision infrastructure with Terraform

### Build Infrastructure

```
touch main.tf
```

```hcl
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
    }
  }
}
provider "google" {
  version = "3.5.0"
  project = "<PROJECT_ID>"
  region  = "us-central1"
  zone    = "us-central1-c"
}
resource "google_compute_network" "vpc_network" {
  name = "terraform-network"
}
```

```sh
terraform init
terraform apply
yes
terraform show
```

- **Navigation menu** > **VPC network**

### Change Infrastructure

#### Adding Resources

```hcl
#...
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
    }
  }
  network_interface {
    network = google_compute_network.vpc_network.name
    access_config {
    }
  }
}
```

```sh
terraform apply
yes
```

#### Changing Resources

```hcl
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"
  tags         = ["web", "dev"]
  # ...
}
```

```sh
terraform apply
yes
# The prefix ~ means that Terraform will update the resource in-place.
```

#### Destructive Changes

A destructive change is a change that requires the provider to replace the existing resource rather than updating it. 

```hcl
  boot_disk {
    initialize_params {
      image = "cos-cloud/cos-stable"
    }
  }
```

```sh
terraform apply
yes
# The prefix -/+ means that Terraform will destroy and recreate the resource, rather than updating it in-place.
```

#### Destroy Infrastructure

```sh
terraform destroy
yes
# The - prefix indicates that the instance and the network will be destroyed. 
```

### Create Resource Dependencies

In this section, you will be shown a basic example of how to configure multiple resources and how to use resource attributes to configure other resources.

```sh
terraform apply
yes
```

#### Assigning a Static IP Address

```hcl
# ...
resource "google_compute_address" "vm_static_ip" {
  name = "terraform-static-ip"
}
```

```sh
terraform plan
```

```hcl
  network_interface {
    network = google_compute_network.vpc_network.self_link
    access_config {
      nat_ip = google_compute_address.vm_static_ip.address
    }
  }
```

```sh
# Saving the plan this way ensures that you can apply exactly the same plan in the future. 
terraform plan -out static_ip

# Apply plan
terraform apply "static_ip"
```

#### Implicit and Explicit Dependencies

By studying the resource attributes used in interpolation expressions, Terraform can automatically infer when one resource depends on another. In the example above, the reference to `google_compute_address.vm_static_ip.address` creates an *implicit dependency* on the `google_compute_address` named `vm_static_ip`.

Sometimes there are dependencies between resources that are *not* visible to Terraform. The `depends_on` argument can be added to any resource and accepts a list of resources to create explicit dependencies for.

For example, perhaps an application you will run on your instance expects to use a specific Cloud Storage bucket, but that dependency is configured inside the application code and thus not visible to Terraform. In that case, you can use `depends_on` to explicitly declare the dependency.

```hcl
# New resource for the storage bucket our application will use.
resource "google_storage_bucket" "example_bucket" {
  name     = "<UNIQUE-BUCKET-NAME>"
  location = "US"
  website {
    main_page_suffix = "index.html"
    not_found_page   = "404.html"
  }
}
# Create a new instance that uses the bucket
resource "google_compute_instance" "another_instance" {
  # Tells Terraform that this VM instance must be created only after the
  # storage bucket has been created.
  depends_on = [google_storage_bucket.example_bucket]
  name         = "terraform-instance-2"
  machine_type = "f1-micro"
  boot_disk {
    initialize_params {
      image = "cos-cloud/cos-stable"
    }
  }
  network_interface {
    network = google_compute_network.vpc_network.self_link
    access_config {
    }
  }
}
```

```sh
terraform plan
terraform apply
```

### Provision Infrastructure

Google Cloud allows customers to manage their own [custom operating system images](https://cloud.google.com/compute/docs/images/create-delete-deprecate-private-images). This can be a great way to ensure the instances you provision with Terraform are pre-configured based on your needs. [Packer](https://www.packer.io/) is the perfect tool for this and includes a [builder for Google Cloud](https://www.packer.io/docs/builders/googlecompute.html).

Terraform uses provisioners to upload files, run shell scripts, or install and trigger other software like configuration management tools.

#### Defining a Provisioner

```hcl
resource "google_compute_instance" "vm_instance" {
  name         = "terraform-instance"
  machine_type = "f1-micro"
  tags         = ["web", "dev"]
  provisioner "local-exec" {
    command = "echo ${google_compute_instance.vm_instance.name}:  ${google_compute_instance.vm_instance.network_interface[0].access_config[0].nat_ip} >> ip_address.txt"
  }
  # ...
}
```

```sh
terraform apply
# Terraform found nothing to do
terraform taint google_compute_instance.vm_instance
# Terraform will remove any tainted resources and create new resources, attempting to provision them again after creation.
terraform apply
cat ip_address.txt
```

#### Destroy Provisioners

If you need to use destroy provisioners, please see the [provisioner documentation](https://www.terraform.io/docs/provisioners/).

## Interact with Terraform Modules

### What are modules for?

- **Organize configuration**
- **Encapsulate configuration**
- **Re-use configuration**
- **Provide consistency and ensure best practices**

A Terraform module is a set of Terraform configuration files in a single directory,You may have a simple set of Terraform configuration files like this:

```
$ tree minimal-module/
.
├── LICENSE
├── README.md
├── main.tf
├── variables.tf
├── outputs.tf
```

### Module best practices

- Start writing your configuration with a plan for modules. 
- Use local modules to organize and encapsulate your code.
- Use the public [Terraform Registry](https://registry.terraform.io/) to find useful modules.
- Publish and share modules with your team.

### Task 1. Use modules from the Registry

#### Create a Terraform configuration

```
git clone https://github.com/terraform-google-modules/terraform-google-network
cd terraform-google-network
git checkout tags/v3.3.0 -b v3.3.0
vi examples/simple_project/main.tf
```

#### Set values for module input variables

Some input variables are *required*, which means that the module doesn't provide a default value; an explicit value must be provided in order for Terraform to run correctly.

https://registry.terraform.io/modules/terraform-google-modules/network/google/3.3.0?tab=inputs

#### Define root input variables

```sh
vi examples/simple_project/variables.tf
```

```hcl
variable "project_id" {
  description = "The project ID to host the network in"
  default     = "FILL IN YOUR PROJECT ID HERE"
}
variable "network_name" {
  description = "The name of the VPC network being created"
  default     = "example-vpc"
}
```

#### Define root output values

Like input variables, module outputs are listed under the `outputs` tab in the [Terraform registry](https://registry.terraform.io/modules/terraform-google-modules/network/google/3.3.0?tab=outputs).

```
cat examples/simple_project/outputs.tf
```

#### Provision infrastructure

```
cd ~/terraform-google-network/examples/simple_project
terraform init
terraform apply
yes
terraform destroy
```

When using a new module for the first time, you must run either `terraform init` or `terraform get` to install the module. When either of these commands is run, Terraform will install any new modules in the `.terraform/modules` directory within your configuration's working directory. For local modules, Terraform will create a symlink to the module's directory. Because of this, any changes to local modules will be effective immediately, without your having to re-run `terraform get`.

### Task 2. Build a module

#### Module structure

```
$ tree minimal-module/
.
├── LICENSE
├── README.md
├── main.tf
├── variables.tf
├── outputs.tf
```

#### Create a module

ref.https://github.com/terraform-google-modules/terraform-google-cloud-storage/tree/master/modules/simple_bucket

```sh
cd ~
touch main.tf
mkdir -p modules/gcs-static-website-bucket
```

```sh
cd modules/gcs-static-website-bucket
```

```sh
vi README.md
```

```markdown
# GCS static website bucket
This module provisions Cloud Storage buckets configured for static website hosting.
```

```sh
vi LICENSE
```

```
Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at
    http://www.apache.org/licenses/LICENSE-2.0
Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

```sh
vi website.tf
```

```hcl
resource "google_storage_bucket" "bucket" {
  name               = var.name
  project            = var.project_id
  location           = var.location
  storage_class      = var.storage_class
  labels             = var.labels
  force_destroy      = var.force_destroy
  uniform_bucket_level_access = true
  versioning {
    enabled = var.versioning
  }
  dynamic "retention_policy" {
    for_each = var.retention_policy == null ? [] : [var.retention_policy]
    content {
      is_locked        = var.retention_policy.is_locked
      retention_period = var.retention_policy.retention_period
    }
  }
  dynamic "encryption" {
    for_each = var.encryption == null ? [] : [var.encryption]
    content {
      default_kms_key_name = var.encryption.default_kms_key_name
    }
  }
  dynamic "lifecycle_rule" {
    for_each = var.lifecycle_rules
    content {
      action {
        type          = lifecycle_rule.value.action.type
        storage_class = lookup(lifecycle_rule.value.action, "storage_class", null)
      }
      condition {
        age                   = lookup(lifecycle_rule.value.condition, "age", null)
        created_before        = lookup(lifecycle_rule.value.condition, "created_before", null)
        with_state            = lookup(lifecycle_rule.value.condition, "with_state", null)
        matches_storage_class = lookup(lifecycle_rule.value.condition, "matches_storage_class", null)
        num_newer_versions    = lookup(lifecycle_rule.value.condition, "num_newer_versions", null)
      }
    }
  }
}
```

```sh
vi variables.tf
```

```hcl
variable "name" {
  description = "The name of the bucket."
  type        = string
}
variable "project_id" {
  description = "The ID of the project to create the bucket in."
  type        = string
}
variable "location" {
  description = "The location of the bucket."
  type        = string
}
variable "storage_class" {
  description = "The Storage Class of the new bucket."
  type        = string
  default     = null
}
variable "labels" {
  description = "A set of key/value label pairs to assign to the bucket."
  type        = map(string)
  default     = null
}
variable "bucket_policy_only" {
  description = "Enables Bucket Policy Only access to a bucket."
  type        = bool
  default     = true
}
variable "versioning" {
  description = "While set to true, versioning is fully enabled for this bucket."
  type        = bool
  default     = true
}
variable "force_destroy" {
  description = "When deleting a bucket, this boolean option will delete all contained objects. If false, Terraform will fail to delete buckets which contain objects."
  type        = bool
  default     = true
}
variable "iam_members" {
  description = "The list of IAM members to grant permissions on the bucket."
  type = list(object({
    role   = string
    member = string
  }))
  default = []
}
variable "retention_policy" {
  description = "Configuration of the bucket's data retention policy for how long objects in the bucket should be retained."
  type = object({
    is_locked        = bool
    retention_period = number
  })
  default = null
}
variable "encryption" {
  description = "A Cloud KMS key that will be used to encrypt objects inserted into this bucket"
  type = object({
    default_kms_key_name = string
  })
  default = null
}
variable "lifecycle_rules" {
  description = "The bucket's Lifecycle Rules configuration."
  type = list(object({
    # Object with keys:
    # - type - The type of the action of this Lifecycle Rule. Supported values: Delete and SetStorageClass.
    # - storage_class - (Required if action type is SetStorageClass) The target Storage Class of objects affected by this Lifecycle Rule.
    action = any
    # Object with keys:
    # - age - (Optional) Minimum age of an object in days to satisfy this condition.
    # - created_before - (Optional) Creation date of an object in RFC 3339 (e.g. 2017-06-13) to satisfy this condition.
    # - with_state - (Optional) Match to live and/or archived objects. Supported values include: "LIVE", "ARCHIVED", "ANY".
    # - matches_storage_class - (Optional) Storage Class of objects to satisfy this condition. Supported values include: MULTI_REGIONAL, REGIONAL, NEARLINE, COLDLINE, STANDARD, DURABLE_REDUCED_AVAILABILITY.
    # - num_newer_versions - (Optional) Relevant only for versioned objects. The number of newer versions of an object to satisfy this condition.
    condition = any
  }))
  default = []
}
```

```sh
vi outputs.tf
```

```hcl
output "bucket" {
  description = "The created storage bucket"
  value       = google_storage_bucket.bucket
}
```

```sh
vi ~/main.tf
```

```hcl
module "gcs-static-website-bucket" {
  source = "./modules/gcs-static-website-bucket"
  name       = var.name
  project_id = var.project_id
  location   = "us-east1"
  lifecycle_rules = [{
    action = {
      type = "Delete"
    }
    condition = {
      age        = 365
      with_state = "ANY"
    }
  }]
}
```

```sh
vi ~/outputs.tf
```

```
output "bucket-name" {
  description = "Bucket names."
  value       = "module.gcs-static-website-bucket.bucket"
}
```

```sh
vi variables.tf	
```

```hcl
variable "project_id" {
  description = "The ID of the project in which to provision resources."
  type        = string
  default     = "FILL IN YOUR PROJECT ID HERE"
}
variable "name" {
  description = "Name of the buckets to create."
  type        = string
  default     = "FILL IN YOUR (UNIQUE) BUCKET NAME HERE"
}
```

```
terraform init
terraform apply
```

```sh
cd ~
curl https://raw.githubusercontent.com/hashicorp/learn-terraform-modules/master/modules/aws-s3-static-website-bucket/www/index.html > index.html
curl https://raw.githubusercontent.com/hashicorp/learn-terraform-modules/blob/master/modules/aws-s3-static-website-bucket/www/error.html > error.html
```

```sh
gsutil cp *.html gs://YOUR-BUCKET-NAME
```

<https://storage.cloud.google.com/YOUR-BUCKET-NAME/index.html>

```
terraform destroy
```

## Managing Terraform State

Terraform must store state about your managed infrastructure and configuration. This state is used by Terraform to map real-world resources to your configuration, keep track of metadata, and improve performance for large infrastructures.This state is stored by default in a local file named `terraform.tfstate`, but it can also be stored remotely, which works better in a team environment.

### Purpose of Terraform state

- Mapping to the real world
- Metadata
- Performance
- Syncing

- State locking
- Workspaces

### Working with backends

A *backend* in Terraform determines how state is loaded and how an operation such as `apply` is executed.By default, Terraform uses the "local" backend, which is the normal behavior of Terraform you're used to. Here are some of the benefits of backends:

- **Working in a team:**
- **Keeping sensitive information off disk:**
- **Remote operations:** 
- **Backends are completely optional:** 

#### Add a local backend

In this section, you will configure a local backend.

```sh
touch main.tf
gcloud config list --format 'value(core.project)'
```

```hcl
provider "google" {
  project     = "# REPLACE WITH YOUR PROJECT ID"
  region      = "us-central-1"
}
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "# REPLACE WITH YOUR PROJECT ID"
  location    = "US"
  uniform_bucket_level_access = true
}
terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}
```

```sh
terraform init
terraform apply
terraform show
```

The `init` command must be called:

- On any new environment that configures a backend
- On any change of the backend configuration (including type of backend)
- On removing backend configuration completely

#### Add a Cloud Storage backend

A Cloud Storage backend stores the state as an object in a configurable prefix in a given bucket on Cloud Storage. This backend also supports [state locking](https://www.terraform.io/docs/state/locking.html).

- replacing the local backend:

```hcl
terraform {
  backend "gcs" {
    bucket  = "# REPLACE WITH YOUR BUCKET NAME"
    prefix  = "terraform/state"
  }
}
```

```sh
terraform init -migrate-state
yes
```

#### Refresh the state

The `terraform refresh` command is used to reconcile the state Terraform knows about (via its state file) with the real-world infrastructure. This can be used to detect any drift from the last-known state and to update the state file.

This does not modify infrastructure, but does modify the state file. If the state is changed, this may cause changes to occur during the next plan or apply.

- **Navigation menu** > **Cloud Storage > Browser**
- Select the check box next to the name, and click **Show info panel**.
- **Labels** >  **Add Label**:  **Key** = `key` and **Value** = `value` > **Save**

```sh
terraform refresh
terraform show
```

#### Clean up your workspace

- Copy and replace the gcs configuration with the following:

```hcl
terraform {
  backend "local" {
    path = "terraform/state/terraform.tfstate"
  }
}
```

```
terraform init -migrate-state
```

- If you try to delete a bucket that contains objects, Terraform will fail that run.

```
resource "google_storage_bucket" "test-bucket-for-state" {
  name        = "qwiklabs-gcp-03-c26136e27648"
  location    = "US"
  uniform_bucket_level_access = true
  force_destroy = true
}
```

```sh
terraform apply
yes
terraform destroy
yes
```

### Import Terraform configuration

The default Terraform workflow involves creating and managing infrastructure entirely with Terraform.

- Write a Terraform configuration that defines the infrastructure you want to create.
- Review the Terraform plan to ensure that the configuration will result in the expected state and infrastructure.
- Apply the configuration to create your Terraform state and infrastructure.

Bringing existing infrastructure under Terraform’s control involves five main steps:

- Identify the existing infrastructure to be imported.
- Import the infrastructure into your Terraform state.
- Write a Terraform configuration that matches that infrastructure.
- Review the Terraform plan to ensure that the configuration matches the expected state and infrastructure.
- Apply the configuration to update your Terraform state.

In this section, first you will create a Docker container with the Docker CLI. Next, you will import it into a new Terraform workspace. Then you will update the container’s configuration using Terraform before finally destroying it when you are done.

#### Create a Docker container

```
docker run --name hashicorp-learn --detach --publish 8080:80 nginx:latest
docker ps
```

#### Import the container into Terraform

- Initialize your Terraform workspace

  ```
  git clone https://github.com/hashicorp/learn-terraform-import.git
  cd learn-terraform-import
  terraform init
  ```

- learn-terraform-import/main.tf

  ```hcl
  provider "docker" {
  #   host    = "npipe:////.//pipe//docker_engine"
  }
  ```

  This is a current workaround for a known issue with a Docker initialization error.

- learn-terraform-import/docker.tf

  ```hcl
  resource "docker_container" "web" {}
  ```

  Define an empty `docker_container` resource in your `docker.tf` file, which represents a Docker container with the Terraform resource ID `docker_container.web`:

- Run the following `terraform import` command to attach the existing Docker container to the `docker_container.web` resource you just created.

  ```sh
  terraform import docker_container.web $(docker inspect -f {{.ID}} hashicorp-learn)
  terraform show
  ```

#### Create configuration

```sh
terraform plan
# Terraform will show errors for the missing required arguments image and name. 
```

```sh
terraform show -no-color > docker.tf
terraform plan
# Terraform will show warnings and errors about a deprecated argument ('links'), and several read-only arguments (ip_address, network_data, gateway, ip_prefix_length, id).
```

You can now selectively remove these optional attributes.

```hcl
resource "docker_container" "web" {
    image = "sha256:c316d5a335a5cf324b0dc83b3da82d7608724769f6454f6d9a621f3ec2534a5a"
    name  = "hashicorp-learn"
    ports {
        external = 8080
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}
```

```
terraform plan
terraform apply
yes
```

#### Create image resource

In some cases, you can bring resources under Terraform's control without using the `terraform import` command. This is often the case for resources that are defined by a single unique ID or tag, such as Docker images.

```
resource "docker_image" "nginx" {
  name         = "nginx:latest"
}
```

```
terraform apply
yes
```

```
resource "docker_container" "web" {
    image = docker_image.nginx.latest
    name  = "hashicorp-learn"
    ports {
        external = 8080
        internal = 80
        ip       = "0.0.0.0"
        protocol = "tcp"
    }
}
```

```
terraform apply
yes
```

#### Manage the container with Terraform

Now that Terraform manages the Docker container, use Terraform to change the container's external port from `8080` to `8081`:

```
resource "docker_container" "web" {
  name  = "hashicorp-learn"
  image = docker_image.nginx.latest
  ports {
    external = 8081
    internal = 80
    ip       = "0.0.0.0"
    protocol = "tcp"
  }
}
```

```sh
terraform apply
yes
docker ps
```

#### Destroy infrastructure

```sh
terraform destroy
docker ps --filter "name=hashicorp-learn"
```

### Limitations and other considerations

Terraform import can only know the current state of infrastructure as reported by the Terraform provider. It does not know:

- Whether the infrastructure is working correctly.
- The intent of the infrastructure.
- Changes you've made to the infrastructure that aren't controlled by Terraform; for example, the state of a Docker container's filesystem.

Following infrastructure as code (IaC) best practices such as [immutable infrastructure](https://www.hashicorp.com/resources/what-is-mutable-vs-immutable-infrastructure) can help prevent many of these problems, but infrastructure created manually is unlikely to follow IaC best practices.

## Automating Infrastructure on Google Cloud with Terraform: Challenge Labr

ref.<https://gist.github.com/Syed-Hassaan/e41a83345832666846ee6be0f69c1f36>

- instance need these two arguments

  ```
  metadata_startup_script = <<-EOT
          #!/bin/bash
      EOT
  allow_stopping_for_update = true
  ```

- Need to change

  - **Bucket Name** in task 3
  - **Instance Name** in task4, 5
  -  **VPC Name** in task6, 7

  
