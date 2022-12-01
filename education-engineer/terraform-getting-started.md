# Getting Started with Terraform

Terraform is the most popular language for defining and provisioning infrastructure as code. Operators define their desired infrastructure state with configuration code written in HCL, the HashiCorp Configuration Language. Terraform reads this configuration and determines the actions it must run against an API to match the configuration. In this guide, you will create a basic Terraform configuration and familiarize yourself with Terraform's essential functionality.

## Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/downloads) v0.13 and above
- [Docker](https://docs.docker.com/get-docker/)

Confirm that you can run `terraform` from your shell before proceeding.

## Create a Terraform working directory

Create a new directory in your environment called `terraform-demo`. This is where you will create your new Terraform configuration.

### CLI

```shell
mkdir terraform-demo
cd terraform-demo
```

## Create your Terraform configuration

Terraform parses all `.tf` files in the working directory together, but it's good practice to separate these files according to what they define. Create three files to reflect your infrastructure:

- `versions.tf` defines the version of the Terraform providers to use. Providers are plugins that enable Terraform to communicate with service APIs.
- `provider.tf` defines the services to communicate with. In this tutorial, Terraform integrates with Docker running in your environment.
- `main.tf` defines the Terraform resources to create. In this tutorial, we are creating two resources: one for the Docker image you want to use, and one for the container to create from that image.

Open the files in your preferred editor and paste the content below into each file.

- `versions.tf`
```hcl
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
    }
  }
}
```

- `provider.tf`
```hcl
provider "docker" {
    host = "unix:///var/run/docker.sock"
}
```

- `main.tf`
```hcl
resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}
resource "docker_image" "nginx" {
  name = "nginx:latest"
}
```

## Initialize the Terraform working directory

In the initialization step, Terraform installs the configured providers to the `.terraform` directory. In this case, Terraform will install the `kreuzwerker/docker` provider.

### CLI
```shell
terraform init
```

## Apply the infrastructure state

Now, run `terraform apply`. Inspect the output -- it reflects the image to pull and the Docker container that will be created. This is an important part of using Terraform, as it allows the operator to review Terraform's actions before taking them. 

### CLI
```shell
$ terraform apply
```

After a few moments, Terraform will indicate the resources have been created. You have now created your first infrastructure resources with Terraform! Run `docker ps` to confirm that the container exists.

### CLI
```shell
docker ps
```

## Destroy the newly created infrastructure

Finally, use Terraform to destroy the resources you have created with `terraform destroy`. This removes the resources Terraform created in the previous step. Be sure to inspect Terraform's output and confirm that the correct resources will be removed!

### CLI

```shell
$ terraform destroy
```

## Next steps

You have now performed the basics of operating Terraform. After defining your desired infrastructure state according to best practices, you created the infrastructure using Terraform. You then removed the infrastructure, destroying only the resources you created in the step prior.

Now, take a look at the [Terraform Registry](https://registry.terraform.io/) to more learn about Terraform's capabilities.
