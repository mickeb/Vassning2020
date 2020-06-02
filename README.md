# Vässning 2020

## Kommandon

### Helm

Skapa ny Helm chart:

`helm create vassning-chart`

Rensa ut "scaffoldade" templates:

`rm -rf vassning-chart/templates/*`

Trunkera .Values filen:

`:> vassning-chart/values.yaml`

Generera service:

`kubectl create deployment nginx --dry-run=true --image=nginx --output yaml > vassning-chart/templates/deployment.yaml`

Installera Helm release:

`helm install my-vassning-release vassning-chart`

Lista Helm releases:

`helm list`

Uppgradera Helm release

`helm upgrade my-vassning-release vassning-chart`

Ta bort Helm release

`helm uninstall my-vassning-release vassning-chart`

Uppgradera (eller installera vid behov) 

`helm upgrade --install my-vassning-release vassning-chart` 

### Services

Generera service

`kubectl create service clusterip nginx --tcp=80 --dry-run=true --output yaml > vassning-chart/templates/service.yaml`

Lista services

`kubectl get svc`

Skapa "shell"

`kubectl run pod --image=alpine -it --rm --generator=run-pod/v1 sh`

### Ingress

Lägg till nginx-ingress helm repo:

`helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx`

Installera ingress-nginx release:

`helm install ingress-nginx ingress-nginx/ingress-nginx`