# Getting Started with Terraform

Terraform is the most popular language for defining and provisioning infrastructure as code. Engineers define their desired infrastructure state with configuration code written in HCL, the HashiCorp Configuration Language. Terraform reads this configuration and determines the actions it needs to run against an API to match the infrastructure state.

## Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/downloads)
- [Docker](https://docs.docker.com/get-docker/)

Confirm that you can run `terraform` from your shell before proceeding.

## Create a Terraform working directory

Create a new directory in your environment called `terraform-demo`. This is where your Terraform configuration will live.

### CLI

```shell
mkdir terraform-demo
cd terraform-demo
```

## Create your Terraform configuration

Terraform parses all `.tf` files in the working directory together, but it's good practice to separate these files according to what configuration they define. Create three files to reflect your infrastructure:

- `versions.tf` defines the version of the Terraform providers to use. Providers are plugins that enable Terraform to communicate with service APIs.
- `provider.tf` defines the services that we will communicate with. In this tutorial, Terraform will integrate with Docker running in your environment.
- `main.tf` defines the Terraform resources to create. In this tutorial, you will create two resources: one for the Docker image you want to use, and one for the container to create from that image.

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

This will install the `docker` provider to the `.terraform` directory.

### CLI
```shell
terraform init
```

## Apply the infrastructure state

Now, we will run `terraform apply`. Inspect the output -- it will reflect the Docker container and image that we want to use. This is an important part of using Terraform, as it allows you to review the actions Terraform will take to match your desired infrastructure. 

```shell
$ terraform apply
```

Terraform will ask you to confirm that you want to take the actions shown. When you confirm by typing `yes`, Terraform will take a few moments to run, and display a message indicating that the resources have been created.

## Confirm the infrastructure has been created

You have now created your first infrastructure resources with Terraform! Confirm the container is up and running with `docker ps`.

### CLI
```shell
docker ps
```

## Destroy the newly created infrastructure

Finally, use Terraform to destroy the resources you have created with `terraform destroy`. This action requires user confirmation, like `apply`, but will remove the resources Terraform has created. Be sure to inspect the output to confirm the correct resources will be removed!

### CLI

```shell
$ terraform destroy
```

## Next steps

You have now mastered the basics of operating Terraform. After defining your desired infrastructure state according to best practices, you created it using Terraform, and destroyed only the resources you created.

Now, take a look at the [Terraform Registry](https://registry.terraform.io/) to learn about other parts of Terraform.
