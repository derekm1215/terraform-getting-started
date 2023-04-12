# Terraform - Getting Started

Terraform is the most popular language for defining and provisioning infrastructure as code (IaC). In this tutorial, you will learn how to deploy Docker infrastructure using Terraform.

## Learning Objectives

- Install Terraform.
- Create a Terraform configuration file.
- Follow the Terraform Init, Plan, and Apply workflow to deploy a Docker image and container. 
- Verify the resources using Terraform CLI commands. 
- Destroy the infrastructure using Terraform. 

## Prerequisites

- A local computer or virtual machine
- [Docker](https://docs.docker.com/engine/install/) installed 

## Install Terraform 

Visit [downloads](https://www.terraform.io/downloads.html), choose your operating system, architecture, and preferred installation method, and follow the instructions to install to your system. 

## Create Terraform code

Create a new directory on your machine and change into that directory. 

```shell
$ mkdir terraform-demo
$ cd terraform-demo
```

Next, create a file for your Terraform configuration code. Terraform executes code in any file with a `.tf` file extension. 

[MacOS/Linux](#MacOS/Linux) / [Windows](#Windows)

### MacOS/Linux

```shell
$ touch main.tf
```

### Windows

```shell
$ New-Item -Path "main.tf" -ItemType File
```

Paste the following lines into the file. More information on the Docker Provider can be found [here](https://registry.terraform.io/providers/kreuzwerker/docker/latest/docs).

```hcl
# Specify the Docker provider and version.
terraform {
  required_providers {
    docker = {
      source = "kreuzwerker/docker"
      version = "3.0.2"
    }
  }
}

# Provide configuration information for the provider.
provider "docker" {
    host = "unix:///var/run/docker.sock"
}

# Pull the NGINX Docker image to the local machine.
resource "docker_image" "nginx" {
  name = "nginx:latest"
}

/* Deploy a Docker container based on the nginx image, name it "training," and expose port 80. */
resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "training"
  ports {
    internal = 80
    external = 80
  }
}
```
## Initialize the Working Directory

Initialize Terraform with the [`init`](https://developer.hashicorp.com/terraform/cli/commands/init) command. The initialization process will install the plugin for the Docker provider. 

```shell
$ terraform init
```
```
Initializing the backend...

Initializing provider plugins...
- Finding kreuzwerker/docker versions matching "3.0.2"...
- Installing kreuzwerker/docker v3.0.2...
- Installed kreuzwerker/docker v3.0.2 (self-signed, key ID BD080C4571C6104C)

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

## Plan the Changes

Now, run a Terraform [Plan](https://developer.hashicorp.com/terraform/cli/commands/plan). The Terraform Plan displays the proposed changes. If your code is correct, it will propose the creation of an image resource and a container resource. This does not make any changes to your infrastructure. 

```shell
$ terraform plan
```
```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
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
      + hostname                                    = (known after apply)
      + id                                          = (known after apply)
      + image                                       = (known after apply)
      + init                                        = (known after apply)
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
      + name        = "nginx:latest"
      + repo_digest = (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.
```

## Apply the Changes and Verify

If everything was successful, it's time to apply these changes with the [apply](https://developer.hashicorp.com/terraform/cli/commands/apply) command.

```shell
$ terraform apply
```

You will see a confirmation message after Terraform runs another plan. Enter "yes" to allow Terraform to create the infrastructure. If you use automation software, such as [Terraform Cloud](https://www.hashicorp.com/products/terraform), it may be helpful to skip the confirmation by specifying `-auto-approve`. 

```
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

Once the process is complete, you will receive an "Apply complete" message. 

```hcl
docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 7s [id=sha256:080ed0ed8312deca92e9a769b518cdfa20f5278359bd156f3469dd8fa532db6bnginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 2s [id=bf6f2f4e5630232dee209d1c8c09ccf8fa7aaabe4ecbece0f337b3f5ae3a42b1]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```
## Inspect the State

Verify the image and the container exist by querying the Terraform [state](https://developer.hashicorp.com/terraform/language/state). use the [`terraform state list`](https://developer.hashicorp.com/terraform/cli/commands/state/list) command to list the items in the Terraform state.

```shell
$ terraform state list
```
```
docker_container.nginx
docker_image.nginx
```
When you use a [local backend](https://developer.hashicorp.com/terraform/language/settings/backends/local), the state information is stored in the `terraform.tfstate` file. Open the `terraform.tfstate` file to see its contents. We do not advise editing the state directly except in rare circumstances. We always recommend using the Terraform CLI for any [state manipulation](https://developer.hashicorp.com/terraform/cli/state). 

## Verify the Container is Accessible

Confirm the container is accessible on port 80 using the `curl` command. The default HTML used by NGINX will be displayed. 

```shell
$ curl localhost
```
```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
## Destroy the Infrastructure

Now that you have confirmed everything was successful, it's time to [destroy](https://developer.hashicorp.com/terraform/cli/commands/destroy) the infrastructure.

```shell
$ terraform destroy
```
The `destroy` command will run a destroy plan. 

```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
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
      - group_add                                   = [] -> null
      - hostname                                    = "5ab0f6b62f27" -> null
      - id                                          = "5ab0f6b62f27e179372b1eed362e63a9dc1db3eddae0122cf0eac30daf2dce20" -> null
      - image                                       = "sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1f" -> null
      - init                                        = false -> null
      - ipc_mode                                    = "private" -> null
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
              - ip_address                = "172.17.0.3"
              - ip_prefix_length          = 16
              - ipv6_gateway              = ""
              - mac_address               = "02:42:ac:11:00:03"
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
      - id          = "sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1fnginx:latest" -> null
      - image_id    = "sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1f" -> null
      - name        = "nginx:latest" -> null
      - repo_digest = "nginx@sha256:5b9853baac8d612ed434ecf84b13ae0552280005c173022a7fb0235efd3a0320" -> null
    }

Plan: 0 to add, 0 to change, 2 to destroy.
```
Terraform will ask for another confirmation. Enter `yes` to confirm the destruction. 

```
Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes
  ```
After confirmation, Terraform will destroy the resources and clear the resources from the state. 
```
docker_container.nginx: Destroying... [id=5ab0f6b62f27e179372b1eed362e63a9dc1db3eddae0122cf0eac30daf2dce20]
docker_container.nginx: Destruction complete after 0s
docker_image.nginx: Destroying... [id=sha256:6efc10a0510f143a90b69dc564a914574973223e88418d65c1f8809e08dc0a1fnginx:latest]
docker_image.nginx: Destruction complete after 0s

Destroy complete! Resources: 2 destroyed.
```
## Inspect the State
Use the `terraform state list` command to ensure Terraform deleted the resources.

```shell
$ terraform state list
```
The command should return nothing and the `terraform.tfstate` file will be empty.

## Next Steps

Follow the rest of the [Docker with Terraform](https://developer.hashicorp.com/terraform/tutorials/docker-get-started) tutorial and continue your learning. 
