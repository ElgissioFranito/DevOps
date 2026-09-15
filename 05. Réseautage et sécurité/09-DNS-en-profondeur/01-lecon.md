# Leçon 9 — DNS en profondeur : enregistrements, TTL et diagnostic

> **Bloc 5 · Réseautage et sécurité** — Leçon complémentaire
> 🧭 **Pont depuis la Leçon 8** : tu sais lire une réponse TLS. Mais tout commence en amont par une question : **à quelle adresse pointe mon domaine ?** La Leçon 1 t'a fait utiliser `dig` ; cette leçon approfondit : **qu'est-ce qu'un enregistrement DNS, quels types existent, combien de temps il se met à jour (TTL)** — des questions que tu poseras chaque semaine en production.

---

## 1. Objectifs d'apprentissage

1. **Expliquer** le chemin d'une résolution DNS (résolveur → racine → TLD → serveur autoritaire).
2. **Distinguer** les enregistrements A, AAAA, CNAME, MX, TXT, NS et leur usage.
3. **Lire** la sortie de `dig` section par section.
4. **Comprendre** le TTL et pourquoi « la propagation DNS » est en réalité un problème de cache.
5. **Diagnostiquer** un problème DNS avec `dig` (`+short`, `+trace`, interroger un serveur précis).

## 2. Explication simple

**Le pourquoi** : quand tu changes l'IP d'un serveur, que tu bascules vers un load balancer (Leçon 6) ou que ton site est injoignable, la cause est DNS une fois sur deux. Sans comprendre les enregistrements et le TTL, tu seras « en attente de propagation » sans jamais savoir quand ça finira.

**L'analogie** : le DNS est un **annuaire téléphonique à étages**. Tu demandes à ton concierge (le **résolveur** de ton fournisseur). S'il ne connaît pas, il demande à l'accueil général (la **racine**), qui l'envoie au guichet des `.fr` (le **TLD**), qui l'envoie au **serveur autoritaire** du domaine — le seul qui fait autorité sur la réponse. La réponse est ensuite **mémorisée** (cache) pendant la durée indiquée par le **TTL**.

**Le comment — les enregistrements** sont des lignes de l'annuaire :
- **A** : nom → adresse IPv4. **AAAA** : nom → adresse IPv6.
- **CNAME** : nom → *autre nom* (« le site, c'est en fait la même chose que cette autre adresse »).
- **MX** : où livrer le courrier de ce domaine (serveurs de mail).
- **TXT** : texte libre — utilisé pour la preuve de propriété et l'anti-spam (SPF, vu plus bas).
- **NS** : quels serveurs font autorité sur ce domaine.

**Le quand** : tout changement DNS se fait via la **zone DNS** chez ton registrar/fournisseur ; le diagnostic se fait toujours avec `dig`.

## 📖 Vocabulaire / Abréviations

| Terme | Définition (1 ligne) |
|---|---|
| **DNS** | Domain Name System : l'annuaire nom → IP (Leçon 1) |
| **Résolveur** | serveur DNS qui fait les recherches pour toi (ex. ton FAI, 1.1.1.1, 8.8.8.8) |
| **Racine (root)** | sommet de l'arborescence DNS : sait où sont les TLD |
| **TLD** | Top-Level Domain : `.fr`, `.com`, `.org`… |
| **Serveur autoritaire** | serveur qui possède la réponse officielle pour un domaine |
| **Enregistrement** | une entrée de la zone DNS (A, CNAME, MX…) |
| **Zone DNS** | l'ensemble des enregistrements d'un domaine |
| **TTL** | Time To Live : durée (en secondes) pendant laquelle une réponse peut être gardée en cache |
| **Cache** | mémoire qui évite de reposer la même question pendant le TTL |
| **SPF** | enregistrement TXT listant les serveurs autorisés à envoyer du mail pour un domaine |

## 3. Exemples concrets

### 3.1 Les types d'enregistrements en action

```bash
dig +short example.com A          # l'adresse IPv4 (+short = réponse compacte)
dig +short example.com AAAA       # l'adresse IPv6
dig +short github.com CNAME       # un « alias » vers un autre nom (souvent vers un LB cloud)
dig +short gmail.com MX           # où livrer le mail du domaine
dig +short google.com TXT         # textes libres (preuve, SPF…)
dig +short example.com NS         # les serveurs autoritaires du domaine
```

### 3.2 Lire une réponse `dig` complète

```bash
dig example.com
```

```text
;; QUESTION SECTION:
;example.com.                    IN      A          # la question posée

;; ANSWER SECTION:
example.com.             3600    IN      A      93.184.216.34
#  ↑ le nom                ↑ TTL (secondes)   ↑ la réponse

;; Query time: 12 msec                    # durée de la réponse
;; SERVER: 1.1.1.1#53(1.1.1.1)            # quel résolveur a répondu
```

> 💡 Le **TTL (3600 = 1 h)** est l'information la plus stratégique : c'est combien de temps les résolveurs du monde entier garderont cette réponse en cache avant de re-demander.

### 3.3 Diagnostiquer

```bash
dig +trace www.example.com
# +trace : refait TOUT le chemin (racine → TLD → autoritaire) — pour voir OÙ ça casse

dig @8.8.8.8 www.example.com
# @serveur : interroger un résolveur PRÉCIS — compare les réponses (ton cache est-il à jour ?)

dig example.com +noall +answer +comments
# sortie allégée : seulement la réponse et les commentaires (TTL, serveur)
```

**Méthode de diagnostic « mon changement DNS n'est pas visible »** :
1. `dig +short mondomaine A @le-serveur-autoritaire` → la **source** est-elle à jour ? Si non : le problème est dans la zone (côté registrar).
2. `dig +short mondomaine A` (résolveur par défaut) → le **cache** est-il à jour ? Si non : c'est le TTL qui n'a pas expiré — rien à « réparer », il faut attendre.

## 4. Bonnes pratiques modernes (2025-2026)

- **Baisser le TTL (300 s = 5 min) AVANT une migration planifiée**, plusieurs heures à l'avance : le jour J, les caches expirent vite et la bascule est rapide. Le remonter ensuite (3600+).
- **A directement, CNAME avec parcimonie** : sur le sommet d'un domaine (`mondomaine.com`), un CNAME est interdit (norme DNS) — il existe des alternatives (ALIAS/ANAME chez certains fournisseurs).
- **Vérifier SPF/DKIM** sur tout domaine qui envoie des mails : sans enregistrement TXT SPF, tes mails tombent en spam.
- **Résolveurs publics fiables** pour tester (1.1.1.1, 8.8.8.8) — mais en production, laisse le système faire.
- **Jamais d'IP en dur** dans les configs applicatives (rappel Leçon 1) : passer par des noms DNS, comme ça une migration = un changement d'enregistrement, pas un redéploiement.

## 5. Pièges à éviter

| ❌ Anti-pattern | ⚠️ Pourquoi c'est dangereux | ✅ Version correcte |
|---|---|---|
| « Attendre la propagation » les yeux fermés | C'est du cache, pas de la propagation : chaque résolveur expire **à son rythme** selon le TTL. | Vérifier la source (autoritaire) puis le cache, avec `dig @serveur`. |
| TTL très bas (60 s) en permanence | Plus de requêtes pour les résolveurs, latence accrue, dépendance au serveur autoritaire. | TTL bas seulement autour d'une migration planifiée. |
| CNAME sur le domaine racine (`mondomaine.com`) | Interdit par la norme : les enregistrements requis (SOA…) cohabiteraient mal. | Enregistrement A direct (ou ALIAS si le fournisseur le propose). |
| Modifier le DNS sans noter l'ancienne valeur | Impossible de revenir en arrière en cas d'erreur. | Copier l'ancien enregistrement avant de changer (règle du Bloc 2 : sauvegarde avant modification). |
| Croire que `ping` suffit à tester le DNS | `ping` mélange DNS et connectivité ICMP : on ne sait plus ce qui casse. | `dig` pour le DNS seul, puis `ping`/`curl` pour la suite. |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`**, la correction dans **`03-correction.md`**.

**Énoncé court** : mène une enquête DNS complète sur deux domaines réels (ex. `github.com` et un domaine de ton choix) : A/AAAA/CNAME/MX/TXT/NS, TTL observés, chemin `+trace` résumé, comparaison de deux résolveurs (`@8.8.8.8` vs `@1.1.1.1`), et réponses à des questions de diagnostic type production.

## 7. Correction détaillée de l'exercice

> La correction complète est dans **`03-correction.md`**. Essentiel : lire le TTL comme une durée de cache, distinguer A (IP) de CNAME (alias), et la méthode source-puis-cache pour diagnostiquer un changement invisible.

## 8. Checklist de validation

- [ ] J'explique le chemin racine → TLD → serveur autoritaire → cache.
- [ ] Je distingue A, AAAA, CNAME, MX, TXT, NS et je cite un usage pour chacun.
- [ ] Je lis une sortie `dig` (question, réponse, TTL, serveur).
- [ ] Je compare deux résolveurs et j'explique une divergence de TTL/cache.
- [ ] J'applique la méthode de diagnostic d'un changement DNS invisible.

---

🧭 **Prochaine étape** — Tu sais résoudre le nom et lire les erreurs de transport. La prochaine leçon s'occupe du **langage des réponses HTTP** (codes de statut) et de la façon dont une application te reconnaît (authentification : sessions, JWT, OAuth2).

