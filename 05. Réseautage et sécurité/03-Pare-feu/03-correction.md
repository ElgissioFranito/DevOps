# Correction — Leçon 3 : Pare-feu et contrôle des flux

> **Bloc 5 · Leçon 3** — Correction pas à pas.

---

## 1. Installation et défaut

```bash
sudo apt install ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

## 2. Ouverture des ports

```bash
sudo ufw allow 22/tcp      # SSH — avant enable !
sudo ufw allow 80/tcp      # HTTP
sudo ufw allow 443/tcp     # HTTPS
sudo ufw allow 8080/tcp    # ton application de test
```

## 3. Blocage de la base

```bash
sudo ufw deny 5432/tcp     # PostgreSQL fermé publiquement
```

## 4. Activation

```bash
sudo ufw enable
```

## 5. Vérification

```bash
sudo ufw status numbered
# [ 1] 22/tcp            ALLOW IN  Anywhere
# [ 2] 80/tcp            ALLOW IN  Anywhere
# [ 3] 443/tcp           ALLOW IN  Anywhere
# [ 4] 8080/tcp          ALLOW IN  Anywhere
# [ 5] 5432/tcp          DENY  IN  Anywhere
ss -tulpn
```

## 6. Test depuis un autre terminal

```bash
nc -zv <ip> 8080   # Connected ! (app accessible)
nc -zv <ip> 5432   # timeout / refus (base filtrée)
```

Le test `nc 5432` ne répondant pas confirme que le pare-feu **filtre** (il ne dit pas « je suis là »).

---

## 7. Réponses au questionnement

**Pourquoi autoriser SSH avant d'activer le pare-feu ?**
> Si SSH est fermé au moment du `enable`, le pare-feu bloque ta connexion de gestion : tu deviens **enfermé dehors** du serveur distant (plus moyen d'y revenir, sauf console d'urgence du cloud).

**Quelle réflexion guide les ports autorisés ?**
> Le principe du **moindre privilège / moindre exposition** : n'ouvrir que les ports strictement nécessaires au service (22, 80, 443, 8080), et garder les données sensibles (5432) après le pare-feu, accessibles seulement côté interne.

---

## Checklist de validation (leçon 3)

- [ ] J'installe/règle UFW : défaut deny incoming, allow outgoing.
- [ ] J'ouvre 22, 80, 443, 8080 ; je bloque 5432.
- [ ] J'active et je vérifie `ufw status numbered`.
- [ ] Je teste avec `nc` les deux sens : ouvert / filtré.
- [ ] Je verbalise l'ordre critique (SSH avant enable) et le principe de moindre exposition.

---

## 🧠 Conseils pour la suite

- Sur une **VPS de prod**, ouvre uniquement 22/80/443 ; les ports d'app (8080) passent mieux via un reverse proxy (Leçon 5) que directement en public.
- Ne **jamais** désactiver UFW « pour tester » sans garder une session de secours ouverte.
- Relie ce principe aux **sécurité groups cloud** (Bloc 6) et aux **NetworkPolicies/Ingress** (Kubernetes, Bloc 10).