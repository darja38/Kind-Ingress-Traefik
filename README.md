# Kind-Ingress-Traeffik

##  Objectif

**A ne pas reproduire en Production !!!**

L'objectif de ce cours, est de mettre à disposition des utilisateurs un environnement de développement local utilisant les memes ressources qu'un cluster kubernetes de pre-prod/prod. C'est à dire dans notre cas avec l'utilisation de traefik comme **[Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)**. Ainsi tout les environnements de travail ont la meme base commune, ce qui facilite la migration entre ceux-ci. 

## Installation de kind

**Attention: Kind nécessite la présence de Docker sur son PC:**
- [Windows](https://docs.docker.com/desktop/install/windows-install/)
- [Mac](https://docs.docker.com/desktop/install/mac-install/)
- [Linux](https://docs.docker.com/desktop/install/linux-install/)

Pour suivre ce cours, vous aurez besoin d'un cluster kubernetes. Vous pouvez en créer un en utilisant [kind](https://kind.sigs.k8s.io/docs/user/quick-start/). Vous aurez également besoin d'installer le [kubectl CLI](https://kubernetes.io/docs/tasks/tools/) pour votre distribution.

## Création du cluster

Pour créer notre cluster kubernetes, nous devons taper la commande suivante:

```shell
kind create cluster --config config.yaml
```

Afin de vérifier que tout fonctionne, taper la commande:
```shell
kubectl get nodes
```

## Création & Configuration de Traefik

### Installation des config prérequis pour Treaefik
[Lien vers la documentation](https://doc.traefik.io/traefik/providers/kubernetes-crd/)

- Installation des `Ressources Definition`:
```shell
kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v2.10/docs/content/reference/dynamic-configuration/kubernetes-crd-definition-v1.yml
```
- Installation du RBAC pour Traefik:
```shell
kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v2.10/docs/content/reference/dynamic-configuration/kubernetes-crd-rbac.yml
```
### Installation des différentes ressources pour Traefik

- Installation du [Service Account](https://kubernetes.io/docs/concepts/security/service-accounts/):
```shell
kubectl apply -f traefik/traefik-service-account.yaml
```
- Installation du [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) contenant les informations du certificat SSL encodé en base64:
```shell
kubectl apply -f traefik/traefik-secret.yaml  
```

- Installation du [Service](https://kubernetes.io/docs/concepts/services-networking/service/) pour Trefik:
```shell
kubectl apply -f traefik/traefik-svc.yaml  
```
- Installation du [ConfigMap](https://kubernetes.io/docs/concepts/configuration/configmap/) contenant la configuration de Traefik:
```shell
kubectl apply -f traefik/traefik-configmap.yaml
```

- Installation du [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/):
```shell
kubectl apply -f traefik/traefik-ds.yaml
```

- Installation de l'[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) permettant d'accéder au Dashboard de Traefik:
```shell
kubectl apply -f traefik/traefik-ingress.yaml
```

- Vérifier que le pod Traefik est bien en état `Running`
```shell
kubectl get pods
```

## Création & Configuration d'un pod NGINX

La création de ce pod de démo à pour objectif de permettre la compréhension de comment exposer ses pods avec Traefik.

- Installation de Nginx en tant que [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/):
```shell
kubectl apply -f nginx/nginx-dp.yaml
```

- Installation du Service(https://kubernetes.io/docs/concepts/services-networking/service/):
```shell
kubectl apply -f nginx/nginx-svc.yaml
```

- Installation de l'Ingress pour nginx:
```shell
kubectl apply -f nginx/nginx-ingress.yaml
```

## Accès aux services

- Il faut avant tout éditer son fichier `hosts` afin de pouvoir accéder aux différents services.
[Pour Windows](https://lecrabeinfo.net/ouvrir-et-modifier-le-fichier-hosts-de-windows.html#via-le-bloc-notes)
[Pour Mac/Linux] => `sudo nano /etc/hosts`

Mettre les informations suivantes:
```shell
127.0.0.1   traefik.local
127.0.0.1   nginx.local
```

- Pour accéder au Dashboard de Traefik => `https://traefik.local`
- Pour accéder au Nginx => `http://nginx.local` ou `https://nginx.local`


## Suppression du cluster 

- Pour supprimer le cluster:
```shell
kind delete cluster --name=dev-k8s
```