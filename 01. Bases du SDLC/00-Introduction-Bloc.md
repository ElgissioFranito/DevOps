# Introduction au Bloc 1 — Bases du SDLC

> **À lire en premier**, avant la Leçon 1. Ce fichier te dit :
> - de quoi parle ce bloc et **pourquoi tu en as besoin**,
> - ce que tu dois **savoir avant de commencer** (spoiler : peu de choses),
> - les 4 leçons du bloc et le **fil rouge** qui les relie,
> - le vocabulaire des **outils croisés** au passage (que tu n'as surtout **pas besoin** de maîtriser encore).

---

## 🎯 De quoi parle ce bloc ?

**SDLC** = *Software Development Life Cycle* = le **cycle de vie d'un logiciel** : toutes les étapes par lesquelles passe une application, de l'idée jusqu'à sa mise en production et sa maintenance.

C'est le **bloc fondateur** de tout ton parcours DevOps : avant de manipuler des serveurs, des outils de déploiement ou des pipelines, tu dois comprendre **le parcours** qu'une application suit. Tout le reste du livre (Linux, Docker, Kubernetes, CI/CD…) viendra automatiser ou améliorer une étape de ce parcours.

> 💡 **Un bloc 100 % conceptuel.** Tu n'as **aucun logiciel à installer** ici : pas de compilateur, pas de serveur, pas d'outil. Tu vas surtout **raisonner**. C'est voulu : les bases avant la technique.

---

## ✅ Prérequis (avant de commencer)

- **Aucun** : tu n'as pas besoin de savoir coder, ni d'avoir un outil installé.
- Seule compétence requise : **la curiosité** et une bonne lecture attentive.
- Une **fonctionnalité simple de la vie courante** à garder en tête (ex. le panier d'un site e-commerce) — elle servira de **fil rouge** à tout le bloc.

---

## 🗺️ Les 4 leçons du bloc (et le fil rouge)

Tout le bloc suit un **fil unique** : *« comment une fonctionnalité passe de l'idée à la production ? »*. On utilisera un exemple récurrent : **le panier d'achat d'un site e-commerce**.

| Leçon | Dossier | Question traitée |
|-------|---------|------------------|
| **Leçon 1** — Le cycle de vie d'une application (SDLC) | `01-Le-Cycle-de-Vie-d-une-Application/` | Quelles sont les **8 phases** par lesquelles passe une fonctionnalité ? |
| **Leçon 2** — Planification et backlog | `02-Planification-et-Backlog/` | Comment **organiser et prioriser** le travail avant de coder ? |
| **Leçon 3** — Tests et environnements | `03-Tests-et-Environnements/` | Comment **vérifier** que ça marche, et **où** le faire tourner ? |
| **Leçon 4** — Du Git à la production | `04-Du-Git-a-la-Production/` | Comment assembler **tout le parcours**, du code jusqu'au serveur ? |

Chaque dossier contient **trois fichiers** : `01-lecon.md` (la leçon), `02-exercice.md` (l'exercice en autonomie) et `03-correction.md` (la correction commentée).

---

## 🧠 Le vocabulaire des outils « croisés » (à ne pas maîtriser encore)

Au fil des leçons, tu vas **croiser** des noms d'outils et de technologies (Maven, npm, Spring Boot, Git…). **C'est normal et voulu** : on te les montre pour que le parcours soit concret. Mais tu **n'as pas besoin de les connaître** pour valider ce bloc.

Voici de quoi il s'agit, pour ne pas être surpris — et **quand tu les apprendras vraiment** :

| Outil / terme | C'est quoi ? (en 1 phrase) | Vrai apprentissage |
|----------------|----------------------------|--------------------|
| **Java / JavaScript** | Deux **langages de programmation** très répandus, pris en exemple ici. | Bloc 03 (scripting) |
| **Maven** | L'outil qui **compile** du code Java et produit l'**artifact**. | Bloc 03 |
| **npm / Node.js** | L'écosystème JavaScript pour exécuter du code et construire une app. | Bloc 03 |
| **Spring Boot / NestJS** | Des « cadres de travail » (frameworks) pour créer des apps web plus vite. | Bloc 03/04 |
| **Artifact** | Le **résultat du build** : un fichier (`.jar`, `dist/`…) prêt à déployer. | Bloc 1 (Leçons 3-4) |
| **Build** | L'action de **transformer le code source en artifact**. | Bloc 1 (Leçons 3-4) |
| **Git** | L'outil qui **enregistre l'historique du code** et permet de collaborer. | Bloc 04 |
| **Dépôt Git** | Le « coffre central » où vit le code versionné. | Bloc 04 |
| **CI / CD** | L'**automatisation** du build, des tests et du déploiement. | Bloc 11 |
| **Serveur** | Un ordinateur dédié qui fait tourner l'application en continu. | Bloc 02 (Linux) |
| **ssh / scp / curl** | Des commandes pour se connecter ou interroger des machines à distance. | Bloc 02 & 05 |

> ⚠️ **Ne te décourage surtout pas** si un nom revient sans avoir été défini avant. Chaque fois que c'est le cas, on te le signale et on te renvoie vers le bloc où tu l'apprendras. À ce stade, **comprendre le parcours suffit**.

---

## 🧭 Où mène ce bloc ? (et pourquoi la suite est Linux)

Après la Leçon 4, tu sauras **expliquer sans hésiter** comment ton code passe de ton ordinateur au serveur de production — c'est le critère de validation du bloc (voir la roadmap).

Mais pour **faire tourner réellement** une application, il faut un **système d'exploitation** sur ce serveur. C'est pourquoi la suite logique est le **Bloc 02 — Linux** : le langage des serveurs (99 % des serveurs dans le monde). C'est là que tu commenceras à **toucher du concret** (commandes, fichiers, utilisateurs), alors qu'ici tout reste conceptuel.

> 🎯 **Objectif de fin de bloc** : ne pas coder, mais **raconter le voyage** d'une application, de l'idée à la production, sans se tromper d'ordre.

---

*Démarre maintenant avec la **Leçon 1** dans `01-Le-Cycle-de-Vie-d-une-Application/`.*