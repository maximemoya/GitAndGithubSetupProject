Voici une **gestion propre et professionnelle de Git + GitHub en SSH pour un projet**, comme on le ferait en équipe ou en contexte pro 👇
(je pars du principe que tu es sur macOS / Linux, zsh, et que tu veux éviter les pièges classiques)

---

## 1️⃣ Principe général (important)

👉 **SSH = identité + projet**

* **Une clé SSH = une identité GitHub**
* **Un projet = un remote Git précis**
* Git choisit la clé **via `~/.ssh/config`**, pas au hasard

Objectif :

* Pouvoir travailler sur **plusieurs projets**
* Avec **plusieurs comptes GitHub**
* Sans conflit ni “push sur le mauvais compte”

---

## 2️⃣ Générer une clé SSH par compte GitHub

### Exemple : compte perso + compte pro

```bash
ssh-keygen -t ed25519 -C "github-perso" -f ~/.ssh/id_ed25519_github_perso
ssh-keygen -t ed25519 -C "github-pro"   -f ~/.ssh/id_ed25519_github_pro
```

👉 Résultat :

```
~/.ssh/id_ed25519_github_perso
~/.ssh/id_ed25519_github_perso.pub
~/.ssh/id_ed25519_github_pro
~/.ssh/id_ed25519_github_pro.pub
```

Ajoute **chaque clé publique** sur le bon compte GitHub :

* GitHub → Settings → SSH and GPG keys → New SSH key

---

## 3️⃣ Configurer `~/.ssh/config` (clé de voûte)

Crée / édite le fichier :

```bash
nano ~/.ssh/config
```

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

👉 **Très important** :

* `Host github-perso` ≠ `github.com`
* C’est ce nom que Git va utiliser

---

## 4️⃣ Configurer Git **au niveau du projet** (best practice)

### Dans le dossier du projet :

```bash
git config --local user.name "Ton Nom"
git config --local user.email "ton@email.com"
```
👉 Ces valeurs **écrasent le local uniquement sur ce repository**

👉 Évite les commits avec le mauvais compte (global par défault)

### si vous préférez écraser en global:

```bash
git config user.name "Ton Nom"
git config user.email "ton@email.com"
```

👉 Ces valeurs **écrasent le global**

👉 Évite les commits avec le mauvais compte par défault

Vérifier :

```bash
git config user.name
git config user.email
```

ET/OU

```bash
git config --local --list
```

---

## 5️⃣ Lier un projet Git à la bonne identité

### Projet non-existant:

- ajouter un fichier .gitignore
- ajouter un fichier README.md
```bash
git init
```
```bash
git branch -M main
```
```bash
git add .
```
```bash
git commit -m "first commit"
```
 ```bash
git remote add origin git@<github-perso>OU<github-pro>:githubOrganizationName/nameOfTheProject.git
```
 ```bash
git push -u origin main
```

### Ou changer le remote d’un projet existant que vous avez en local

```bash
git remote set-url origin git@github-perso:username/mon-projet.git
```

### Ou cloner depuis un projet existant sur github

```bash
git clone git@github-perso:username/mon-projet.git
```

ou

```bash
git clone git@github-pro:org/mon-projet.git
```

ATTENTION quand vous récupérez un projet pensez à bien =>  4️⃣ Configurer Git (user.name et user.email avant de faire vos commit)

Vérification :

```bash
git remote -v
```

---

## 6️⃣ Tester la connexion SSH

```bash
ssh -T git@github-perso
```

Résultat attendu :

```
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

Si ça échoue :

* mauvaise clé
* mauvais Host
* remote mal configuré

---

## 7️⃣ Cas fréquent : “ça push avec le mauvais compte”

Cause habituelle :

* remote en `git@github.com:...`
* Git utilise **la première clé trouvée**
* `~/.ssh/config` ignoré

✅ Solution :

* **NE JAMAIS utiliser `github.com` directement**
* Toujours `github-perso` / `github-pro`

---

## 8️⃣ Workflow recommandé (résumé)

```text
1 compte GitHub = 1 clé SSH
1 clé SSH = 1 Host SSH
1 projet = 1 remote clair
git config user.* par projet
```

C’est **scalable**, **pro**, et **sans surprise**.

---

## 9️⃣ Bonus : debug rapide

Voir quelle clé est utilisée :

```bash
ssh -vT git@github-perso
```

Voir la config Git effective :

```bash
git config --show-origin --get user.email
```

---
