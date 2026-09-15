# Exercice — Leçon 3 : le state et son backend

> **Bloc 8 · Leçon 3** — Exercice en autonomie, **100 % local et gratuit** : tu vas ouvrir le « registre » de Terraform, provoquer un drift, le voir détecté, puis écrire la configuration d'un backend distant (sans l'activer — elle s'activera en Leçon 5).

---

## Contexte

Le state est le fichier le plus important — et le plus fragile — d'un projet Terraform. L'exercice te fait **manipuler le registre** (l'inspecter, le voir détecter un drift, le sauvegarder) et **préparer sa migration** vers un backend distant. Tout se pratique sur le projet de la Leçon 2.

---

## Énoncé

> 📌 **Rappels d'options** : `terraform state list` = liste des ressources gérées ; `terraform state show RESSOURCE` = détails d'une ressource ; `terraform show` = tout le state en lisible ; `terraform plan` = prévisualisation (il rafraîchit le state en lisant le réel) ; `rm FICHIER` = supprimer un fichier.

### Étape 1 — Préparer le terrain

```bash
# Retourne dans le projet de la Leçon 2 (recrée-le si tu l'as supprimé :
# les fichiers sont dans la correction 03-correction.md de la Leçon 2).
cd ~/atelier-securise

# Vérifie que tout est propre : après un destroy, relance apply.
terraform apply    # tape yes si des ressources doivent être créées
```

### Étape 2 — Ouvrir le registre

Exécute ces commandes et note les résultats dans `notes-exercice-03.md` :

```bash
# 1. Liste les ressources que Terraform gère (son registre).
terraform state list

# 2. Affiche les détails de la ressource "secret".
terraform state show random_string.secret

# 3. Affiche tout le state, présenté de façon lisible.
terraform show
```

Questions (réponds en une ligne chacune) :
1. Combien de ressources apparaissent dans `state list` ?
2. Dans `state show random_string.secret`, où retrouve-t-on la **valeur réelle** du secret ?
3. Compare avec `cat terraform.tfstate | head -30` : de quel format s'agit-il ?

### Étape 3 — Provoquer un drift et le voir détecté

C'est l'expérience clé de la leçon : jouer le rôle du « collègue qui modifie à la main ».

```bash
# 4. Supprime À LA MAIN le fichier créé par Terraform (le drift !).
rm journal-securise.txt

# 5. Demande le plan : Terraform interroge le réel, détecte l'écart, propose la réparation.
terraform plan

# 6. Reprends le contrôle : applique la réparation.
terraform apply   # tape yes

# 7. Vérifie que le fichier est revenu.
ls journal-securise.txt
```

Note le résumé du plan de l'étape 5 (combien de `+` / `~` / `-` ?) et commente : **d'où Terraform sait-il que ce fichier « lui appartenait » ?**

### Étape 4 — Sauvegarder et protéger le registre

```bash
# 8. Sauvegarde manuelle du state dans un fichier daté (tu verras que le backend fera mieux).
terraform state pull > sauvegarde-state-$(date +%F).tfstate

# 9. Crée (ou ouvre) le fichier .gitignore du projet.
nano .gitignore
```

Ajoute ces trois lignes (une phrase dans tes notes : **pourquoi chacune ?**) :

```
.terraform/
*.tfstate
*.tfstate.*
```

### Étape 5 — Écrire la configuration du backend (sans l'activer)

Crée le fichier `backend.tf` avec ce contenu — **ne le lance pas** (il sera activé en Leçon 5, avec tes clés AWS) :

```hcl
# Déclare le backend distant : le state sera stocké dans S3 (Bloc 6, Leçon 4).
terraform {
  backend "s3" {
    bucket         = "MON-NOM-unique-de-compartiment"   # le compartiment S3 qui hébergera le state
    key            = "atelier-securise/terraform.tfstate"  # le "chemin" du fichier dans le compartiment
    region         = "eu-west-3"                        # ta région (Bloc 6)
    dynamodb_table = "terraform-locks"                  # la table du verrouillage (2 apply ne peuvent pas se marcher dessus)
    encrypt        = true                               # chiffre le state au repos (il contient des secrets !)
  }
}
```

> 💡 **Pourquoi ne pas l'activer maintenant ?** Le backend S3 exige des clés AWS configurées et un compartiment créé — ce sera le travail de la Leçon 5. Ici, tu **écris la configuration** et tu comprends chaque ligne.

Questions de synthèse (dans tes notes) :
1. Que se passe-t-il si deux collègues lancent `terraform apply` **au même moment** avec un state local ? Et avec le backend + verrou ?
2. Pourquoi le state ne doit-il **jamais** aller dans Git ? Donne deux raisons.

---

## Livrable

`notes-exercice-03.md` : les 3 réponses d'inspection (étape 2), le résumé du plan + l'explication du drift (étape 3), les 3 lignes de `.gitignore` commentées (étape 4), le fichier `backend.tf` écrit (étape 5), et les 2 questions de synthèse.

Correction détaillée dans **`03-correction.md`**.