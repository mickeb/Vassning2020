# Vässning 2020 Terraform

## Kommandon

### 1. Start

I `main.tf`

```
terraform {
  required_providers {
    docker = {
      source = "terraform-providers/docker"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "app-server-1" {
  image = docker_image.nginx.latest
  name  = "app-server-1"
  ports {
    internal = 80
    external = 8000
  }
}
```
Kör:

1. `terraform init` # Kommentar: Detta är som att köra maven sync, npm install etc 
2. `terraform apply` # Kommentar: Planerar och kör
3. `docker ps`

### 2. Lägg till container  


```
resource "docker_container" "app-server-2" {
  image = docker_image.nginx.latest
  name  = "app-server-2"
  ports {
    internal = 80
    external = 8001
  }
}
```

Kör:

1. `terraform apply`
2. `docker ps`

### 3. Bryt ut variabel

Kör:

1. `terraform destroy`

Lägg till högst upp:

```
variable "image_id" {
  type = string
}
```

Uppdatera:

`name         = "nginx:latest"`

till

`name         = docker_image.nginx.latest`

Kör:

1. `terraform apply`
2. Skriv in variabel svaret
   
Kommentar: Men vi kan bättre

1. Skapa `variables.tf` # Kommentar: Standardplats för variabler 
2. Flytta in variabel deklarationen där.
3. Skapa `terraform.tfvars` med innehållet: `image_id`