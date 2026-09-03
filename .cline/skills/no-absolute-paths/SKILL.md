---
name: no-absolute-paths
description: Interdiction totale et absolue d'utiliser des chemins absolus dans TOUTES les commandes et TOUS les outils. Toujours utiliser des chemins relatifs uniquement.
---

# RÈGLE CRITIQUE - CHEMINS RELATIFS UNIQUEMENT

## Interdiction formelle
Tu n'as **JAMAIS** le droit d'utiliser un chemin absolu.

Interdit :
- Tout chemin contenant `/media/`, `/home/`, `/Users/`, `C:\`, etc.
- Tout chemin commençant par `/`
- Tout `find`, `grep`, `ls`, `cat`, `cd` ou autres commande avec un chemin absolu

## Obligation
- Utilise **uniquement** des chemins relatifs à la racine du projet.
- Le working directory est déjà la racine du projet. Tu n'as pas besoin de la connaître.

### Exemples corrects :
```bash
find . -type f -name "*.ts" -o -name "*.tsx" | head -100
grep -r "analytics" src/
ls src/app