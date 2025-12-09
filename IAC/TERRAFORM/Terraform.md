
A `.tf` file in **Terraform** is a configuration file written in HCL (HashiCorp Configuration Language). It defines IaC - everything Terraform needs to create, manage, and destroy infrastructure.

>Terraform is declarative - it matches the desired state of infrastructure as declared. If, for example, one container is declared, the result is one container. Changing the configuration, does NOT create new containers.

Directory layout (Best-practice)

```
terraform/
├── environments/ 
│ ├── dev.tfvars 
│ └── prod.tfvars 
├── modules/ 
│ └── networking/ 
│ ├── variables.tf 
│ ├── main.tf 
│ └── outputs.tf 
├── variables.tf 
├── main.tf 
├── outputs.tf 
└── terraform.tfvars # sane defaults for dev
```

Terraform code goes in the `main.tf` file
#### main.tf Anatomy

`main.tf`
```
# Main blocks (provider-specific)
Terraform
Provider
Resource block 1
Resource block N
# Optional blocks (Terraform generic)
Variables
Locals
Data sources
Outputs
```

Refer to provider documentation for `terraform, provider, resource` blocks.
##### Terraform Block

Defines required providers and versions. 

`terraform.tf`
```
terraform {
  required_providers {
	<provider1> = { ... }
	<provider2> = { ... }
  }
	required_version = ">=1.0"
}
```

##### Provider Block

Specifies the backend Terraform must use (like AWS, Docker, GCP, etc.).

`terraform.tf`
```
provider "<provider>" { ... }
```

##### Resource Block

The core building blocks - define infrastructure objects (a VM, container, bucket, etc.).

`main.tf`
```
resource "<resource_type>" "<res_name>" { ... }
```

Each resource block declares:
1. **resource type**: e.g., `docker_container`, `aws_instance`, etc. (provider-specific)
2. **resource object name**: Terraform internal object name

The `provider` and `resource` blocks are provider-specific.

##### Variable Block (Optional)

Variables can be declared in `main.tf` or exported to external `variables.tf` file.

Input values can be defined outside the code for flexibility.
- Example: `variable "region" { ... }`

Primitive types

- `string`: `"Hello"`
- `number`: `10`
- `bool`: `true | false`
- `any`: Any type

Complex types:

- `list(TYPE)`
- `map(TYPE)`: Key-value pairs. `{ env = "dev", port = "8080" }`
- `set(TYPE)`: Unique, unordered. 

`variables.tf`
```
variable "v_con_name" {
	type = string
	default = "my-container"
}
```

Example: Assigning variable values depending on other values (mode in this case). In effect, conditional logic depending based on dictionaries.

`variables.tf`
```
variable "mode" {
  description = "mode: prod or dev"
  type        = string
  
  validation { # Defines the possible choices
    condition = contains(["prod", "dev"], var.mode)
    error_message = "Mode must be either 'prod' or 'dev'. Received: var.mode"
  }
}

variable "v_con_name" {
  description = "Container name"
  type        = map(string)
  default = {
    dev  = "site-dev"
    prod = "site-prod"
  }
}
```

Variable referencing in the resources

`main.tf`
```
resource "docker_container" "con-web" {
  name  = var.v_con_name[var.mode]
  image = docker_image.img-web.image_id
  env   = [                # Pass an ENV variable to the container
    "APP_MODE=${var.mode}"
  ]
```

Python equivalent

```python
mode = ['prod', 'dev']  
v_con_name = {'dev': 'site-dev',  
              'prod': 'site-prod',  
              }  
mode_value = input("Enter mode: ")  
print(v_con_name[mode_value]) if mode_value in mode else print(f"Mode must be either 'prod' or 'dev'. Received: {mode_value}")
```

Get the value from the map `var.v_ext_port` using the key `var.mode`

```
external = var.v_con_name[var.mode]
```

Default keys can be used as a fallback to avoid runtime errors

```
external = try(var.v_con_name[var.mode], "dev")
```

Environment variables can be passed to containers.

`main.tf`
```
resource "docker_container" "con-web" {
  ...
  env   = [                
    "APP_MODE=${var.mode}"  # = docker -e APP_DATA="dev or prod"
  ]
```

Terraform will pass the variable to docker and docker to PHP

`index.php`
```php
getenv('APP_MODE')
```

Pass a variable value in CLI

```bash
terraform apply -var "KEY=VALUE" 
```

If `dev` is specified as value for the variable, the container will be named "`site-dev"

Variables can be assigned values in the `*.tfvars` files. Sensitive data can be moved here.

`terraform.tfvars`
```
mode = "dev"
v_con_name = "webinario"
v_image = "httpd:alpine"
v_ext_port = 8080
```

If the file has a custom name or is in another folder

```bash
terraform plan -var-file="environments/prod.tfvars"
```

##### Locals (Optional)

Locals are like "scratch variables" that live only inside  a module. They are not exposed at the module boundary (unlike `variable` and `output`) and can be referenced with the prefix `local.` anywhere in the same module.

`locals.tf`
```
locals {
  app_name       = "myapp"
  image_name     = "nginx"
  container_name = "${local.app_name}-container"
  app_port       = 8080
  internal_port  = 80
}
```

Reference the locals

`main.tf`
```
resource "docker_image" "nginx" {
  name = local.image_name
}

resource "docker_container" "nginx" {
  name  = local.container_name
  image = docker_image.nginx.name
  ports {
    internal = local.internal_port
    external = local.app_port
  }
}
```

Using variables and locals together

```
# variable
variable "environment" {
  type    = string
  default = "dev"
}

# locals using variable
locals {
  container_name = "myapp-${var.environment}"
  port_mapping   = var.environment == "prod" ? 80 : 8080
}
```

##### Data Sources

- Read information from existing infrastructure outside Terraform’s control.
- Useful when you need to query something instead of creating it.
- Example: `data "aws_ami" "ubuntu" { ... }`

##### Output Block (Optional)

- Values Terraform prints after apply, often used to pass important info (like IPs, URLs, IDs).
- Example: `output "instance_ip" { ... }`

List terraform resource objects:

```bash
terraform state list
```

```
docker_container.web-cnt
docker_image.web-image
```

Resource attributes can be listed with:

```bash
terraform state show "RES_TYPE"."RES.NAME"
```

Example: List resource attributes of a `docker_container` object

```bash
terraform state show docker_container.web-cnt
```

```
entrypoint                                  = []
hostname                                    = "eda3d0fa64e0"
name                                        = "web"
network_data                                = [
	{
		gateway                   = "172.17.0.1"
		ip_address                = "172.17.0.2"
		ip_prefix_length          = 16
		mac_address               = "42:c0:dc:5e:8f:18"
		network_name              = "bridge"
	},
]
```

The output block structure. 

```
output <output_object_name> {
  value = <resource_type>.<resource_object_name>.<res_property>
}
```

We want to print the IP address of the `web-cnt` resource object after apply:

```
output "network-informaciq" {
  value = docker_container.web-cnt.network_data[0].ip_address
}
```

```
Outputs:
network-informaciq = "172.17.0.2"
```

To print multiple items from a single resource

```
output "container_inforamciqon" {
  value = {
    name     = docker_container.web-cnt.name
    hostname = docker_container.web-cnt.hostname
  }
}
```

```
container_informacion = {
  "hostname" = "eda3d0fa64e0"
  "name" = "web"
}
```

Lists, like `network_data`, can be iterated with a for loop to display multiple values:

```
output "network-informaciq" {
  value = [
    for net in docker_container.web-cnt.network_data : {
        ip_address = net.ip_address
        gateway    = net.gateway
    }
  ]
}
```

Outputs can be fetched:

```bash
terraform output [-json] ["SPECIFIC_OUTPUT"]
```

#### Terraform Administration

```bash
terraform init # Set up an environment 
terraform validate # Validate the config
terraform plan [-out "FILE".plan] # See changes before apply [save them to file]
terraform apply # Execute the environment
terraform destroy # Destroy the environment
```

##### Terraform State

Terraform maintains a state file `terraform.tfstate` that stores information about all resources it manages:
- Resource IDs
- Attributes (Like IPs, hostnames, etc.).
- Dependencies between resources

Reads and outputs a Terraform state in human-readable format

```bash
terraform show
```

Terraform keeps this data to compare between the current state and the desired state. It is critical for `outputs` as terraform pulls the info from there.

List resource objects

```bash
terraform state list
```

List resource attributes of a resource

```bash
terraform state show "RESOURCE_TYPE.RES_OBJECT"
```

To pull the raw state in `.json` format:

```bash
terraform state pull
```

#### Variables VS Locals

Variables:

- Set from User input, CLI `-var`, `*.tvars`, or defaults
- Accept dynamic input from user
- Can be overridden during runtime

Locals:

- Defined only inside Terraform code
- Simplify and reuse expressions internally
- Static, can't be overridden
#### Modules (Optional)

Re-usable chunks of Terraform code from local directories or registries.

```
module "network" {
	source = "./network"
	cidr = "10.0.0.0/16"
}
```

#### File Naming

- `main.tf`: Main resources
- `provider.tf`: Provider configs
- `variables.tf`: Input variables
- `outputs.tf`: Outputs
- `terraform.tfvars`: Actual variable values

#### Best Practices

- Keep it modular (use modules for repeated infra)
- Use variables for flexibility
- Use outputs for key info
- Version-lock your providers

#### Workspaces

Terraform workspaces are a way to manage multiple instances of the same IaC - useful for environments like `dev`, `staging` and `prod`.

Workspaces allow to maintain separate state files using the same code. Work **per folder!**

List workspaces

```bash
terraform workspace list
```

Create a workspace

```bash
terraform workspace new dev
```

Switch workspaces

```bash
terraform workspace select dev
```

Delete a workspace

```bash
terraform workspace delete dev
```

Each workspace has its own Terraform state:
- `terraform.tfstate` for `default`
- `terraform.tfstate.d/<workspace>/terraform.tfstate` for others

#### Taints

Taints are used to mark a resource (a container for example) for recreation during the next `terraform apply`. Marking the a container as "tainted" means:

> This resource is no longer valid - destroy it and recreate it on the next apply

Tainting comes in handy when a configuration is altered before the old has been destroyed.

List states

```bash
terraform state list
```

`resources`
```
RESOURCE TYPE    .RESOURCE NAME
docker_container.con-web
docker_image.img-web
```

Syntax

```bash
terraform taint <resource_type>.<resource_name>
terraform taint docker_container.con-web
```

Undo a taint

```bash
terraform untaint <resource_type>.<resource_name>
```

On the next apply:

- Destroy the existing container (`docker_container.con-web)
- Recreate it with the updated config from `main.tf`

#### Executing Remote Commands

Running commands on a remote hosts is done with the `remote-exec` provider

Basic setup

`main.tf`
```
resource "null_resource" "clone_repo" {
  provisioner "remote-exec" {
    connection {
      type        = "ssh"
      host        = "dockerhost"
      user        = "vagrant"
      private_key = file("~/.ssh/id_rsa")
    }

    inline = [
      "rm -rf /home/vagrant/bgapp",
      "git clone https://github.com/shekeriev/bgapp.git"
    ]
  }
}
```