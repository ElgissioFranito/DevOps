# Correction — Projet récapitulatif : détruire et reconstruire

> **Bloc 8 · Leçon 9** — Correction pas à pas, avec les chronos attendus et les pièges du jour J. Compare tes durées aux repères : ce qui compte, c'est la **mesure**, pas la performance.

---

## Phase 0 — Assembler et tester (avant le sinistre)

**Structure attendue** (Leçon 4 + Leçon 7 assemblées) :

```
bibliotheque-iac/
├── .gitignore                  (.terraform/, *.tfstate*, *.tfvars)
├── README.md
├── terraform/                  (versions, backend, variables, main, outputs)
├── ansible/                    (inventory.ini, site.yml, roles/serveur_web/)
└── docs/
    ├── plan-reprise.md         (les 6 étapes avec TES commandes)
    └── rapport-reprise.md      (à remplir après la reconstruction)
```

**Explications des points sensibles** :
- `.gitignore` créé **avant** le premier `git add` (Leçon 3) — sinon un `tfstate` se glisse dans l'historique et y reste pour toujours.
- Le **test de reconstruction « normale »** avant destruction est l'étape que les impatients sautent : si le stack ne se reconstruit pas en état sain, le sinistre n'apprendra rien de nouveau. (Chrono de cette pré-validation : ~10-15 min en A, ~30-40 min en B — RDS oblige.)

## Phase 1 — Le sinistre

```bash
cd terraform
terraform destroy     # tape yes
```

**Chrono attendu** : 1-2 min (A) / 8-15 min (B — la base RDS met plusieurs minutes à se supprimer : c'est normal, le chrono ne compte pas contre toi ici, ce n'est pas du temps de reprise).

**Vérification du désastre** : A — plus aucun fichier généré ; B — console AWS vide (sauf le backend + la table de verrou, **voulus**).

**La complication optionnelle (perte de state)** : si tu as restauré une version antérieure du state, tu as vu `terraform plan` annoncer des créations de ressources qui **existent déjà** — les ressources fantômes. La leçon au passage : le versionnage du backend (Leçon 5) est ta police d'assurance du state ; dans la vraie vie, on restaure le **dernier bon état** plutôt que de laisser les fantômes.

---

## Phase 2 — La reconstruction, étape par étape

**Étape 1 — Terraform (A : 2-3 min / B : 15-25 min)**

```bash
cd terraform
terraform init        # re-télécharge les providers + vérifie le backend
terraform plan        # lit le plan AVANT (réflexe permanent, Leçon 2)
terraform apply       # tape yes
```

**Piège n°1 attendu** : en B, une erreur `DBInstanceAlreadyExists` si tu relances trop vite après un destroy récent (la « période de veille » de RDS, Leçon 5). Solution : attendre 5-10 min, ou changer l'identifiant. **Si ça t'est arrivé : c'est une erreur à noter dans le rapport ET à ajouter au plan** (« étape 1 : attendre la purge RDS le cas échéant ») — c'est exactement de cette façon qu'un plan s'améliore.

**Étape 2 — Ansible (2-3 min)**

```bash
cd ../ansible
ansible-playbook site.yml -i inventory.ini --check   # simulation (Leçon 6)
ansible-playbook site.yml -i inventory.ini           # exécution
```

**Piège n°2 attendu** : l'inventaire contient l'**ancienne IP** (la nouvelle EC2 a une nouvelle IP publique — Leçon 5). C'est le piège n°1 de la reprise, annoncé en leçon (§ 3.2) : la correction est de remplir l'inventaire depuis `terraform output -json ip_publique | jq -r .` — et de l'écrire **dans le plan** (étape 4) pour la prochaine fois.

**Étape 3 — (B) La restauration des données** : le stack reconstruit sans ses données n'est qu'une demi-reprise. En B, si ton scénario incluait la destruction de la base **avec** des données, la restauration se fait depuis le snapshot/datadump (Bloc 7, Leçon 4) — et le nouveau mot de passe de la base vient du `random_password` regénéré : à re-transmettre à l'application via les variables (piège classique des reprises).

**Étape 5 — Vérification** :

```bash
curl http://<nouvelle-ip>     # la page attendue (A) / le service répond (B)
```

---

## Phase 3 — Le rapport et les chronos de référence

**Chronos de référence (à titre indicatif, pas à imiter absolument)** :

| Phase | Scénario A (local) | Scénario B (AWS) |
|-------|--------------------|------------------|
| Terraform apply | 2-3 min | 15-25 min (RDS) |
| Ansible playbook | 2-3 min | 2-5 min (+ délai SSH après la création EC2) |
| Restauration base | — | 5-10 min |
| **RTO réel total** | **5-6 min** | **25-40 min** |

**Lecture des chronos** : un RTO réel est rarement inférieur au chrono du test trimestriel de la Leçon 8 — c'est justement l'intérêt du rapport : il transforme le RTO « espéré » en RTO **mesuré**, et révèle les goulots (chez AWS : RDS ; en local : presque rien). Si ton total dépasse l'objectif, le plan en 6 étapes te dit **où** optimiser (automatiser le pont Terraform→Ansible, pré-créer la base depuis un snapshot…).

**Le rapport bien écrit contient** : les chronos par phase, chaque erreur avec sa correction **et le commit associé**, et la phrase de conclusion sur le critère d'acquis. Relis le gabarit de `02-exercice.md` — et commite.

---

## La rélecture du critère d'acquis

La roadmap clôt le bloc 8 ainsi : *« Tu peux supprimer ton infrastructure et la reconstruire de manière reproductible »*. Vérification, point par point :

- ✅ **Supprimer** : `terraform destroy` → infra vide, console vide (vérifiée).
- ✅ **Reconstruire** : infrastructure (Terraform) + configuration (Ansible) + données (backup) — depuis le code seul.
- ✅ **De manière reproductible** : le même plan rejouable (test n°2), par un tiers (test n°3), idempotent (`changed=0`), avec un RTO **mesuré**.

> 🎉 **Le Bloc 8 est complet.** Tu ne « connais » plus Terraform et Ansible : tu as **prouvé** — par un sinistre simulé, chronométré et documenté — que ton infrastructure est du code. C'est exactement ce que le Bloc 7 t'avait fait prouver pour les données (le runbook vécu) ; le Bloc 8 l'a étendu à toute l'infrastructure.

---

## Checklist de validation (projet final)

- [ ] J'ai un dépôt unique et propre : Terraform + Ansible + plan de reprise + README, tout commité.
- [ ] J'ai détruit intégralement mon infrastructure et la reconstruite **en suivant uniquement le plan**.
- [ ] La configuration Ansible s'est réappliquée idempotente (`changed=0` au 2ᵉ passage).
- [ ] J'ai mesuré le **RTO réel** et écrit le rapport (chronos + erreurs + corrections).
- [ ] Je peux refaire l'exercice le lendemain avec le même plan, sans aide.
- [ ] ✋ **Le critère de la roadmap est coché** : « Tu peux supprimer ton infrastructure et la reconstruire de manière reproductible. »

---

## 🧠 Conseils pour la suite

- **Rejoue ce projet chaque trimestre** (Leçon 8) : le chrono qui baisse est ta DR qui progresse.
- **Passe le dépôt en revue avant le Bloc 9** : le rôle Ansible accueillera Docker ; le CI/CD (Bloc 11) exécutera `terraform apply` + `ansible-playbook` — ton dépôt en est déjà la trame.
- **Les trois réflexes du bloc à garder à vie** : `plan` avant `apply` ; jamais de state dans Git ; tout changement passe par le code. Ce sont eux qui séparent un DevOps formé d'un technicien de clics.