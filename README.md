Voici un **README court, clair et pro**, adapté à un projet technique 👇

---

# Gestion Git & GitHub – Connexion SSH

Ce projet documente une **configuration propre et sécurisée de Git et GitHub via SSH**, adaptée aux environnements professionnels et multi-comptes.

## Objectifs

* Utiliser GitHub **sans mot de passe** via SSH
* Gérer **plusieurs comptes GitHub** sur une même machine
* Éviter les erreurs de push / commit sur le mauvais compte
* Appliquer de bonnes pratiques **par projet**

## Contenu

* Génération de clés SSH par compte
* Configuration de `~/.ssh/config`
* Association d’un projet Git à une identité SSH
* Configuration Git locale (`user.name`, `user.email`)
* Tests et debug de connexion SSH

## Prérequis

* Git
* Accès à GitHub
* macOS / Linux (zsh, bash)

## Principe clé

> **1 compte GitHub = 1 clé SSH = 1 host SSH**

## Usage

Utiliser des remotes Git de la forme :

```bash
git@github-perso:username/repo.git
git@github-pro:org/repo.git
```

## Public cible

Développeurs souhaitant une **gestion Git fiable**, claire et sans surprise, en solo ou en équipe.

---
