# Vässning 2020 Terraform

Installering:
  Levereras som en enda körbarfil go
  Mac OS = homebrew

Editering:
  VS Code + Plugin

## Kommandon

### 1. Start
1. Skapa mappen `TerraformDemo` i `VassningWorkSpace`
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
  keep_locally = true
}

resource "docker_container" "app-server-1-container" {
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

### 1.5 State

1. Öppna upp statefilen.
2. Visa kopplingen mellan terraforms id och dockers id
3. Kommentera bort containern och kör `terraform apply` - Terraform vill ta bort den.
4. Ta bort containern `docker rm -f <id>` i docker och kör `terraform apply` - Terraform säger att allt är grönt
5. Avkommentera och kör `terraform apply`
6. Ta bort containern `docker rm -f <id>` i docker och kör `terraform apply -refresh=true` - Terraform säger att allt är grönt

### 1.6 Terraform rör bara resurser i sitt state

1. Kör `terraform apply igen`
2. Skapa en separat container med: `docker run --rm -d nginx`
3. Kör `terraform destroy`
4. Containern finns kvar

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
4. `terraform.tfvars` includeras automtiskt. det gär även filer på formen fil.auto.tfvars

### 4. Dynamiskt antal containers

Skapa variabeln: `app_server_count`

Kommentar: Ny datatyp number!
 
```
variable "app_server_count" {
  type = number
  default = "2"
}
```

1. Lägg till `count = var.app_server_count` under resource
2. Uppdatera name till: `"app-server-${count.index + 1}"`
3. Uppdatera external port till `external = 8000 + count.index + 1`
4. Uppdatera namnet på resursen!

### 5. Input validering

Lägg till:

```
  validation {
    condition = var.app_servers_count > 1 && var.app_servers_count <= 10
    error_message = "Number of app servers must be between 2 and 10."
  }
```

### 6. Extrahera till module

KÖR: `terraform `

1. Skapa mappen `base_env`
2. Flytta samtliga filer in till `base_env`
3. Ta bort `provider "docker" {}` i modulens main 
4. Skapa ny `main.tf` och `variables.tf`
5. Lägg till NYA variabler i modulens main:

```
variable "env_name" {
  type = string
}

variable "port_range_start" {
  type = number
}
```

Klistra in:

```
terraform {
  required_providers {
    docker = {
      source = "terraform-providers/docker"
    }
  }
}


module "sandbox" {
    source = "./base_env"

    env_name = "sandbox"
    port_range_start = 8000

    image_id = "nginx:alpine"
    app_server_count = 2
}
```
Kör `terraform init`
Kör `terraform apply`

Klistra in:

```
module "production" {
    source = "./env_name"

    environment_name = "production"
    port_range_start = 9000
    
    image_id = "nginx:alpine"
    app_server_count = 8
}
```

Kör `terraform apply`

### 7. Bryt ut till variabel

Lägg till `variables.tf` i root modulen med innehåll:

```
variable "environment_config" {
  type = any
}
```

Lägg till: `terraform.tfvars` i root modulen med innehåll:

```
env_config = {
    sandbox = {
        image_id = "nginx-latest"
        app_server_count = 2
        port_range_start = 8000
    },
    production = {
        image_id = "nginx-alpine"
        app_server_count = 2
        port_range_start = 10000
    }
}
```
Byt ut till en "env" modul

```
module "env" {
  for_each = var.env_config

  source           = "./base_env"
  env_name         = each.key
  port_range_start = each.value["port_range_start"]
  app_server_count = each.value["app_server_count"]
}

```


Kör `terraform init`

Lägg sedan till:

```
preproduction = {
    image_id = "nginx-alpine"
    number_of_app_servers = 2
    port_range_start = 9000
},
```

Nämn anledningen till att vi använder any som typ. 