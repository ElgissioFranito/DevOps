# Leçon 2 — Planification et backlog

> **Bloc 1 · Bases du SDLC** — Leçon 2 sur 4
> 🧭 **Pont depuis la Leçon 1** : la Leçon 1 t'a donné le *parcours* d'une application (les 8 phases du cycle de vie), mais pas la façon de **choisir quoi faire en premier**. Cette leçon transforme ce parcours en **liste de travail organisée** : le backlog, les User Stories, les sprints, la priorisation et l'estimation.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Définir** ce qu'est un backlog et son rôle dans un projet logiciel.
2. **Écrire** une User Story correcte au format « En tant que… je veux… afin de… ».
3. **Distinguer** une User Story, une tâche et un bug.
4. **Expliquer** ce qu'est un sprint et la différence entre backlog produit et backlog de sprint.
5. **Prioriser** des éléments de backlog avec des méthodes simples (MoSCoW / RICE).
6. **Estimer** grossièrement la complexité d'un travail sans se perdre en précision.

---

## 2. Explication simple

### Le « pourquoi » : pourquoi planifier avant de coder ?

Dans la Leçon 1, on a vu que la première étape du cycle est l'**analyse du besoin**. Mais un projet a souvent **des dizaines, voire des centaines** de besoins. Impossible de tout coder d'un coup : il faut **organiser**, **classer** et **choisir quoi faire en premier**.

Le **backlog** est cette liste de travail à faire. C'est l'outil qui transforme une masse de besoins vagues en une **liste ordonnée et actionnable**.

> 💡 **Analogie** : le backlog, c'est la **liste de courses du projet**. Sans liste, tu remplis ton caddie n'importe comment, tu oublies des choses essentielles et tu achètes des trucs dont tu n'as pas besoin. Avec une liste bien priorisée, tu sais exactement quoi prendre et dans quel ordre.

### Le « comment » : les pièces du backlog

Un backlog n'est pas juste une liste de phrases. Chaque élément est **typé** pour qu'on sache quoi faire :

- **User Story** : une fonctionnalité vue du point de vue de l'utilisateur. Format : *« En tant que [rôle], je veux [action], afin de [bénéfice]. »*
- **Tâche (task)** : une sous-étape technique. Ex. : « créer la table SQL du panier ».
- **Bug** : un défaut à corriger. Ex. : « le total n'est pas à jour après suppression d'une ligne ».
- **Épic** : une grosse fonctionnalité qu'on découpera en plusieurs User Stories.

Exemple d'organisation :

```text
BACKLOG PRODUIT
├── [User Story] En tant que client, je veux ajouter un produit au panier afin de commander en une fois.
│       └── (découpée en tâches : modèle, base, endpoint, écran)
├── [User Story] En tant que client, je veux voir le total du panier afin de vérifier ma commande.
├── [Bug] Le total n'est plus à jour après une suppression.
└── [User Story] En tant qu'administrateur, je veux exporter la liste des commandes en Excel.
```

### Le « comment » : sprint et priorité

- **Sprint** : une période courte et fixe (souvent 1 à 4 semaines) pendant laquelle l'équipe s'engage à livrer un sous-ensemble du backlog. À la fin du sprint, on doit avoir des **résultats utilisables**.
- **Backlog produit (product backlog)** : toute la liste, à long terme.
- **Backlog de sprint (sprint backlog)** : seulement les éléments choisis pour le sprint en cours.

**Prioriser** = décider ce qu'on fait en premier. Deux méthodes simples :
- **MoSCoW** : **M**ust have (indispensable), **S**hould have (important), **C**ould have (bonus), **W**on't have (pas maintenant).
- **RICE** (plus chiffrée) : priorité = `(Reach × Impact × Confidence) / Effort`. *Reach* = nb de personnes touchées, *Impact* = score 0-3, *Confidence* = 0-100 %, *Effort* = temps estimé.

**Estimer** = donner un ordre de grandeur de la difficulté, pas une date précise. On utilise souvent des **points** (1, 2, 3, 5, 8…) plutôt que des heures, pour rester relatif et simple.

### Le « quand » : quand planifie-t-on ?

- **En continu** : le backlog est vivant, on l'enrichit et le re-priorise en permanence.
- **Avant chaque sprint** : on sélectionne, on estime, on découpe.
- **Pendant le sprint** : on ne déplace plus les priorités (sinon le sprint n'a plus de sens).

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) |
|---|---|
| **Backlog** | liste ordonnée de tout ce qu'il faut faire sur un projet |
| **User Story** | besoin écrit au format « En tant que… je veux… afin de… » |
| **Tâche** | unité de travail concrète (plus petite qu'une story) |
| **Bug** | défaut : le logiciel ne se comporte pas comme prévu |
| **Sprint** | période fixe (ex. 2 semaines) pendant laquelle l'équipe livre un lot |
| **Priorité** | l'ordre d'importance des éléments du backlog |
| **Estimation** | approximation de l'effort nécessaire (jours, points) |
| **MoSCoW** | méthode de priorisation : Must / Should / Could / Won't |

---

## 3. Exemples concrets

### Exemple 1 — Écrire une vraie User Story

Format « En tant que / je veux / afin de » :

```text
❌ « Le panier »  → trop vague, on ne sait ni pour qui ni pourquoi.
✅ « En tant que client, je veux supprimer un produit de mon panier afin de corriger ma commande. »
```

La bonne User Story :
- **un acteur** (client),
- **un verbe d'action** (supprimer),
- **un bénéfice** (corriger ma commande).

### Exemple 2 — Priorisation MoSCoW sur un début de projet

```text
BACKLOG PRIORISÉ (MoSCoW)
[M] Ajouter un produit au panier         → sans ça, pas de vente : indispensable
[M] Commander mon panier                 → cœur du métier
[S] Voir l'historique de mes commandes   → utile, après le minimum
[C] Recevoir un email de confirmation    → bonus, appréciable
[W] Thème sombre                        → on garde en tête, pas maintenant
```

### Exemple 3 — Estimation en points (Fibonacci-like)

```text
1 point  : petite et évidente (ex. corriger une faute d'orthographe)
2 points : simple (ex. ajouter un champs)
3 points : moyenne (ex. nouvelle page basique)
5 points : complexe (ex. paiement en ligne)
8 points : très grosse (à découper en plusieurs stories !)
```

> 🧠 **Jargon** : **Points de story** = unité relative de complexité. Deux stories de 2 points ≈ même effort, ce qui permet de comparer sans donner de dates fausses.

---

## 4. Bonnes pratiques modernes (2025-2026)

1. **Écrire des User Stories orientées valeur utilisateur**, courtes (le critère « INVEST ») — pas des descriptions techniques codées.
2. **Tenir le backlog en ordre permanent** (grooming / refinement régulier), pour que le prochain sprint soit toujours prêt.
3. **Découper les gros éléments (épics) en petites stories** livrables en quelques jours.
4. **Imposer le WIP limité** : ne pas travailler sur dix choses à la fois, mais en terminer peu, bien, jusqu'au bout.
5. **Automatiser la priorité avec des métriques** (RICE, impact vs effort) plutôt que par intuition seule.
6. **Traiter les bugs sérieusement** mais les trier : tous ne bloquent pas le sprint.

> 🧠 **Jargon** : **INVEST** = acronyme pour une bonne User Story : **I**ndependent, **N**egotiable, **V**aluable, **E**stimable, **S**mall, **T**estable. **Grooming** / **refinement** = moment dédié pour nettoyer et détailler le backlog. **WIP** = Work In Progress (travail en cours).
---

## 5. Pièges à éviter

| ❌ Anti-pattern | ⚠️ Pourquoi c'est dangereux | ✅ Version correcte |
|----------------|------------------------------|----------------------|
| Écrire des User Stories vagues (« le panier ») | Personne ne sait quoi faire ni comment le tester → début de conflits. | Format complet « En tant que… je veux… afin de… » avec des critères vérifiables. |
| Prioriser à l'intuition sans critère | On livre des fonctionnalités peu importantes et on rate les urgentes. | Utiliser MoSCoW ou RICE avec des critères explicites. |
| Estimer en heures précises | Les estimations heures sont presque toujours fausses → fausses promesses. | Estimer en **points** relatifs (1,2,3,5,8). |
| Surcharger le sprint | Mauvaise qualité, dette, surmenage. | Livrer moins d'éléments mais les **terminer réellement** (scope limité). |
| Changer les priorités en plein sprint | Le sprint perd son sens, rien n'est fini. | Figer le sprint une fois qu'il a commencé ; déplacer les nouvelles idées au sprint suivant. |
| Un élément backlog de 40 h non découpé | Impossible à estimer, tester et livrer proprement. | Découper en petites User Stories de quelques jours max. |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`** et la correction dans **`03-correction.md`**.

**Énoncé court** : pour ta fonctionnalité de la Leçon 1 (le panier d'achat, par exemple, ou une autre), construis :
1. Un **backlog produit** avec **au moins 4 éléments** (mix User Stories + au moins 1 tâche technique + 1 bug).
2. Les User Stories rédigées au bon format **« En tant que… je veux… afin de… »**.
3. Une **priorisation MoSCoW** de l'ensemble.
4. Une **estimation en points (1-2-3-5-8)** de chaque élément, avec une phrase pour justifier.
5. Un **choix pour un sprint d'une semaine** (combien d'éléments et pourquoi).

---

## 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**.

**Essentiel du raisonnement attendu** :

- Un backlog est un **document vivant**, jamais figé.
- Toute User Story doit décliner les 3 parties (acteur / action / bénéfice).
- La **priorisation MoSCoW** place en [M] ce sans quoi le produit ne fonctionne pas (ajouter au panier, commander), en [S]/[C] les commodités, en [W] les idées reportées.
- L'**estimation en points** doit être **relative** : comparer les éléments entre eux, pas donner des dates.
- Le **sprint** doit regrouper un nombre d'éléments **faisable**, de préférence cohérents (ex. la boucle « ajouter → commander »), et **terminables** dans la durée.

---

## 8. Checklist de validation

Coche chaque case que tu réussis :

- [ ] Je sais **définir** un backlog et son rôle.
- [ ] Je sais **écrire** une User Story au bon format, complète, avec les 3 parties.
- [ ] Je sais **distinguer** User Story, tâche, bug et épic.
- [ ] Je sais **expliquer** un sprint et la différence backlog produit / backlog de sprint.
- [ ] Je sais **prioriser** avec MoSCoW (et connaître l'idée de RICE).
- [ ] Je sais **estimer** en points relatifs (1-2-3-5-8) et justifier.
- [ ] J'ai **construit un backlog complet** dans `02-exercice.md` et vérifié ma correction.

---

🧭 **Pont vers la suite** — On a décidé *quoi* faire (le backlog) et *dans quel ordre* (les sprints). La suite logique : **vérifier que ce qu'on livre fonctionne** et **choisir où le faire tourner**. La Leçon 3 relie donc le travail planifié aux **tests** et aux **environnements** (dev / staging / prod) — sans ces garde-fous, on livrerait n'importe comment.

---

*Prochaine étape :* Leçon 3 — **Tests et environnements** dans `03-Tests-et-Environnements/`.