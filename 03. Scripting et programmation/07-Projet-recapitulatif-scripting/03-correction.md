# Correction détaillée — Projet : script de diagnostic & rapport

> **Bloc 3 · Leçon 7 (projet final)** — Correction pas-à-pas du projet. Suis chaque étape et compare à ton script.

---

## ✅ Le script complet

```python
#!/usr/bin/env python3
"""diagnostic.py — Verifie serveur, disque, service, logs; produit un rapport."""

import subprocess
import datetime
import sys
import json
from pathlib import Path

PROBLEMES = 0


def log(niveau, message):
    """Affiche une ligne de rapport horodatee."""
    stamp = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')
    print(f"[{stamp}] {niveau:9s} {message}")


def run(cmd):
    """Execute une commande, retourne (returncode, stdout)."""
    r = subprocess.run(cmd, capture_output=True, text=True)
    return r.returncode, r.stdout.strip()


def check_server():
    global PROBLEMES
    code, out = run(['uptime'])
    if code == 0 and out:
        log('OK', f'Serveur : {out}')
    else:
        log('PROBLEME', 'Serveur injoignable (uptime en echec)')
        PROBLEMES += 1


def check_disk(seuil=80):
    global PROBLEMES
    code, out = run(['df', '-h', '/'])
    if code != 0:
        log('PROBLEME', 'df a echoue')
        PROBLEMES += 1
        return
    ligne = out.splitlines()[-1]                 # derniere ligne = /
    pct = int(ligne.split()[-2].rstrip('%'))
    if pct >= seuil:
        log('Attention', f'Espace disque : / {pct}% utilise')
        PROBLEMES += 1
    else:
        log('OK', f'Espace disque : / {pct}% utilise')


def check_service(nom):
    global PROBLEMES
    code, _ = run(['systemctl', 'is-active', '--quiet', nom])
    if code == 0:
        log('OK', f'Service {nom} : actif')
    else:
        log('PROBLEME', f'Service {nom} : INACTIF')
        PROBLEMES += 1


def check_logs(chemin_log):
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


def main():
    check_server()
    check_disk(seuil=80)
    check_service('nginx')
    check_logs('/var/log/syslog')

    # bonus : rapport JSON
    rapport_js = {
        'date': datetime.datetime.now().isoformat(),
        'problemes': PROBLEMES,
    }
    Path('rapport.json').write_text(json.dumps(rapport_js, indent=2))

    if PROBLEMES > 0:
        log('PROBLEME', f'Fin : {PROBLEMES} probleme(s) detecte(s)')
        sys.exit(1)
    log('OK', 'Toutes les verifications sont OK')
    sys.exit(0)


if __name__ == '__main__':
    main()
```

---

## ✅ Explication des choix techniques

- **`global PROBLEMES`** : sans lui, l'assignation `PROBLEMES += 1` à l'intérieur d'une fonction crée une variable **locale** qui n'a aucun lien avec le `PROBLEMES` global. `global` force la modification de la variable de niveau script.
- **`systemctl is-active --quiet`** : retourne `0` si actif, non nul sinon — idéal en condition. `systemctl status` est verbeux et paginé, inadapté.
- **`df -h /` + parsing** : on récupère la dernière ligne (`splitlines()[-1]`, celle de `/`), puis l'avant-dernière colonne (`split()[-2]` = « 78% ») dont on retire le `%`. Robuste à la plupart des locales importantes.
- **`sys.exit(1)`** : le code de sortie non nul est **lu par la CI/cron** (un `cron` peut alerter, une pipeline `&& ./diag` s'arrêtera). Sans lui, le script « réussirait » même sur problème.
- **`check_logs`** vérifie `p.exists()` avant `read_text()` pour ne pas lever `FileNotFoundError`.
- **Bonus JSON** : `json.dumps(rapport_js, indent=2)` sérialise le dict en un fichier lisible — lien direct avec la leçon 4.
- **`if __name__ == '__main__':`** : le `main` ne s'exécute que si le fichier est lancé directement.

---

## ✅ Résultat attendu (sur une machine où nginx est actif)

```
[2026-09-04 10:00:01] OK        Serveur :  10:00:01 up 2 days,  4 users,  load average: 0.30, 0.25, 0.20
[2026-09-04 10:00:01] OK        Espace disque : / 45% utilise
[2026-09-04 10:00:01] OK        Service nginx : actif
[2026-09-04 10:00:01] OK        Logs syslog : 0 ERROR, 1 WARNING
[2026-09-04 10:00:01] OK        Toutes les verifications sont OK
```

Avec un service inexistant (`check_service('zzz')`), la sortie se termine par :

```
[2026-09-04 10:00:02] PROBLEME  Service zzz : INACTIF
[2026-09-04 10:00:02] PROBLEME  Fin : 1 probleme(s) detecte(s)
```

et `echo $?` → `1`.

---

## ✅ Réponses de l'auto-vérification

1. **`global`** : une assignation dans une fonction crée une variable locale ; `global` fait que cette variable *est* le `PROBLEMES` du module. Sans lui, chaque fonction aurait son propre compteur qui serait perdu.
2. **`is-active --quiet`** : renvoie juste un **code de retour** (0/non nul), parfait pour un `if`. `status` affiche un pavé formaté pour humains (inutile en script).
3. **Pourcentage disque** : `df -h /` → on garde la ligne de `/`, on prend l'avant-dernière colonne (`%`), on retire le `%` et on `int()`.
4. **`sys.exit(1)`** : un code non nul signale un échec. En `cron`, on peut alerter ; en pipeline (`./diag && deploy`), le déploiement s'arrête. C'est le contrat « est-ce sain ? ».
5. **Bash vs Python** : Bash pour la « colle de commandes » (enchaîner des pipes/outils système, ultra léger) ; Python dès qu'on manipule des **données** (parse `df`, comptage logs), des **conditions** complexes, des **fichiers/JSON**, ou du **réseau/API**. Ici Python est plus lisible et testable.

---

## 📝 Checklist de validation (récapitulatif + conseils)

- [ ] Script découpé en **fonctions** dédiées (`check_server`, `check_disk`, `check_service`, `check_logs`, `main`).
- [ ] Vérification **uptime**, **disque** (avec seuil), **service** (`is-active --quiet`), **logs** (ERROR/WARNING).
- [ ] `global PROBLEMES` présent dans chaque fonction qui l'incrémente.
- [ ] Rapport **horodaté** avec niveaux `OK / Attention / PROBLEME`.
- [ ] **Code de sortie** `0` (OK) / `1` (problème) ; testé avec `echo $?`.
- [ ] Bonus : `rapport.json` généré via `json.dumps`.
- [ ] Testé dans les 2 états (service OK / service inexistant).

### 💡 Conseils pour la suite

- **Re-teste-toi** en changeant les services et le log à analyser (pense à des services réalistes : `nginx`, `postgresql`, `docker`).
- **Improvise** une amélioration : accepter le service/log en **argument** de ligne de commande (`sys.argv`) pour rendre le script réutilisable sans le rééditer.
- Ce script est le **prélude au monitoring** (bloc 12) : tu viens de construire manuellement les « metrics » qu'un Prometheus/Grafana centralisera plus tard.

---

🎉 **Félicitations — le bloc 3 « Scripting et programmation » est terminé !** Tu peux maintenant automatiser des tâches répétitives en Bash ET en Python, manipuler YAML/JSON, et produire des diagnostics exploitables.
