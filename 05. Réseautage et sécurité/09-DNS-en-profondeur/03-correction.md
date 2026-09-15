# Correction 9 — Enquête DNS

> **Bloc 5 · Leçon 9 (complément)** — Correction pas à pas de `02-exercice.md`.

## ✅ Étape 1 — Cartographier (réponses types, les valeurs exactes évoluent)

```bash
dig +short github.com A
# 140.82.121.3  (parfois plusieurs lignes : plusieurs IP = redondance et répartition)
dig +short github.com NS
# dns1.p08.nsone.net. ... (les serveurs autoritaires de GitHub, chez leur fournisseur DNS)
dig +short github.com TXT
# "v=spf1 ip4:192.30.252.0/22 include:_spf.google.com ~all"  ← SPF (anti-usurpation d'emails)
```

**Lecture** : plusieurs enregistrements A = le trafic peut aller vers plusieurs machines (résilience). Le TXT `v=spf1…` déclare **quels serveurs ont le droit d'envoyer des mails** se réclamant de ce domaine — un serveur de mail qui en reçoit un « de github.com » venant d'ailleurs le refuse ou le met en spam.

## ✅ Étape 2 — Les alias

```bash
dig +short www.wikipedia.org
# wikipedia.org.          ← www est un CNAME vers le domaine racine
# 208.80.154.224          ← puis le A du racine
```

**Chaîne** : `www.wikipedia.org` (CNAME) → `wikipedia.org` (A) → `208.80.154.224`.
**Intérêt du CNAME** : si l'IP racine change, on ne modifie **qu'un enregistrement** au lieu de tous les alias (`www`, `mobile`, `api`…). C'est le principe DRY (Don't Repeat Yourself) appliqué au DNS. Rappel du piège de la leçon : ce raccourci est **interdit sur le domaine racine lui-même**.

## ✅ Étape 3 — TTL et cache

Les deux résolveurs donnent **le même TTL d'origine** (ex. 3600), mais la valeur affichée **diminue** : chaque résolveur a mis la réponse en cache à un instant différent, et son compte à rebours n'a pas le même reste. C'est la preuve vivante que « la propagation » est en réalité **une expiration de caches indépendants**.

## ✅ Étape 4 — Le chemin complet

```bash
dig +trace wikipedia.org | tail -n 20
```

Résumé attendu :
1. Les **serveurs racine** (`.`) sont interrogés → ils désignent les serveurs du TLD `.org`.
2. Le TLD `.org` désigne les **serveurs autoritaires** de `wikipedia.org`.
3. Le serveur autoritaire répond : l'adresse finale.

## ✅ Étape 5 — Réponses scénarios de production

5. **Changement d'IP non visible** : c'est le **cache des résolveurs** qui garde l'ancienne valeur jusqu'à expiration du TTL (3600 s = 1 h). Vérifications :
```bash
dig +short monapp.com A @ns1.mon-fournisseur-dns    # la SOURCE est-elle à jour ? (si non → erreur de saisie chez le registrar)
dig +short monapp.com A                              # le CACHE est-il à jour ? (si non → attendre l'expiration du TTL)
```
Si la source est à jour et le cache pas : rien à réparer, tout sera convergé **au plus tard à 15h00** (14h00 + 1 h de TTL). Prévention : baisser le TTL à 300 s *la veille* d'une migration.

6. **« Could not resolve host »** — diagnostic dans l'ordre :
```bash
dig +short monapp.com                # 1. Le DNS public résout-il ? (oui → problème local)
dig +short monapp.com @1.1.1.1       # 2. Un autre résolveur ? (isole ton résolveur local)
cat /etc/resolv.conf                 # 3. Quel DNS ta machine utilise-t-elle ? (fichier de configuration locale)
```
Ordre logique : le problème est-il dans la **zone DNS** (source), dans le **résolveur** (cache/local), ou dans la **machine** (config) ? On élimine de la source vers le client — même logique de cascade que DNS → IP → port → service (Leçon 1).

## 🧠 Conseils pour la suite

- Réflexe migration : **TTL à 300 la veille**, bascule, vérification source puis cache, **remontée du TTL** après stabilisation.
- Garde `dig +trace` pour les cas bizarres (réponses incohérentes entre fournisseurs) : il montre la vérité, pas les caches.
- 🧭 Maintenant que le nom et l'adresse sont maîtrisés, place au **contenu des réponses** : codes de statut HTTP et authentification (Leçon 10).

## ✅ Checklist de validation

- [ ] J'explique racine → TLD → autoritaire → cache, et je sais le montrer avec `dig +trace`.
- [ ] Je distingue A, AAAA, CNAME, MX, TXT, NS avec un usage pour chacun.
- [ ] Je lis une sortie `dig` (réponse, TTL, serveur interrogé).
- [ ] J'explique pourquoi deux résolveurs affichent des TTL différents.
- [ ] Je diagnostique un changement DNS invisible (source puis cache) et un « Could not resolve host ».
