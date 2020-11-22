# Vässning 2020 Terraform

## Kommandon

### 1. Start
1. Skapa mappen `TerraformDemo`
2. Skapa filen `main.tf`

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

### 4. Dynamiskt antal containers

Skapa variabeln: `number_of_app_servers`

# Kommenter: Ny datatyp number!
 
```
variable "number_of_app_servers" {
  type = number
  default = "2"
}
```

1. Lägg till `count = var.number_of_app_servers` under resource
2. Uppdatera name till: `"app-server-${count.index + 1}"`
3. Uppdatera external port till `external = 8000 + count.index + 1`

### 5. Input validering

Lägg till:

```
  validation {
    condition = var.number_of_app_servers > 1 && var.number_of_app_servers <= 10
    error_message = "Number of app servers must be between 2 and 10."
  }
```