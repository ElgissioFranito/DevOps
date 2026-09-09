# Exercice — Leçon 2 : Adressage IP et diagnostic réseau

> **Bloc 5 · Leçon 2** — Exercice à faire en autonomie.
> **Objectif** : diagnostiquer un réseau comme un pro : se connaître soi-même, monter couche par couche, et raisonner sur les plages d'adresses.

---

## Contexte

Un collègue te dit : « de mon poste, mon app Angular appelle le backend Spring Boot sur `localhost:8080` et ça marche, mais un ami depuis **l'extérieur** ne joint rien. » Tu vas reproduire le diagnostic réseau de ton côté et apprendre à raisonner en IP/plages.

---

## Énoncé

> 📌 **Options utilisées** : `ip addr show` / `ip route show` = afficher IP, masque et routes ; `ping -c 4` = envoyer **4** paquets puis s'arrêter (`-c` = count/nombre) ; `traceroute -m 5` = limiter à **5** sauts ; `curl -4` = forcer l'IPv4.

### Étape 1 — Connaître sa machine
- `ip addr show` : relève ton adresse, ton masque (ex. `/24`), ton interface (wlan0/eth0/…).
- `ip route show` : relève ta **gateway par défaut** (ex. `192.168.1.1`).
- Note ces infos dans `notes-exercice-02.md`.

### Étape 2 — Tester la liaison locale
- `ping -c 4 127.0.0.1` (localité).
- `ping -c 4 192.168.1.1` (ou ta gateway) — mesure la latence.
- `ping -c 4 8.8.8.8` (Internet, IP de Google DNS).

### Étape 3 — Tracer l'itinéraire
- `traceroute -m 5 example.com` (ou `tracepath`).
- Repère les sauts : combien de routeurs avant d'arriver.

### Étape 4 — IP publique
- `curl -4 https://api.ipify.org` pour obtenir ton IP publique.
- Compare avec ton IP privée de l'étape 1 : sont-elles différentes ?

### Étape 5 — Raisonner sur les plages
Pour chacune des adresses suivantes, dis **si** elle est dans la plage indiquée :
1. `192.168.1.100` est-elle dans `192.168.1.0/24` ?
2. `10.0.5.9` est-elle dans `10.0.0.0/8` ?
3. `172.16.0.1` est-elle dans `172.16.0.0/12` ?
4. `8.8.8.8` est-elle dans `192.168.1.0/24` ?

---

## Livrable

`notes-exercice-02.md` avec les sorties + les 4 réponses de l'étape 5.
La correction est dans **`03-correction.md`**.