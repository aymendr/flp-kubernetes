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
