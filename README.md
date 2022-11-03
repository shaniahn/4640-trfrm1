Setting Up A Droplet using Terraform

1) You need to install Terraform before you start as this is what you'll be using to set up the infrastructure.

- You'll need to go to Terraform's website and download it, here's the link: https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli
- Once you confirm that it's downloaded, you'll need to move the unzipped contents to your machine's path; usually it's best to move it to: ```~/bin```
- Run ```terraform -help``` to see if it's correctly installed

2) You'll need to start by creating a home directory for this lab, then you'll need to create 3 files to add inside:
- .env
- main.tf
- .gitignore

3) The .env file is used to save the value of your API token. We need to take the API token from DigitalOcean and put it into a variable in this file. You can do so like this:
- ```export TF_VAR_do_token={actual_token_value}```
- source this file so that Terraform will use it automatically --> ```source .env```

4) Create a file called main.tf; this is where we will tell Terraform how to provision the infrastructure

- This portion tells Terraform which providers we need packages from so that the .terraform is created when you run ``` terraform init ```
- This portion is also mainly copied and pasted from the documentation for Digital Ocean

```
terraform {
  required_providers {
    digitalocean = {
      source  = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

# Set the variable value in *.tfvars file
# or using -var="do_token=..." CLI option
variable "do_token" {}

# Configure the DigitalOcean Provider
provider "digitalocean" {
  token = var.do_token
}

```

 - Ensure that there is a section in your file where you declare the ssh key to access the droplet like below:

```
data "digitalocean_ssh_key" "droplet_ssh_key" {
  name = "Pasta"
}
```
 - the first set of quotes is the name of the object in Terraform
 - the second set of quotes is what it'll be called when using Terraform
 - the last set of quotes is what the name of the ssh key is on your DigitalOcean account

- For the following block of code, most of the structure is the same as the ssh key block, with just the differentiation between data and resource blocks

```
data "digitalocean_project" "lab_project" {
  name = "4640-labs"
}

# Create a new tag
resource "digitalocean_tag" "do_tag" {
  name = "Web"
}

# Create a new VPC
resource "digitalocean_vpc" "web_vpc" {
  name   = "4640-labs"
  region = "sfo3"
}
```
- The following blocks of code create a new droplet in the selected region (sfo3), adds it to the existing project on Digital Ocean and prints out the server's ip address

```
# Create a new Web Droplet in the sfo3 region
resource "digitalocean_droplet" "web" {
  image    = "rockylinux-9-x64"
  name     = "web-1"
  region   = "sfo3"
  size     = "s-1vcpu-512mb-10gb"
  tags     = [digitalocean_tag.do_tag.id]
  ssh_keys = [data.digitalocean_ssh_key.droplet_ssh_key.id]
  vpc_uuid = digitalocean_vpc.web_vpc.id
}

# add new web-1 droplet to existing 4640-labs project
resource "digitalocean_project_resources" "project_attach" {
  project = data.digitalocean_project.lab_project.id
  resources = [
    digitalocean_droplet.web.urn
  ]
}

output "server_ip" {
  value = digitalocean_droplet.web.ipv4_address
}
```
5) You can now run the following commands:

- ``` terraform init ``` to initialize the packages required
- ``` terraform validate ``` to run a check on your main.tf file
- ``` terraform fmt ``` to format the file automatically
- ``` terraform plan ``` to show what the configuration plan looks like
- ``` terraform apply ``` to run the script to configure infrastructure
- ``` terraform destroy ``` to take down infrastructure that was set up