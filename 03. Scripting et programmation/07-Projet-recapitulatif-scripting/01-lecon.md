# Leçon 7 — Projet : script de diagnostic & rapport

> **Bloc 3 · Scripting et programmation** — Leçon 7 sur 7 (projet récapitulatif)
> Cette leçon est **le projet de fin de bloc** : tu assembles tout ce que tu as appris (Bash + Python + YAML/JSON) pour créer un outil de diagnostic système réel. La roadmap définit ainsi le critère « bloc acquis » : *créer un script qui vérifie un serveur, l'espace disque, un service, analyse les logs et envoie un rapport*.
> C'est le **projet fil rouge** de ce bloc : chaque brique a été utilisée dans les leçons 1 à 6.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon (et donc du bloc 3), tu seras capable de :

1. **Concevoir** un script d'automatisation complet et **découper** le problème en fonctions.
2. **Vérifier l'état d'un serveur** (uptime, charge) et **l'espace disque** avec des commandes (`uptime`, `df`) via Python/`subprocess`.
3. **Contrôler un service système** (`systemctl is-active`) et en rendre compte.
4. **Analyser des logs** (compter les `ERROR`/`WARNING`, extraire info) à partir d'un fichier de log.
5. **Générer un rapport** consolidé (texte + optionnel JSON) avec horodatage.
6. **Produire un code de sortie** cohérent (0 = sain, 1 = problème détecté) exploitable en CI/CD.

---

## 2. Explication simple

### 2.1 Le « pourquoi » : un script de diagnostic, l'outil n°1 de l'admin

Quand un serveur « ne va pas bien », au lieu d'ouvrir 10 terminaux pour tout vérifier à la main, tu écris **une seule commande** qui te sort un **rapport clair** :

```
./diagnostic.sh all
```

Résultat attendu :

```
[2026-09-04 10:00] OK       Serveur : up 2 days, 4 users, load 0.3
[2026-09-04 10:00] Attention Espace disque : / 78% utilise
[2026-09-04 10:00] OK       Service nginx : actif
[2026-09-04 10:00] PROBLEME Logs : 3 ERROR, 12 WARNING
```

C'est le genre d'outil qu'on garde en production, qu'on planifie (`cron`), et qu'on branché sur une alerte. Cette leçon reprend **le critère exact de la roadmap** : vérifier serveur → disque → service → logs → rapport.

### 2.2 Le « comment » : l'architecture en petites fonctions

Plutôt qu'un gros script monolithique (illisible), on découpe en **fonctions dédiées**, chacune retournant un résultat qu'on agrège :

```
main()
 ├── check_server()   -> uptime / load / nb users
 ├── check_disk()     -> espace disque (df)
 ├── check_service()  -> systemctl is-active
 ├── check_logs()     -> comptage ERROR/WARNING
 └── ecrire_rapport() -> consolide + retourne le code de sortie
```

Chaque fonction affiche `OK / Attention / PROBLEME` + un message. À la fin, on **agrège** : si au moins un `PROBLEME` → `exit 1`, sinon `exit 0`. C'est ce code de sortie que la CI/CD lira.

### 2.3 Le « quand » : pourquoi ce périmètre précis ?

La roadmap cible 5 vérifications qui couvrent l'essentiel d'un diagnostic de base d'un serveur Linux :

| Vérification | Commande | Ce que ça révèle |
|--------------|----------|------------------|
| Serveur | `uptime` | la machine est-elle vivante ? charge ? |
| Disque | `df -h` | y a-t-il encore de la place ? |
| Service | `systemctl is-active` | le service critique tourne-t-il ? |
| Logs | `grep ERROR` | y a-t-il des anomalies récentes ? |
| Rapport | (consolidation) | synthèse lisible + code de sortie |
---

## 3. Exemples concrets

### 3.1 Le squelette et les helpers

```python
#!/usr/bin/env python3
"""diagnostic.py — verifie un serveur, l'espace disque, un service,
analyse les logs et produit un rapport."""

import subprocess, datetime, sys
from pathlib import Path

PROBLEMES = 0   # compteur global de problemes detectes

def log(niveau, message):
    """Affiche une ligne de rapport horodatee."""
    print(f"[{datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')}] {niveau:9s} {message}")

def run(cmd):
    """Execute une commande, retourne (returncode, stdout)."""
    r = subprocess.run(cmd, capture_output=True, text=True)
    return r.returncode, r.stdout.strip()
```

> 💡 Le helper `run()` centralise `subprocess.run([...], capture_output=True, text=True)` (vu en leçon 6) pour le réutiliser.

### 3.2 `check_server` — l'uptime

```python
def check_server():
    """Verifie que le serveur est vivant (uptime)."""
    global PROBLEMES
    code, out = run(['uptime'])
    if code == 0 and out:
        log('OK', f'Serveur : {out}')
    else:
        log('PROBLEME', 'Serveur injoignable (uptime en echec)')
        PROBLEMES += 1
```

### 3.3 `check_disk` — l'espace disque

```python
def check_disk(seuil=80):
    """Alerte si l'espace disque de / depasse un seuil (%)."""
    global PROBLEMES
    code, out = run(['df', '-h', '/'])
    if code != 0:
        log('PROBLEME', 'df a echoue')
        PROBLEMES += 1
        return
    ligne = out.splitlines()[-1]       # derniere ligne = /
    # ex: "tmpfs  200M  80M  120M  40% /"
    pct = int(ligne.split()[-2].rstrip('%'))
    niveau = 'OK' if pct < seuil else 'Attention'
    if pct >= seuil:
        PROBLEMES += 1
    log(niveau, f'Espace disque : / {pct}% utilise')
```

### 3.4 `check_service` — un service système

```python
def check_service(nom):
    """Verifie qu'un service systemd est actif."""
    global PROBLEMES
    code, _ = run(['systemctl', 'is-active', '--quiet', nom])
    if code == 0:
        log('OK', f'Service {nom} : actif')
    else:
        log('PROBLEME', f'Service {nom} : INACTIF')
        PROBLEMES += 1
```

### 3.5 `check_logs` — analyse des journaux

```python
def check_logs(chemin_log):
    """Compte les ERROR et WARNING dans un fichier de log."""
    global PROBLEMES
    p = Path(chemin_log)
    if not p.exists():
        log('Attention', f'Log absent : {chemin_log}')
        return
    texte = p.read_text()
    erreurs = sum(1 for l in texte.splitlines() if 'ERROR' in l)
    warnings = sum(1 for l in texte.splitlines() if 'WARNING' in l)
    log('OK', f'Logs {p.name} : {erreurs} ERROR, {warnings} WARNING')
    if erreurs > 0:
        PROBLEMES += erreurs
```

### 3.6 Le `main` qui orchestre et fixe le code de sortie

```python
def main():
    check_server()
    check_disk(seuil=80)
    check_service('nginx')
    check_logs('/var/log/mon-app/error.log')   # adapte au chemin reel

    if PROBLEMES > 0:
        log('PROBLEME', f"Fin : {PROBLEMES} probleme(s) detecte(s)")
        sys.exit(1)
    log('OK', 'Toutes les verifications sont OK')
    sys.exit(0)

if __name__ == '__main__':
    main()
```

> 💡 `if __name__ == '__main__':` est le point d'entrée standard Python : le script ne s'exécute que lancé directement, pas à l'import.
---

## 4. Bonnes pratiques modernes (2025-2026) — pour un vrai outil

- **Un point d'entrée unique + fonctions dédiées** : on garde le script lisible, testable fonction par fonction, maintenable. Évite le « script spaghetti ».
- **`systemctl is-active --quiet` plutôt que `systemctl status`** : ne renvoie qu'un code d'échec/succès, parfait pour un `if` (vu en leçon 2).
- **`df -h` + parsing de la dernière ligne** : on ne parse que ce qu'il faut (`rstrip('%')`, `int`), sans réinventer un parser complet.
- **Code de sortie explicite (`sys.exit(0/1)`)** : exploitable par un cron ou une pipeline CI (le script « fail » quand un problème est détecté).
- **Logs horodatés + niveau (`OK / Attention / PROBLEME`)** : un rapport lisible par un humain **et** greppable par une machine.
- **Seuils configurables** : `check_disk(seuil=80)` plutôt qu'un 80 en dur dans la fonction — testable et réglable.
- **Rester Bash si c'est juste de la « colle de commandes »**, Python dès qu'on manipule des données/logique (decider au cas par cas, vue en leçon 5).
- **Versionner le script dans Git** et le planifier avec `cron` (ou un systemd timer) pour de la vérification régulière, couplé à une alerte.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | Pourquoi c'est un problème | ✅ Version correcte |
|----------------|------------------------------|---------------------|
| Un script monolithique d'un seul bloc | Illisible, impossible à tester unitairement. | Découper en fonctions (`check_server`, `check_disk`...). |
| Afficher les logs mais oublier le **code de sortie** | La CI/cron croit que tout va bien même en cas de problème. | `PROBLEMES` → `sys.exit(1)` si détecté, `0` sinon. |
| Utiliser `systemctl status` (verbeux) | Sortie longue et paginée, inadaptée à un script. | `systemctl is-active --quiet` (code de retour). |
| Parser `df` en supposant une colonne fixe | Les formats varient (espace noms, locales). | Cibler la ligne voulue et extraire la colonne `%` (`split()[-2].rstrip('%')`). |
| `check_logs` sur un log absent → crash | `read_text()` sur fichier inexistant lève une exception. | Vérifier `p.exists()` avant, loguer `Attention`. |
| Compteur `PROBLEMES` sans `global` | La modification dans une fonction n'a pas d'effet (portée locale). | Déclarer `global PROBLEMES` au début de chaque fonction qui l'incrémente. |
| `main()` jamais exécuté | Script silencieux si on oublie l'appel. | `if __name__ == '__main__': main()`. |

---

## 8. Checklist de validation

- [ ] Je découpe mon script en **fonctions dédiées** (`check_server`, `check_disk`, `check_service`, `check_logs`, `main`).
- [ ] Je vérifie l'**uptime** (`subprocess`/`uptime`) et je log `OK` ou `PROBLEME`.
- [ ] Je vérifie **l'espace disque** (`df -h /`) avec un **seuil** (`check_disk(seuil=80)`).
- [ ] Je vérifie un **service** avec `systemctl is-active --quiet`.
- [ ] J'**analyse des logs** (compte `ERROR`/`WARNING`) en gérant le fichier absent.
- [ ] Je produis un **rapport horodaté** avec niveaux `OK / Attention / PROBLEME`.
- [ ] Je retourne un **code de sortie** : `sys.exit(1)` si problème, `0` sinon.
- [ ] J'ai testé le script sur un vrai cas (ex. service arrêté) et vérifié le `echo $?`.

---

> 📖 Prochaine étape : réalise le **projet** dans `02-exercice.md`, puis compare avec `03-correction.md`.

> 💡 **Lien avec tes acquis** : c'est exactement ce que fera plus tard un système de monitoring (Prometheus, Grafana — bloc 12). Ici on le fait simplement, à la main, en script — avant de passer aux outils dédiés.