# Référence rapide — Leçon 3 : Pare-feu (UFW)

> Bloc 5 · Leçon 3 — Aide-mémoire.

## Concepts
- **Pare-feu réseau** (L3/L4) : contrôle par IP / port / protocole.
- **WAF** (applicatif, L7) : inspecte le contenu des requêtes (pas abordé en profondeur ici).
- **Défaut-deny** : refuser l'entrant sauf explicitement autorisé.
- **Port ouvert vs filtré** : `ss` (écoute réelle) VS `ufw` (règle).

## UFW — commandes

| Besoin | Commande |
|--------|----------|
| Installer | `sudo apt install ufw` |
| Défaut deny entrant | `sudo ufw default deny incoming` |
| Défaut allow sortant | `sudo ufw default allow outgoing` |
| Autoriser un port | `sudo ufw allow 443/tcp` |
| Autoriser par IP | `sudo ufw allow from 192.168.1.0/24` |
| Bloquer | `sudo ufw deny 5432/tcp` |
| Activer | `sudo ufw enable` |
| Voir les règles | `sudo ufw status numbered` |
| Supprimer une règle | `sudo ufw delete N` (N = numéro) |

## Ordre critique
1. `allow 22/tcp` (SSH) **avant** `enable`.
2. `default deny incoming`.
3. `enable`.
4. Vérifier : `ufw status numbered` + `ss -tulpn`.

## Réflexes sécurité
- N'exposer que l'indispensable (moindre exposition).
- Ne jamais ouvrir une base de données (5432) en public.
- Restreindre SSH à une IP/VPN admin.