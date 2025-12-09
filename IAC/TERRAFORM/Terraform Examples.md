
Two container setup web + db to deploy a web app

`docker-httpd.tf`
```
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "3.6.2"
    }
  }
}

provider "docker" {
  host = "unix:///var/run/docker.sock"
}

resource "docker_network" "app_network_t" {
  name = var.v_net_name
}

resource "docker_image" "web-image" {
  name = "shekeriev/bgapp-web"
}

resource "docker_image" "db-image" {
  name = "shekeriev/bgapp-db"
}

resource "docker_container" "con-web" {
  name  = "web-app"
  image = docker_image.web-image.image_id
  networks_advanced {
    name    = var.v_net_name
  }

  volumes {
    container_path = "/var/www/html"
    host_path = "/home/vagrant/bgapp/web"
  }

  ports {
    internal = 80
    external = 8080
  }
}

resource "docker_container" "con-db" {
  name = "db-app"
  image = docker_image.db-image.image_id
  networks_advanced {
    name    = var.v_net_name
    aliases = ["db"] # Additional DNS record to match config.php
  }

  env = [
    "MYSQL_ROOT_PASSWORD=12345"
  ]
}
```

`variables.tf`
```
variable "v_net_name" {
  type = string
}
```

- `host = "unix:///var/run/docker.sock"`: Deploy the container on the local host
- `host = "tcp://<remote_IP>:2375/"`: Deploy the container on the remote docker host

- `resource "docker_image" "httpd"` : Creates a Docker container resource named `httpd` (Terraform internal resource object name, like a name of a function or class)
- `name = "foo"`: The actual Docker container name
- `image = docker_image.httpd.image_id`: Use the Docker image resource `web-image` defined elsewhere, inject its `image_id`

Execute the environment:

```
terraform apply
```
