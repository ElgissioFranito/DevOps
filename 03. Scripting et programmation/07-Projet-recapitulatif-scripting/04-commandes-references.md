# Aide-mémoire — Projet : script de diagnostic & rapport

> **Bloc 3 · Leçon 7** — Fiche de référence du projet final.

## 📌 Squelette d'un script Python d'admin

```python
#!/usr/bin/env python3
import subprocess, datetime, sys, json
from pathlib import Path

PROBLEMES = 0

def log(n, m):
    print(f"[{datetime.datetime.now():%Y-%m-%d %H:%M:%S}] {n:9s} {m}")

def run(cmd):
    r = subprocess.run(cmd, capture_output=True, text=True)
    return r.returncode, r.stdout.strip()

def main():
    # ... verifications ...
    if PROBLEMES > 0:
        sys.exit(1)
    sys.exit(0)

if __name__ == '__main__':
    main()
```

## 📌 Vérifications clés

## Serveur (uptime)

```python
code, out = run(['uptime'])
log('OK', f'Serveur : {out}') if code == 0 else log('PROBLEME', 'echec')
```

## Disque (df)

```python
code, out = run(['df', '-h', '/'])
ligne = out.splitlines()[-1]
pct = int(ligne.split()[-2].rstrip('%'))  # ex. 78 -> '%'
```

## Service (systemctl)

```python
code, _ = run(['systemctl', 'is-active', '--quiet', 'nginx'])
if code == 0: log('OK', 'nginx : actif')
```

## Logs (comptage)

```python
erreurs = sum(1 for l in Path('/var/log/syslog').read_text().splitlines() if 'ERROR' in l)
```

## 📌 Rapport JSON (bonus, lien lecon 4)

```python
rapport = {'date': datetime.datetime.now().isoformat(), 'problemes': PROBLEMES}
Path('rapport.json').write_text(json.dumps(rapport, indent=2))
```

## 📌 Etape suivante (monitoring — bloc 12)

Ce que ce script calcule manuellement (uptime, disque, service, logs) sera plus tard collecté par des outils :

```
Script Python (cette lecon)
        ->  Prometheus (metrics)
        ->  Grafana (dashboards)
        ->  Alertmanager (alertes)
```

## 📌 Règles d'or du projet

- Toujours `global PROBLEMES` dans les fonctions qui modifient le compteur.
- `systemctl is-active --quiet` pour un test booléen, jamais `status`.
- `sys.exit(1)` si un problème, sinon `0` — c'est l'interface avec la CI.
- Vérifier `p.exists()` avant `read_text()` sur un log.
- Un script opérationnel = fonctions + point d'entrée `if __name__ == '__main__':`.
