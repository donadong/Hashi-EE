
# Getting Started with Terraform Using Docker

## Introduction

Terraform lets you define infrastructure as code (IaC). You write configuration files that describe resources, and Terraform creates and manages them for you.

In this guide, you use Terraform and the Docker provider to run an NGINX container on your local machine.

### Learning objectives

In this guide, you will:

- Install and verify Terraform  
- Write a basic Terraform configuration  
- Initialize a Terraform working directory  
- Plan and apply infrastructure changes  
- Destroy resources when you no longer need them  

---

## Prerequisites

You need:

- Terraform installed (latest stable)  
  https://developer.hashicorp.com/terraform/downloads  
- Docker installed and running  
- A terminal and a text editor  
- Internet access to pull Docker images  

Verify Docker is running:

```shell
docker ps
```

### Terminal output

```text
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

```

---

# Action 1: Create a working directory

Create a directory for your Terraform project and move into it.

```shell
md terraform-demo
cd terraform-demo
```

### Terminal output

```text
D:\terraform-demo>
```

---

# Action 2: Create the configuration file

Create a Terraform configuration file.

```shell
notepad main.tf
```

Open `main.tf` and add:

```hcl
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0"
    }
  }
}

provider "docker" {
  host = "unix:///var/run/docker.sock"
}

resource "docker_image" "nginx" {
  name = "nginx:latest"
}

resource "docker_container" "nginx" {
  name  = "terraform-nginx"
  image = docker_image.nginx.image_id

  ports {
    internal = 80
    external = 8080
  }
}
```

---

# Action 3: Initialize Terraform

Initialize the directory. Terraform downloads the Docker provider.

```shell
terraform init
```

### Terminal output

```text
Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

# Action 4: Format and validate

Format the configuration:

```shell
terraform fmt
```

Validate syntax:

```shell
terraform validate
```

### Terminal output

```text
Success! The configuration is valid.
```

---

---

# Action 5: Plan changes

Preview what Terraform will create.

```shell
terraform plan
```

### Terminal output

```text
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
      + hostname                                    = (known after apply)
      + id                                          = (known after apply)
      + image                                       = (known after apply)
      + init                                        = (known after apply)
      + ipc_mode                                    = (known after apply)
      + log_driver                                  = (known after apply)
      + logs                                        = false
      + must_run                                    = true
      + name                                        = "terraform-nginx"
      + network_data                                = (known after apply)
      + network_mode                                = "bridge"
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

      + healthcheck (known after apply)

      + labels (known after apply)

      + ports {
          + external = 8080
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

───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if
you run "terraform apply" now.
```

---

# Action 6: Apply the configuration

Create the resources.

```shell
terraform apply
```

Type `yes` when prompted.

### Terminal output

```text
docker_image.nginx: Creating...
docker_image.nginx: Creation complete after 8s [id=sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208nginx:latest]
docker_container.nginx: Creating...
docker_container.nginx: Creation complete after 5s [id=d9d0004af19c4658013f9a219a1017a26b63ca2d1ef095dfeb0598789d07b3b6]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

---

# Action 7: Verify the container

List running containers:

```shell
docker ps
```

### Terminal Output

```text
CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                  NAMES
d9d0004af19c   341bf0f3ce6c   "/docker-entrypoint.…"   58 seconds ago   Up 54 seconds   0.0.0.0:8080->80/tcp   terraform-nginx
```

Test the web server:

```shell
curl http://localhost:8080
```

### Terminal output

```text
D:\terraform-demo>curl localhost:8080
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

---

# Action 8: Destroy infrastructure

Remove the resources:

```shell
terraform destroy
```

Type `yes` when prompted.

### Terminal output

```text
docker_container.nginx: Destroying... [id=d9d0004af19c4658013f9a219a1017a26b63ca2d1ef095dfeb0598789d07b3b6]
docker_container.nginx: Destruction complete after 0s
docker_image.nginx: Destroying... [id=sha256:341bf0f3ce6c5277d6002cf6e1fb0319fa4252add24ab6a0e262e0056d313208nginx:latest]
docker_image.nginx: Destruction complete after 1s

Destroy complete! Resources: 2 destroyed.
```

---

# Next steps

In this guide, you used Terraform to manage a Docker container. You learned how to initialize a project, review changes, apply infrastructure, and clean up resources.

To continue learning, explore:

Terraform fundamentals  
https://developer.hashicorp.com/terraform/intro

Terraform tutorials  
https://developer.hashicorp.com/terraform/tutorials

Terraform Registry  
https://registry.terraform.io/

You can also extend this example by:

- Adding variables for ports or container names  
- Creating multiple containers  
- Using a cloud provider such as AWS, Azure, or Google Cloud  
