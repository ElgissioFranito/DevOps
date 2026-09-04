# Projet — Script de diagnostic & rapport

> **Bloc 3 · Leçon 7 (projet final)** — Exercice à faire en autonomie.
> Contexte : on te confie un serveur. Tu dois écrire un script `diagnostic.py` qui, lancé en une commande, vérifie l'état du serveur et sort un **rapport** clair. C'est l'application du **critère « bloc acquis »** de la roadmap : *un script qui vérifie un serveur, l'espace disque, un service, analyse les logs et envoie un rapport.*
> Objectif bonus : produire aussi un **rapport JSON** (pour la partie YAML/JSON de la leçon 4).

---

## 🎯 Objectif du projet

Écrire `diagnostic.py` qui :

1. **check_server** — affiche l'uptime (`uptime`).
2. **check_disk** — affiche le pourcentage d'espace disque de `/`, alerte si > seuil (défaut 80).
3. **check_service** — vérifie qu'un service (`nginx`) est actif via `systemctl is-active --quiet`.
4. **check_logs** — compte `ERROR`/`WARNING` dans un fichier de log donné en argument.
5. **main** — orchestre, affiche le rapport horodaté, et retourne `0` (tout OK) ou `1` (au moins un problème).

---

## 📋 Étape 1 — Structure

Crée `~/mon-diagnostic/diagnostic.py` avec :

- le shebang `#!/usr/bin/env python3` ;
- un docstring ;
- les imports (`subprocess`, `datetime`, `sys`, `pathlib.Path`) ;
- un compteur global `PROBLEMES = 0` ;
- une fonction `log(niveau, message)` qui affiche `[horodatage] niveau message` (aligné sur 9 caractères).

---

## 📋 Étape 2 — Les fonctions de vérification

Implémente :

```python
def run(cmd): ...                    # subprocess (stylé) : retourne (returncode, stdout.strip())
def check_server(): ...              # uptime
def check_disk(seuil=80): ...       # df -h /
def check_service(nom): ...          # systemctl is-active --quiet
def check_logs(chemin_log): ...      # compte ERROR/WARNING, gere fichier absent
```

Chaque fonction incrémente `PROBLEMES` (avec `global PROBLEMES`) quand elle détecte un problème.

---

## 📋 Étape 3 — le `main`

Crée une fonction `main()` qui :

- appelle `check_server()`, `check_disk(seuil=80)`, `check_service('nginx')`, `check_logs('/var/log/syslog')` (ou un autre log existant) ;
- à la fin : si `PROBLEMES > 0` → affiche le total et `sys.exit(1)`, sinon `sys.exit(0)` ;
- déclare le point d'entrée `if __name__ == '__main__': main()`.

---

## 📋 Étape 4 — Bonus JSON (optionnel mais recommandé)

En plus du rapport texte, écris un fichier `rapport.json` contenant un dict avec les champs `date`, `statuts`, `problemes`. Utilise le module `json` (vu en leçon 5) :

```python
import json
rapport_js = {"date": horodatage, "problemes": PROBLEMES}
Path("rapport.json").write_text(json.dumps(rapport_js, indent=2))
```

---

## 📋 Étape 5 — Tests

```bash
cd ~/mon-diagnostic
python3 diagnostic.py

echo $?    # doit etre 0 ou 1 selon l'etat
cat rapport.json   # si bonus réalise
cat /etc/logrotate.conf > /dev/null 2>&1  # un log realiste ?
```

Teste en changeant le service (ex. un service inexistant `zzz`) pour vérifier qu'il passe à `1` :

```bash
# temporairement : check_service('zzz') dans le script
python3 diagnostic.py ; echo $?   # doit etre 1
```

---

## 📋 Étape 6 — Auto-vérification

1. Pourquoi `global PROBLEMES` est nécessaire dans les fonctions qui l'incrémente ?
2. Pourquoi `systemctl is-active --quiet` plutôt que `systemctl status` ?
3. Comment calcules-tu le pourcentage du disque à partir de `df -h /` ?
4. Pourquoi retourner `1` si un problème est détecté ? Comment la CI/cron le lit-il ?
5. Quelle est la différence, dans la pratique, entre un script « de colle de commandes » (Bash) et un script Python comme celui-ci ?

---

## 🏁 Rendu attendu

Un `diagnostic.py` fonctionnel qui produit un rapport textuel (et JSON si bonus), testé dans les 2 états (OK et problème), avec tes 5 réponses écrites.

> Compare ensuite avec `03-correction.md`.
