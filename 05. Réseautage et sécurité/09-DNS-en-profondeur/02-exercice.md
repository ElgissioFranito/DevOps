# Exercice 9 — Enquête DNS sur des domaines réels

> **Bloc 5 · Leçon 9 (complément)** — Exercice à faire en autonomie.
> Rappels : `dig` a été présenté en Leçon 1 (nom → IP). Toute option nouvelle est expliquée au premier usage. Note tout dans `notes-exercice-09.md`.

## Étape 1 — Cartographier un domaine

```bash
dig +short github.com A          # IPv4 (+short = réponse compacte)
dig +short github.com AAAA       # IPv6
dig +short github.com NS         # serveurs autoritaires
dig +short github.com MX         # serveurs de mail (souvent vides pour un site simple)
dig +short github.com TXT        # textes libres (SPF, preuve de propriété…)
```

1. Note chaque réponse. Que remarques-tu entre le résultat de `A` (une ou plusieurs IP ?) et ce que tu sais de github.com (un site très visité) ?

## Étape 2 — Suivre un alias (CNAME)

```bash
dig +short www.wikipedia.org     # www est-il un alias ? vers quoi ?
dig +short en.wikipedia.org CNAME
```

2. Dessine la chaîne : `www.wikipedia.org → ? → IP finale`. Explique l'intérêt d'un CNAME pour le propriétaire du site.

## Étape 3 — TTL et cache

```bash
dig wikipedia.org +noall +answer +comments
# +noall : tout masquer ; +answer : réafficher la réponse ; +comments : les commentaires (TTL, serveur)
dig @1.1.1.1 wikipedia.org +noall +answer          # interroge le résolveur de Cloudflare
dig @8.8.8.8 wikipedia.org +noall +answer          # interroge celui de Google
```

3. Note le **TTL** vu par chaque résolveur : sont-ils identiques ? Pourquoi peuvent différer (indice : compte à rebours du cache) ?

## Étape 4 — Le chemin complet

```bash
dig +trace wikipedia.org | tail -n 20    # +trace refait toute la hiérarchie ; tail -n 20 garde la fin
```

4. Résume en 4 lignes : racine → TLD → autoritaire → réponse finale.

## Étape 5 — Diagnostiquer comme en production

Réponds par écrit (scénarios) :

5. Tu changes l'IP de `monapp.com` chez ton registrar à 14h00 (TTL précédent : 3600). À 14h30, un collègue voit encore l'ancienne IP. **Que lui réponds-tu ?** Vérifie quoi, avec quelles commandes, et quand ça sera réglé ?
6. `curl https://monapp.com` échoue en « Could not resolve host ». Donne les 3 premières commandes de ton diagnostic, dans l'ordre, et ce que chacune prouve.
