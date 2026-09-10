# Exercice — Leçon 1 : Concepts cloud et panorama des providers

> **Bloc 6 · Leçon 1** — Exercice à faire en autonomie, tout se pratique en local (installation d'un outil + réflexion). Pas besoin de compte AWS payant pour cet exercice.

---

## Contexte

Avant de piloter les services AWS, il faut **poser les fondations** : installer l'outil de commande (`aws`) et **ancrer les concepts** (VM, IaaS/PaaS/SaaS, traduction d'un cloud à l'autre) pour ne pas être perdu(e) dans la suite du bloc.

---

## Énoncé

> 📌 **Rappels d'options** : `--user` = installer pour ton compte (pas tout le système) ; `--upgrade` = mettre à jour s'il existe déjà ; `--version` = afficher la version.

### Étape 1 — Vérifier Python et pip

```bash
python3 --version
python3 -m pip --version
```
Note les deux versions dans `notes-exercice-01.md`.

### Étape 2 — Installer l'AWS CLI

```bash
python3 -m pip install --user --upgrade awscli
aws --version
```
Copie le résultat de `aws --version` (la version installée).

### Étape 3 — Lancer la configuration (interruptible)

```bash
aws configure
```
Tu peux **annuler (Ctrl+C)** dès que ça demande des clés : on les créera en toute sécurité à la Leçon 6. Observe simplement les questions posées (région, format de sortie, clés).

### Étape 4 — Réflexion (dans `notes-exercice-01.md`)

1. Définis **VM**, **IaaS**, **PaaS**, **SaaS** chacun avec **une analogie**.
2. Complète ce tableau de traduction cloud :

| Concept | AWS | GCP | Azure |
|---------|-----|-----|-------|
| Machine virtuelle | EC2 | ? | ? |
| Stockage de fichiers | S3 | ? | ? |
| Base de données managée | RDS | ? | ? |

*(Cherche les noms chez GCP et Azure — tu peux utiliser le web, mais essaie d'abord de réfléchir avec ce que tu as appris.)*

---

## Livrable

`notes-exercice-01.md` avec : les versions Python/pip/aws, les 4 définitions + analogies, le tableau de traduction rempli, et tes réponses (2-3 lignes chacun).

Correction détaillée dans **`03-correction.md`**.