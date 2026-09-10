# Exercice — Leçon 2 : Réseau cloud (VPC)

> **Bloc 6 · Leçon 2** — Exercice en autonomie. **Aucun compte AWS payant requis** : on travaille sur papier/dessin et avec un script local. (Les vraies commandes AWS viendront quand les clés seront créées à la Leçon 6.)

---

## Contexte

Une entreprise veut héberger son application : **frontend Angular** (fichiers statiques), **backend Spring Boot** (API) et **base PostgreSQL**. Toi, tu dois décider **où** placer chaque brique dans le réseau cloud et **pourquoi**.

---

## Énoncé

### Étape 1 — Dessiner l'architecture réseau

Dans `notes-exercice-02.md`, reproduis (schéma ASCII) le **trajet d'une requête** depuis Internet jusqu'à la base, en positionnant correctement ces composants :
`Internet → DNS → Internet Gateway (IGW) → Load Balancer → Public subnet → Private subnet → Application → Database`.

Indique pour **chacun** : **public** ou **privé**, et **pourquoi** en **1 phrase**.

### Étape 2 — Remplir le rôle de chaque brique

| Composant | Rôle (1 ligne) |
|-----------|----------------|
| VPC | ? |
| Subnet public | ? |
| Subnet privé | ? |
| Internet Gateway (IGW) | ? |
| NAT Gateway | ? |
| Security Group | ? |

### Étape 3 — Script Bash local (visualisation)

Crée `afficher-vpc.sh` reprenant l'exemple 3.2 de la leçon (l'affichage `🏢 Mon VPC…`), rends-le exécutable et lance-le :

```bash
nano afficher-vpc.sh        # crée le fichier
chmod +x afficher-vpc.sh    # le rend exécutable (permission vue au Bloc 2)
./afficher-vpc.sh           # l'exécute
```

Colle la sortie dans `notes-exercice-02.md`.

### Étape 4 — Réflexion (dans `notes-exercice-02.md`)

1. Que se passe-t-il si on place la **base PostgreSQL dans un subnet public** avec un security group ouvert à tout ?
2. Pourquoi demande-t-on souvent un **NAT Gateway** pour l'application privée, mais jamais pour la base ?

---

## Livrable

`notes-exercice-02.md` (schéma, tableau, sortie du script, 2 réponses) + le fichier `afficher-vpc.sh`.

Correction détaillée dans **`03-correction.md`**.