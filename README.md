# Vässning 2020

## Kommandon

### Helm

Skapa ny Helm chart:

`helm create vassning2020`

Rensa ut "scaffoldade" templates:

`rm -rf vassning2020/templates/*`

Trunkera .Values filen:

`:> vassning2020/values.yaml`

Generera service:

`kubectl create deployment vassning2020-nginx --dry-run=true --image=nginx --output yaml > vassning2020/templates/deploy.yaml`

Installera Helm release:

`helm install vassning2020staging vassning2020`

Lista Helm releases:

`helm list`

Uppgradera Helm release

`helm upgrade vassning2020staging vassning2020`

Ta bort Helm release

`helm uninstall vassning2020staging vassning2020`

Uppgradera (eller installera vid behov) 

`helm upgrade --install vassning2020staging vassning2020` 


### Services
