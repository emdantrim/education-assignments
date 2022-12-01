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
  image = docker_image.nginx.image_id
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

### Output
```
Initializing the backend...

Initializing provider plugins...
- Finding latest version of kreuzwerker/docker...
- Installing kreuzwerker/docker v2.23.1...
- Installed kreuzwerker/docker v2.23.1 (self-signed, key ID BD080C4571C6104C)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

## Apply the infrastructure state

Now, run `terraform apply`. Inspect the output -- it reflects the image to pull and the Docker container that will be created. This is an important part of using Terraform, as it allows the operator to review Terraform's actions before taking them. 

### CLI
```shell
$ terraform apply
```

### Output
```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create

Terraform will perform the following actions:

  # docker_container.nginx will be created
  + resource "docker_container" "nginx" {
      + attach                                      = false
      + bridge                                      = (known after apply)
      + command                                     = (known after apply)
      + container_logs                              = (known after apply)
      + container_read_refresh_timeout_milliseconds = 15000
      + entrypoint                                  = (known after apply)
      + env                                         = (known after apply)
      + exit_code                                   = (known after apply)
      + gateway                                     = (known after apply)
      + hostname                                    = (known after apply)
      + id                                          = (known after apply)
      + image                                       = (known after apply)
      + init                                        = (known after apply)
      + ip_address                                  = (known after apply)
      + ip_prefix_length                            = (known after apply)
      + ipc_mode                                    = (known after apply)
      + log_driver                                  = (known after apply)
      + logs                                        = false
      + must_run                                    = true
      + name                                        = "training"
      + network_data                                = (known after apply)
      + read_only                                   = false
      + remove_volumes                              = true
      + restart                                     = "no"
      + rm                                          = false
      + runtime                                     = (known after apply)
      + security_opts                               = (known after apply)
      + shm_size                                    = (known after apply)
      + start                                       = true
      + stdin_open                                  = false
      + stop_signal                                 = (known after apply)
      + stop_timeout                                = (known after apply)
      + tty                                         = false
      + wait                                        = false
      + wait_timeout                                = 60

      + healthcheck {
          + interval     = (known after apply)
          + retries      = (known after apply)
          + start_period = (known after apply)
          + test         = (known after apply)
          + timeout      = (known after apply)
        }

      + labels {
          + label = (known after apply)
          + value = (known after apply)
        }

      + ports {
          + external = 80
          + internal = 80
          + ip       = "0.0.0.0"
          + protocol = "tcp"
        }
    }

  # docker_image.nginx will be created
  + resource "docker_image" "nginx" {
      + id          = (known after apply)
      + image_id    = (known after apply)
      + latest      = (known after apply)
      + name        = "nginx:latest"
      + output      = (known after apply)
      + repo_digest = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 7s [id=sha256:6b3e1257976b8a6fd83636599d64df8dd87d008ce2e5d5eb721c11aa8e338862nginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 0s [id=1bdf0d100979fccdf18e12b5193e11e8e7c643d0030ace35a17891b61bf6927f]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

After a few moments, Terraform will indicate the resources have been created. You have now created your first infrastructure resources with Terraform! Run `docker ps` to confirm that the container exists.

### CLI
```shell
docker ps
```

### Output
```
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                NAMES
0a5aae1dd6b9   6b3e1257976b   "/docker-entrypoint.…"   3 seconds ago   Up 2 seconds   0.0.0.0:80->80/tcp   training
```

## Destroy the newly created infrastructure

Finally, use Terraform to destroy the resources you have created with `terraform destroy`. This removes the resources Terraform created in the previous step. Be sure to inspect Terraform's output and confirm that the correct resources will be removed!

### CLI

```shell
$ terraform destroy
```

### Output
```
docker_image.nginx: Refreshing state... [id=sha256:6b3e1257976b8a6fd83636599d64df8dd87d008ce2e5d5eb721c11aa8e338862nginx:latest]
docker_container.nginx: Refreshing state... [id=0a5aae1dd6b9abebfdc87fe356731a63526a318b0ad0c4afe3403c7eb184b2f8]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  - destroy

Terraform will perform the following actions:

  # docker_container.nginx will be destroyed
  - resource "docker_container" "nginx" {
      - attach                                      = false -> null
      - command                                     = [
          - "nginx",
          - "-g",
          - "daemon off;",
        ] -> null
      - container_read_refresh_timeout_milliseconds = 15000 -> null
      - cpu_shares                                  = 0 -> null
      - dns                                         = [] -> null
      - dns_opts                                    = [] -> null
      - dns_search                                  = [] -> null
      - entrypoint                                  = [
          - "/docker-entrypoint.sh",
        ] -> null
      - env                                         = [] -> null
      - gateway                                     = "172.17.0.1" -> null
      - group_add                                   = [] -> null
      - hostname                                    = "0a5aae1dd6b9" -> null
      - id                                          = "0a5aae1dd6b9abebfdc87fe356731a63526a318b0ad0c4afe3403c7eb184b2f8" -> null
      - image                                       = "sha256:6b3e1257976b8a6fd83636599d64df8dd87d008ce2e5d5eb721c11aa8e338862" -> null
      - init                                        = false -> null
      - ip_address                                  = "172.17.0.2" -> null
      - ip_prefix_length                            = 16 -> null
      - ipc_mode                                    = "private" -> null
      - links                                       = [] -> null
      - log_driver                                  = "json-file" -> null
      - log_opts                                    = {} -> null
      - logs                                        = false -> null
      - max_retry_count                             = 0 -> null
      - memory                                      = 0 -> null
      - memory_swap                                 = 0 -> null
      - must_run                                    = true -> null
      - name                                        = "training" -> null
      - network_data                                = [
          - {
              - gateway                   = "172.17.0.1"
              - global_ipv6_address       = ""
              - global_ipv6_prefix_length = 0
              - ip_address                = "172.17.0.2"
              - ip_prefix_length          = 16
              - ipv6_gateway              = ""
              - network_name              = "bridge"
            },
        ] -> null
      - network_mode                                = "default" -> null
      - privileged                                  = false -> null
      - publish_all_ports                           = false -> null
      - read_only                                   = false -> null
      - remove_volumes                              = true -> null
      - restart                                     = "no" -> null
      - rm                                          = false -> null
      - runtime                                     = "runc" -> null
      - security_opts                               = [] -> null
      - shm_size                                    = 64 -> null
      - start                                       = true -> null
      - stdin_open                                  = false -> null
      - stop_signal                                 = "SIGQUIT" -> null
      - stop_timeout                                = 0 -> null
      - storage_opts                                = {} -> null
      - sysctls                                     = {} -> null
      - tmpfs                                       = {} -> null
      - tty                                         = false -> null
      - wait                                        = false -> null
      - wait_timeout                                = 60 -> null

      - ports {
          - external = 80 -> null
          - internal = 80 -> null
          - ip       = "0.0.0.0" -> null
          - protocol = "tcp" -> null
        }
    }

  # docker_image.nginx will be destroyed
  - resource "docker_image" "nginx" {
      - id          = "sha256:6b3e1257976b8a6fd83636599d64df8dd87d008ce2e5d5eb721c11aa8e338862nginx:latest" -> null
      - image_id    = "sha256:6b3e1257976b8a6fd83636599d64df8dd87d008ce2e5d5eb721c11aa8e338862" -> null
      - latest      = "sha256:6b3e1257976b8a6fd83636599d64df8dd87d008ce2e5d5eb721c11aa8e338862" -> null
      - name        = "nginx:latest" -> null
      - repo_digest = "nginx@sha256:e209ac2f37c70c1e0e9873a5f7231e91dcd83fdf1178d8ed36c2ec09974210ba" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

docker_container.nginx: Destroying... [id=0a5aae1dd6b9abebfdc87fe356731a63526a318b0ad0c4afe3403c7eb184b2f8]
docker_container.nginx: Destruction complete after 0s
docker_image.nginx: Destroying... [id=sha256:6b3e1257976b8a6fd83636599d64df8dd87d008ce2e5d5eb721c11aa8e338862nginx:latest]
docker_image.nginx: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
```

## Next steps

You have now performed the basics of operating Terraform. After defining your desired infrastructure state according to best practices, you created the infrastructure using Terraform. You then removed the infrastructure, destroying only the resources you created in the step prior.

Now, take a look at the [Terraform Registry](https://registry.terraform.io/) to more learn about Terraform's capabilities.
