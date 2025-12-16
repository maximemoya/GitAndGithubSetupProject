# Gestion de Git + GitHub en SSH pour Windows

Voici une **gestion propre et professionnelle de Git + GitHub en SSH pour un projet sur Windows 10/11**, comme on le ferait en équipe ou en contexte pro 👇
(je pars du principe que tu utilises **PowerShell** ou un terminal moderne, et que tu veux éviter les pièges classiques)

---

## 1️⃣ Principe général (important)

👉 **SSH = identité + projet**

* **Une clé SSH = une identité GitHub**
* **Un projet = un remote Git précis**
* Git choisit la clé **via le fichier de configuration SSH**, pas au hasard

Objectif :

* Pouvoir travailler sur **plusieurs projets**
* Avec **plusieurs comptes GitHub**
* Sans conflit ni “push sur le mauvais compte”

---

## 2️⃣ Générer une clé SSH par compte GitHub

### Prérequis : OpenSSH Client

Sur Windows 10/11, le client OpenSSH est normalement installé. Pour vérifier, ouvre PowerShell et tape `ssh`. Si la commande est reconnue, tout est bon. Sinon, il faut l'installer depuis les "Fonctionnalités facultatives" de Windows.

### Exemple : compte perso + compte pro

Ouvre PowerShell et exécute ces commandes :

```powershell
# -f spécifie le chemin du fichier. $env:USERPROFILE est votre dossier utilisateur (ex: C:\Users\VotreNom)
ssh-keygen -t ed25519 -C "github-perso" -f "$env:USERPROFILE\.ssh\id_ed25519_github_perso"
ssh-keygen -t ed25519 -C "github-pro"   -f "$env:USERPROFILE\.ssh\id_ed25519_github_pro"
```

👉 Résultat : les fichiers sont créés dans votre dossier `.ssh` :

```
C:\Users\VotreNom\.ssh\id_ed25519_github_perso
C:\Users\VotreNom\.ssh\id_ed25519_github_perso.pub
C:\Users\VotreNom\.ssh\id_ed25519_github_pro
C:\Users\VotreNom\.ssh\id_ed25519_github_pro.pub
```

Ajoute **chaque clé publique** (`.pub`) sur le bon compte GitHub :

*   GitHub → Settings → SSH and GPG keys → New SSH key
*   Copie-colle le contenu des fichiers `.pub` (tu peux les ouvrir avec Notepad).

---

## 3️⃣ Configurer le fichier `config` SSH (clé de voûte)

Crée / édite le fichier `$env:USERPROFILE\.ssh\config`. Tu peux le faire avec Notepad depuis PowerShell :

```powershell
# Crée le dossier .ssh s'il n'existe pas
if (-not (Test-Path "$env:USERPROFILE\.ssh")) { New-Item -Path "$env:USERPROFILE\.ssh" -ItemType Directory }
# Ouvre le fichier config dans Notepad (le créer s'il n'existe pas)
notepad "$env:USERPROFILE\.ssh\config"
```

Copie-colle cette configuration dedans :

```ssh
# ----- GitHub perso -----
Host github-perso
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github_perso
    IdentitiesOnly yes

# ----- GitHub pro -----
Host github-pro
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github_pro
    IdentitiesOnly yes
```

👉 **Notes importantes** :

*   Le chemin `~/.ssh/id_ed25519_...` est correct même sur Windows. Le client SSH le traduit automatiquement en `$env:USERPROFILE\.ssh\...`.
*   `Host github-perso` ≠ `github.com`. C’est ce nom personnalisé que Git va utiliser pour choisir la bonne clé.

---

## 4️⃣ Configurer Git **au niveau du projet** (best practice)

### Dans le dossier de ton projet :

Ouvre PowerShell dans le dossier de ton projet et configure ton identité **pour ce projet uniquement**.

```powershell
git config --local user.name "Ton Nom"
git config --local user.email "ton@email.com"
```

👉 Ces valeurs **écrasent la configuration globale uniquement pour ce repository**. C'est la méthode la plus sûre pour éviter de commit avec le mauvais compte.

### Alternative : configuration globale

Si tu utilises principalement un seul compte, tu peux configurer ton identité globalement :

```powershell
git config --global user.name "Ton Nom"
git config --global user.email "ton@email.com"
```

👉 **Attention**, cette configuration s'appliquera par défaut à tous tes projets (sauf si surchargée par une config locale).

Pour vérifier :

```powershell
# Vérifie la config locale
git config user.name
git config user.email

# Ou pour voir toutes les configs locales et leur origine
git config --local --list
```

---

## 5️⃣ Lier un projet Git à la bonne identité

### Projet non-existant:

- ajouter un fichier .gitignore
- ajouter un fichier README.md
```powershell
git init
git branch -M main
git add .
git commit -m "first commit"
# Utilise le Host personnalisé (github-perso ou github-pro)
git remote add origin git@github-perso:githubOrganizationName/nameOfTheProject.git
git push -u origin main
```

### Changer le remote d’un projet existant

```powershell
git remote set-url origin git@github-perso:username/mon-projet.git
```

### Cloner un projet existant depuis GitHub

Utilise l'alias (`Host`) défini dans ton fichier de config SSH.

```powershell
git clone git@github-perso:username/mon-projet.git
```

ou

```powershell
git clone git@github-pro:org/mon-projet.git
```

**ATTENTION** : Quand tu clones un projet, pense à toujours configurer ton `user.name` et `user.email` en local (voir section 4️⃣) avant de faire tes premiers commits.

Vérification du remote :

```powershell
git remote -v
```

---

## 6️⃣ Tester la connexion SSH

```powershell
# Teste chaque identité
ssh -T git@github-perso
ssh -T git@github-pro
```

Résultat attendu pour chaque commande :

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

Si ça échoue, les causes probables sont :

*   La clé publique n'est pas (correctement) ajoutée à ton compte GitHub.
*   Le `Host` dans ta commande ne correspond pas à celui du fichier `~/.ssh/config`.
*   Le remote de ton projet est mal configuré (`git@github.com` au lieu de `git@github-perso`).

---

## 7️⃣ Cas fréquent : “ça push avec le mauvais compte”

Cause habituelle :

*   Le remote du projet est configuré avec `git@github.com:...`.
*   Sans `Host` spécifique, Git et SSH utilisent **la première clé qu'ils trouvent** qui fonctionne, ce qui n'est pas forcément la bonne.
*   Ton fichier `~/.ssh/config` est ignoré.

✅ Solution :

*   **NE JAMAIS utiliser `github.com` directement dans l'URL du remote SSH.**
*   Toujours utiliser un `Host` personnalisé de ton fichier de config : `github-perso` / `github-pro`.

---

## 8️⃣ Workflow recommandé (résumé)

```text
1 compte GitHub = 1 clé SSH
1 clé SSH       = 1 Host dans le fichier de config SSH
1 projet        = 1 remote Git pointant vers le bon Host
git config user.* par projet (en local)
```

C’est une méthode **fiable**, **professionnelle** et **sans surprise**.

---

## 9️⃣ Bonus : debug rapide

Voir quelle clé SSH est réellement utilisée :

```powershell
ssh -vT git@github-perso
```
(Regarde les lignes qui parlent de `IdentityFile`)

Voir la config Git (email) qui sera utilisée pour le prochain commit :

```powershell
git config --show-origin --get user.email
```
