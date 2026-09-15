# Exercice — Leçon 2 : Terraform, ton premier projet

> **Bloc 8 · Leçon 2** — Exercice en autonomie, **100 % local et gratuit** : tu vas écrire du vrai code HCL, l'appliquer, le modifier, puis tout détruire. Aucun compte cloud.

---

## Contexte

La leçon t'a fait **suivre** le workflow complet ; l'exercice te le fait **refaire en autonomie**, avec des variantes — c'est en écrivant les fichiers toi-même que le vocabulaire (`provider`, `resource`, `variable`, `output`) s'ancre. À la fin, tu sauras lire un `terraform plan` sans hésiter.

---

## Énoncé

> 📌 **Rappels d'options** : `mkdir` = créer un dossier ; `nano` = éditeur de texte (sauver : `Ctrl+O`, quitter : `Ctrl+X`) ; `terraform plan` = prévisualiser ; `terraform apply` = exécuter (confirmation : taper `yes`) ; `terraform destroy` = supprimer ce que le projet a créé.

### Étape 1 — Nouveau projet « atelier-securise »

```bash
# Crée le dossier du projet et entre dedans.
mkdir ~/atelier-securise && cd ~/atelier-securise
```

### Étape 2 — Écrire les 4 fichiers

Recrée la structure de la leçon, **par toi-même**, avec ces exigences :

1. `versions.tf` : mêmes providers que la leçon (`local` et `random`), versions épinglées.
2. `variables.tf` : trois variables —
   - `nom_projet` (string, défaut : `securise`) ;
   - `longueur_secret` (number, défaut : `20`) ;
   - `activer_journal` (bool, défaut : `true`).
3. `main.tf` : deux resources —
   - `random_string` nommée `secret` (longueur = `longueur_secret`, **avec** caractères spéciaux autorisés) ;
   - `local_file` nommée `journal` : crée le fichier `journal-<nom_projet>.txt` dont le contenu contient **sur 3 lignes** : le nom du projet, la longueur du secret, et le secret lui-même.
4. `outputs.tf` : deux outputs —
   - `secret_genere` (la valeur du secret) ;
   - `chemin_journal` (le chemin du fichier).

> 💡 **Aide** : le type « bool » accepte seulement `true` ou `false`. Pour la resource `local_file`, relis `main.tf` de la leçon : la structure est identique, seul le contenu change.

### Étape 3 — Exécuter le rituel complet

```bash
# 1. Prépare le projet (télécharge les providers).
terraform init

# 2. Prévisualise : compte les "+" et lis le résumé "Plan: X to add...".
terraform plan

# 3. Exécute (tape yes), puis vérifie le fichier créé avec ls et cat.
terraform apply

# 4. Re-applique sans rien changer : la sortie doit dire "No changes".
terraform apply

# 5. Change la valeur par défaut de longueur_secret (20 → 30), puis :
terraform plan    # lis la sortie : combien de "+" / "~" / "-" ?
terraform apply   # tape yes

# 6. Démonte tout proprement.
terraform destroy # tape yes
```

Note dans `notes-exercice-02.md` : la sortie du plan de l'étape 3.2 (le résumé), celle du re-apply (étape 3.4), et celle du changement de variable (étape 3.5).

### Étape 4 — Questions de lecture de plan (dans `notes-exercice-02.md`)

Que fait Terraform dans chaque cas ? Réponds en une ligne.

1. `~ local_file.journal will be updated in-place`
2. `- random_string.secret will be destroyed`
3. `+ local_file.journal will be created`

---

## Livrable

- Le dossier `~/atelier-securise/` avec les 4 fichiers `.tf` fonctionnels.
- `notes-exercice-02.md` : les 3 résumés de plan demandés + les 3 réponses aux questions.

Correction détaillée dans **`03-correction.md`**.