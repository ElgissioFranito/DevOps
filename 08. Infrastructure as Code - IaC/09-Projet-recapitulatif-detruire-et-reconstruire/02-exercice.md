# Exercice — Projet récapitulatif : détruire et reconstruire

> **Bloc 8 · Leçon 9** — **Le projet final du bloc.** Choisis ton scénario : **A (100 % local, gratuit)** ou **B (AWS free tier, garde-fous de la Leçon 5)**. La logique est identique : assembler → détruire → reconstruire depuis le code → chronométrer.

---

## Règle du jeu (à relire avant chaque étape)

1. **Tout le code doit être commité dans Git avant la destruction** (Bloc 4). Un fichier non commité = il n'existe pas.
2. Pendant la reconstruction, tu n'as le droit **qu'au dépôt + au plan de reprise** (`docs/plan-reprise.md`). Toute correction nécessaire pendant la reconstruction : corrige le **code ou le plan**, committe, recommence l'étape.
3. **Chronomètre chaque phase** : les durées vont dans le rapport (`docs/rapport-reprise.md`).

---

## Phase 0 — Assembler le dépôt (30 min environ)

```bash
# Crée la structure complète du dépôt (Leçon 4 : 1 module = 1 responsabilité).
mkdir -p bibliotheque-iac/{terraform,ansible/roles/serveur_web/{tasks,handlers,templates,defaults},docs}
cd bibliotheque-iac

# Initialise Git (Bloc 4) et crée le .gitignore (Leçon 3 : le state et les plugins hors Git).
git init
echo -e ".terraform/\n*.tfstate\n*.tfstate.*\n*.tfvars" > .gitignore
```

Remplis ensuite, à partir de tes leçons (c'est le rappel utile : tu réutilises du code **que tu as déjà écrit**) :

- `terraform/` : les 5 fichiers du scénario choisi (Leçon 2-4 en A, Leçon 5 en B) ;
- `ansible/` : `inventory.ini`, `site.yml` et le rôle `serveur_web` complet (Leçon 7) ;
- `docs/plan-reprise.md` : le plan en 6 étapes (Leçon 8, Étape 4 de la correction) **avec tes commandes exactes** ;
- `README.md` : 5 lignes — à quoi sert le dépôt, comment l'exécuter.

Puis **le premier commit** :

```bash
git add -A && git commit -m "Stack IaC complet : Terraform + Ansible + plan de reprise"
```

**Vérification de départ** : le stack est reconstruisible **avant** même d'être détruit. Teste-le une première fois (plan complet, apply, vérification) : si ça ne marche pas en état « normal », il ne marchera pas en sinistre.

## Phase 1 — Le sinistre (5-10 min)

```bash
# Chrono en main. Le sinistre : détruire TOUT ce que Terraform a créé.
cd terraform
terraform destroy        # tape yes → tout disparaît

# Vérification du désastre (scénario A : plus de fichiers générés ;
# scénario B : console AWS vide — sauf le backend et la table de verrou).
```

> 📌 **Scénario B — complication volontaire (optionnelle, pour les braves)** : simule la perte du state en le remplaçant par une version du compartiment S3 **antérieure** à l'apply (versionnage du compartiment, Leçon 5). Constat : les ressources « fantômes » exigent une réparation du state — note l'expérience dans le rapport, c'est le cas réel le plus instructif. (Si tu préfères simple : saute cette complication.)

## Phase 2 — La reconstruction (le chrono tourne : c'est le RTO)

Exécute **exclusivement** les étapes de `docs/plan-reprise.md`, dans l'ordre, en chronométrant :

```
Étape 1 : terraform apply          → l'infrastructure renaît
Étape 2 : ansible-playbook site.yml → les machines sont reconfigurées
Étape 3 : (scénario B) restaurer les données de la base si nécessaire
Étape 4 : re-pointer les applications (outputs → variables)
Étape 5 : vérifier le service (curl / tests)
Étape 6 : noter la durée TOTALE
```

**Outil du pont Terraform → Ansible** (l'étape 4, en pratique) : l'inventaire Ansible a besoin de l'IP (ou de l'endpoint) produite par Terraform. Génère-le depuis les outputs :

```bash
# terraform output -json : les outputs au format JSON (Bloc 3, Leçon 4).
# jq : le petit outil qui lit/filtre du JSON (-r = valeur brute).
cd terraform && terraform output -json ip_publique | jq -r .
```

(Installe `jq` si besoin : `sudo apt install -y jq`.) Pour le scénario A, même principe avec les outputs du projet local.

## Phase 3 — Le rapport (20 min)

Écris `docs/rapport-reprise.md` avec ce gabarit :

```markdown
# Rapport de reprise — <date>
## Chronos
| Phase | Durée | Notes |
|-------|-------|-------|
| terraform apply | ?? min | |
| ansible-playbook | ?? min | |
| restauration base (B) | ?? min | |
| **TOTAL (RTO réel)** | ?? min | objectif : <= 30 min |
## Erreurs rencontrées et corrections
- (chaque erreur, et ce qui a été corrigé dans le code/le plan + commit)
## Conclusion
- Le critère de la roadmap est-il prouvé ? (oui/non + pourquoi)
```

**Commite le rapport** — c'est le livrable final du bloc :

```bash
git add -A && git commit -m "Rapport de reprise : RTO reel mesure"
```

---

## Livrable

1. Le dépôt `bibliotheque-iac/` complet (code + plan + rapport), propre dans Git.
2. Le rapport avec le **RTO réel mesuré** et les corrections effectuées.
3. La checklist de validation cochée (dans le rapport).

La correction pas à pas (avec les chronos attendus et les pièges du jour) est dans **`03-correction.md`**.