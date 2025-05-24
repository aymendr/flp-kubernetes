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
