# flp-kubernetes
cours Introduction à Kubernetes


---

## 📦 Qu’est-ce qu’un container ?

Un **container** est une unité légère, portable et autonome qui exécute une application avec tout ce dont elle a besoin : **code, runtime, librairies, dépendances, configuration**.

> ⚠️ Contrairement à une machine virtuelle, un container **ne virtualise pas un OS complet** : il partage le noyau de l’hôte, ce qui le rend **rapide** et **peu gourmand en ressources**.

---

## 🧱 Éléments clés d’un container

| Élément        | Description                                                                           |
| -------------- | ------------------------------------------------------------------------------------- |
| **Image**      | Modèle immuable (template) à partir duquel on crée un container. Ex : `nginx:latest`. |
| **Container**  | Instance d’une image qui tourne. Une image → plusieurs containers possibles.          |
| **Dockerfile** | Fichier de configuration permettant de construire une image personnalisée.            |
| **Registry**   | Dépôt où sont stockées les images (ex: Docker Hub, GitHub Container Registry).        |

---

## 🔁 Cycle de vie simplifié

1. **Construire** une image → `docker build`
2. **Pousser** l’image dans un registre → `docker push`
3. **Lancer** un container → `docker run`
4. **Superviser / arrêter** → `docker ps`, `docker stop`, `docker logs`

---

## 🧪 Commandes de base Docker

```bash
# Lister les containers actifs
docker ps

# Lister tous les containers (même arrêtés)
docker ps -a

# Lancer un container
docker run -d --name web nginx

# Accéder au shell d’un container
docker exec -it web sh

# Supprimer un container
docker rm -f web

# Construire une image
docker build -t mon-app:1.0 .

# Lancer un container en montant un volume
docker run -v $(pwd)/data:/app/data mon-app:1.0
```

---

## 🔒 Isolation dans un container

Chaque container a sa propre :

* **Vue du système de fichiers**
* **Pile réseau (IP, ports)**
* **Espace de processus**
* **Variables d’environnement**

Mais tous partagent le **même noyau Linux** de l’hôte.

---

## 🤝 Containers vs VMs

| Aspect     | Containers          | Machines virtuelles     |
| ---------- | ------------------- | ----------------------- |
| Légèreté   | Très léger (\~Mo)   | Plus lourd (\~Go)       |
| Démarrage  | Rapide (ms à sec)   | Plus lent (sec à min)   |
| Isolement  | Partage du noyau    | Isolation complète (OS) |
| Ressources | Faible consommation | Plus coûteux            |

---

## 🧱 Conteneurs sans orchestration : qu’est-ce que ça veut dire ?

Cela signifie gérer **manuellement** le cycle de vie des containers :
➡️ Lancement, arrêt, mise à jour, réseau, stockage, logs, etc., **à la main ou avec des scripts**.

---

## 🧰 Outils utilisés

* **Docker** (le plus courant)
* **Podman** (alternative rootless à Docker)
* **containerd / runc** (runtime en arrière-plan, utilisé par Kubernetes)

---

## 📦 Quand utiliser les containers sans orchestrateur ?

### ✅ Cas d’usage valables :

* Environnement de **développement local**
* **Petits projets auto-hébergés** (site perso, blog, scripts, etc.)
* **CI/CD** : tests dans des containers isolés
* **Formation / PoC** (Proof of Concept)

### ❌ Pas adapté à :

* Haute disponibilité
* Scalabilité automatique
* Monitoring centralisé
* Réseaux multi-nœuds complexes

---

## 🔁 Gestion manuelle d’un container (Docker)

### ➤ Démarrer un container

```bash
docker run -d --name monweb -p 8080:80 nginx
```

### ➤ Voir les containers actifs

```bash
docker ps
```

### ➤ Voir les logs

```bash
docker logs monweb
```

### ➤ Entrer dans un container

```bash
docker exec -it monweb sh
```

### ➤ Supprimer un container

```bash
docker rm -f monweb
```

---

## 🔌 Réseau entre containers (sans orchestrateur)

Par défaut, Docker crée un **réseau bridge**. Tu peux créer ton propre réseau pour faire dialoguer plusieurs containers :

```bash
docker network create monreseau

docker run -d --name db --network monreseau mysql

docker run -d --name app --network monreseau myapp
```

* Ici, `app` peut appeler `db` avec `db:3306`

---

## 💾 Partage de données (volumes)

```bash
docker run -v /host/data:/app/data myapp
```

Ou avec un **volume nommé** :

```bash
docker volume create mes-donnees

docker run -v mes-donnees:/app/data myapp
```

---

## 🧪 Simulation basique d'orchestration

Tu peux bricoler un peu d’automatisation avec :

* **Docker Compose** : orchestration simple sur une seule machine.
* **systemd** : pour relancer des containers au démarrage ou après crash.
* **Scripts shell** : enchaîner des `docker run`, `docker logs`, etc.

---

## 📌 Limites sans orchestrateur

| Fonction                | Sans orchestration   | Avec Kubernetes               |
| ----------------------- | -------------------- | ----------------------------- |
| Redémarrage automatique | manuel / script      | natif (`restartPolicy`)       |
| Mise à l’échelle        | manuel (run X times) | `kubectl scale`               |
| Mise à jour progressive | pas supportée        | `RollingUpdate`               |
| Surveillance            | logs/ps/top          | Prometheus / Grafana / probes |
| Répartition de charge   | manuel               | automatique (`Service`)       |

---

## 🎯 Conclusion

> Utiliser des containers **sans orchestration** reste très pertinent **dans des contextes simples ou locaux**.
> Pour des systèmes complexes, il vaut mieux basculer vers **Docker Compose**, puis **Kubernetes** ou autres orchestrateurs.

---

Voici un résumé clair et structuré des **fonctionnalités d'orchestration** dans un environnement de conteneurs (comme Kubernetes, Docker Swarm ou Nomad).

---

## 🚀 Qu’est-ce que l’orchestration de conteneurs ?

L’orchestration consiste à **automatiser le déploiement, la gestion, la mise à l’échelle et la mise en réseau** de conteneurs.
Elle devient indispensable quand on passe de quelques conteneurs à **des dizaines ou centaines répartis sur plusieurs machines**.

---

## ⚙️ Fonctionnalités principales d’un orchestrateur (ex: Kubernetes)

| Fonctionnalité                            | Description                                                                         |
| ----------------------------------------- | ----------------------------------------------------------------------------------- |
| 🔁 **Déploiement automatisé**             | Lancement de containers à partir de fichiers déclaratifs (`YAML`, `Compose`, etc.). |
| 💥 **Redémarrage automatique**            | Relance automatique des containers en cas de crash ou d'erreur (`restartPolicy`).   |
| 📊 **Mise à l’échelle automatique**       | Ajuste dynamiquement le nombre de réplicas selon la charge (CPU, mémoire).          |
| 🔄 **Rolling updates & rollback**         | Mise à jour progressive sans downtime, avec possibilité de retour arrière.          |
| 🔗 **Service discovery & Load balancing** | Attribution automatique de noms DNS et équilibrage de charge entre containers.      |
| 🧠 **Scheduling intelligent**             | Planification des conteneurs sur les nœuds selon les ressources et contraintes.     |
| 🔐 **Sécurité & isolement**               | Gestion fine des accès (RBAC), secrets, politiques réseau, namespaces.              |
| 📦 **Gestion des volumes**                | Montage automatique de volumes persistants (cloud ou locaux).                       |
| 📈 **Monitoring et logs**                 | Intégration avec Prometheus, Grafana, Fluentd, etc.                                 |
| 📡 **Networking multi-nœuds**             | Crée un réseau plat entre containers répartis sur plusieurs serveurs.               |

---

## 🗂️ Exemples d'orchestrateurs populaires

| Outil                 | Particularités                                                |
| --------------------- | ------------------------------------------------------------- |
| **Kubernetes**        | Standard de facto, très complet, puissant mais complexe       |
| **Docker Swarm**      | Plus simple, intégré à Docker, mais moins utilisé aujourd’hui |
| **Nomad** (HashiCorp) | Léger, multi-workload (containers + VMs + autres)             |

---

## 🐳 Qu’est-ce que Kubernetes ?

* **Kubernetes** (ou **K8s**) est une plateforme open-source d’**orchestration de conteneurs**.
* Créé initialement par Google, il est aujourd’hui géré par la Cloud Native Computing Foundation (CNCF).
* Il permet de **déployer, gérer, scaler et monitorer des applications conteneurisées** à grande échelle.

---

## ⚙️ Composants clés de Kubernetes

| Composant              | Rôle principal                                                                              |
| ---------------------- | ------------------------------------------------------------------------------------------- |
| **Master**             | Planifie, contrôle et orchestre le cluster. Gère API Server, Scheduler, Controller Manager. |
| **Node (Worker)**      | Exécute les conteneurs (via kubelet et container runtime comme Docker).                     |
| **Pod**                | Plus petite unité déployable, contient un ou plusieurs containers liés.                     |
| **Service**            | Point d’accès stable pour un ensemble de pods (load balancing, découverte).                 |
| **Deployment**         | Gère la mise à jour et la mise à l’échelle des pods.                                        |
| **ConfigMap & Secret** | Gestion des configurations et des données sensibles.                                        |

---

## 🔑 Fonctionnalités principales de Kubernetes

* **Gestion du cycle de vie** : déploiement, mise à jour (rolling update), rollback.
* **Mise à l’échelle automatique** (Horizontal Pod Autoscaler).
* **Auto-réparation** : redémarrage des containers en cas d’échec.
* **Load balancing** et découverte de services.
* **Orchestration de volumes** pour stockage persistant.
* **Sécurité** avec contrôle d’accès RBAC, Network Policies.
* **Support multi-cloud et hybride**.

---

## 🛠️ Utilisation de base

```bash
kubectl apply -f mon-deploiement.yaml    # déployer une application
kubectl get pods                          # lister les pods
kubectl logs pod-nom                      # consulter logs
kubectl scale deployment/mon-deploiement --replicas=5   # scaler
kubectl delete pod pod-nom                # supprimer un pod
```

---

## 🌐 Qu’est-ce qu’une distribution Kubernetes ?

Une **distribution Kubernetes** est une version packagée de Kubernetes, souvent accompagnée d’outils, configurations, interfaces, et services intégrés pour simplifier l’installation, la gestion, la sécurité et la montée en charge.

---

## 🏆 Distributions Kubernetes populaires

| Distribution                       | Particularités principales                                            | Usage courant                            |
| ---------------------------------- | --------------------------------------------------------------------- | ---------------------------------------- |
| **Vanilla Kubernetes**             | Kubernetes "pur", installé manuellement ou via kops/kubeadm.          | Pour contrôle total, customisation.      |
| **OpenShift (Red Hat)**            | Kubernetes + outils DevOps, sécurité renforcée, interface web.        | Entreprises, plateformes cloud.          |
| **Rancher (SUSE)**                 | Plateforme multi-cluster, interface centralisée, gestion multi-cloud. | Gestion multi-cluster et multi-cloud.    |
| **Google Kubernetes Engine (GKE)** | Kubernetes managé par Google, intégration cloud native.               | Cloud public, simplicité et scalabilité. |
| **Amazon EKS**                     | Kubernetes managé par AWS, intégration services AWS.                  | Cloud AWS, production sécurisée.         |
| **Azure AKS**                      | Kubernetes managé par Microsoft Azure.                                | Cloud Azure, intégration Azure DevOps.   |
| **K3s (Rancher Labs)**             | Kubernetes allégé, très léger (pour edge, IoT, dev local).            | IoT, edge computing, petites infra.      |
| **MicroK8s (Canonical)**           | Kubernetes léger, simple à installer, bonne pour dev et edge.         | Développement local, petits clusters.    |

---

## 🔍 Comment choisir sa distribution ?

* **Cloud managé** (GKE, EKS, AKS) → simplicité, maintenance réduite.
* **Enterprise** (OpenShift, Rancher) → sécurité, support, fonctionnalités avancées.
* **Edge / IoT / Développement** (K3s, MicroK8s) → légèreté, rapidité d’installation.
* **Contrôle total** (Vanilla K8s) → personnalisation maximale.

---



## 📦 Versions de Kubernetes

* Kubernetes suit un cycle de release **trimestriel** (environ tous les 3-4 mois).
* Chaque version est numérotée sous la forme : `vX.Y.Z`

  * `X` = version majeure
  * `Y` = version mineure
  * `Z` = patch/correctif
* Exemples : `v1.23.5`, `v1.24.0`

### Support des versions

* Chaque version majeure est supportée **environ 1 an** (3 versions mineures).
* Les patchs corrigent bugs et failles de sécurité.
* Il est recommandé de toujours utiliser une version **récente** pour bénéficier des dernières fonctionnalités et corrections.

---

## ⚙️ API Kubernetes

* Kubernetes expose une API RESTful pour contrôler toutes les ressources (Pods, Services, Deployments, etc.).
* L’API est versionnée :

  * `v1` (stable)
  * `v1beta1`, `v1alpha1` (fonctionnalités expérimentales)
* Les objets sont définis en YAML ou JSON et envoyés à l’API via `kubectl` ou directement.

### Exemple d’URL API

```plaintext
https://<kube-apiserver>/api/v1/namespaces/default/pods/mypod
```

---

## 📜 Gestion de la compatibilité API

* Certaines APIs évoluent ou sont dépréciées (ex: `extensions/v1beta1` → `apps/v1` pour les Deployments).

* Kubernetes suit un processus de **dépréciation progressive** :

  1. API introduite en alpha/beta.
  2. Passée en stable.
  3. Dépréciée (plus recommandée).
  4. Supprimée dans une version future.

* Il est important de **mettre à jour** ses manifests YAML régulièrement.

---

## 🛠️ Exemples d’API Kubernetes courantes

| Ressource   | API Group | Version |
| ----------- | --------- | ------- |
| Pod         | core      | v1      |
| Deployment  | apps      | v1      |
| StatefulSet | apps      | v1      |
| DaemonSet   | apps      | v1      |
| Service     | core      | v1      |
| ConfigMap   | core      | v1      |
| Secret      | core      | v1      |

---

## 💡 Bonnes pratiques

* Toujours valider la version de Kubernetes cible avant déploiement.
* Surveiller les dépréciations dans les notes de version officielles.
* Utiliser des outils comme `kubectl convert` ou `kubeval` pour valider et convertir les manifests.
* Garder les manifests à jour avec les dernières versions d’API.

---
