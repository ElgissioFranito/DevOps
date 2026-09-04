# Leçon 1 — Le cycle de vie d'une application (SDLC)

> **Bloc 1 · Bases du SDLC** — Leçon 1 sur 4
> *(Début de bloc : pense à lire d'abord l'introduction `00-Introduction-Bloc.md`.)*
> Ce premier module pose les fondations : tu vas comprendre ce qu'est le **cycle de vie d'un logiciel**, pourquoi il existe, et comment il s'articule avec le DevOps.

---

#### 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Définir le SDLC** (Software Development Life Cycle) avec tes propres mots, sans jargon.
2. **Lister et expliquer** les 8 phases du cycle de vie d'une application, dans l'ordre.
3. **Distinguer** le SDLC (cycle de vie) du DevOps (culture + outils autour de ce cycle).
4. **Donner une analogie** simple pour faire comprendre chaque étape à un néophyte.
5. **Expliquer pourquoi** chaque étape existe et ce qui se passe si on la saute.
6. **Décrire** le parcours global d'une application, du besoin jusqu'à la maintenance.

---

#### 2. Explication simple

##### Le « pourquoi » : pourquoi existe-t-il un cycle de vie ?

Imagine que tu construis une maison. Tu ne poses pas le toit avant les fondations, n'est-ce pas ? Tu passes par des étapes ordonnées : comprendre le besoin (une maison de 3 pièces ?), dessiner les plans, bâtir, vérifier la solidité, puis emménager et entretenir.

Un logiciel, c'est pareil. Le **SDLC** est le **chemin obligatoire** qu'une application suit de sa naissance (une idée) jusqu'à sa fin de vie (elle est remplacée ou retirée). Il existe pour **structurer le travail**, **limiter les erreurs** et **garantir la qualité**.

> 💡 **DevOps en une phrase** : si le SDLC est *le parcours*, le **DevOps** est la *façon de conduire et d'automatiser ce parcours* (outils, automatisation, collaboration, feedback rapide). On ne peut pas faire de DevOps sans comprendre le SDLC.

##### Les 8 phases du cycle de vie

Un logiciel passe par **8 phases**, dans l'ordre. C'est le fil rouge que tu retrouveras dans toutes les leçons suivantes (et dans ton exercice). Pas une de plus, pas une de moins — retiens bien ce nombre :

```
Besoin
  ↓
Analyse du besoin
  ↓
Conception
  ↓
Développement
  ↓
Tests
  ↓
Intégration
  ↓
Déploiement
  ↓
Production, Monitoring & Maintenance
```

> 🧠 **Astuce pour retenir le nombre** : on groupe en **4 blocs** → *avant* (Besoin, Analyse), *conception* (Conception), *code* (Développement, Tests, Intégration), *service* (Déploiement, Production/Monitoring). Ça reste **8 phases**.

> 🧠 **Pourquoi ton livre parle de 8 phases alors qu'on entend souvent « 7 étapes » ?** Beaucoup de cours listent les 7 étapes classiques : *Analyse → Conception → Développement → Test → Déploiement → Maintenance* (parfois + *Planification*). C'est **le même cycle**, juste regardé de plus loin : on y **fusionne** « Tests + Intégration » dans une seule case, et « Production + Monitoring » dans « Maintenance ». Ici, on **sépare** l'*Intégration* pour bien montrer la différence entre « une brique marche seule » et « les briques marchent ensemble » — un point central du DevOps (l'Intégration Continue, qu'on abordera au fichier CI/CD). **Ce qui compte n'est pas le nombre, mais l'ordre du raisonnement**, identique dans les deux versions.

| Étape | Question qu'elle répond | Analogie (la maison) |
|-------|-------------------------|----------------------|
| **Besoin** | *Quel problème ?* Pour qui ? | On sait qu'on veut « une maison de 3 pièces, avec jardin » |
| **Analyse du besoin** | *Quoi exactement ?* Attentes précises | On écoute le client : « 3 pièces, jardin, garage » |
| **Conception** | *Comment ?* Architecture, choix techniques | On dessine les plans de la maison |
| **Développement** | *On écrit le code* | On construit les murs, on pose les fenêtres |
| **Tests** | *Est-ce que ça marche ?* | On vérifie que le toit tient, que les portes ferment |
| **Intégration** | *Est-ce que les pièces vont ensemble ?* | On raccorde électricité, plomberie, mur porteur |
| **Déploiement** | *On met en service* | On livre les clés, on emménage |
| **Production & Monitoring & Maintenance** | *Ça tourne toujours bien ?* | La maison est habitée ; on l'entretient, on répare, on ajoute une pièce |

##### Le « comment » : comment ça se passe concrètement ?

Dans la réalité moderne, ces étapes **ne sont pas un long fleuve tranquille**. On ne fait pas « tout l'analyse, puis tout le code, puis tous les tests ». On travaille par **petites boucles** : on prend une petite fonctionnalité, on l'analyse, on la code, on la teste, on la déploie, on vérifie, puis on recommence.

C'est ce qu'on appelle le **cycle DevOps (loop)**, souvent représenté comme un symbole infini *∞* :

```
Plan ► Code ► Build ► Test ► Release ► Deploy ► Operate ► Monitor
   ▲                                                              │
   └──────────────────────────────────────────────────────────────┘
```

- **Plan** : on y fait l'analyse du besoin et la conception.
- **Code / Build / Test** : le développement + tests.
- **Release / Deploy** : intégration + déploiement (on reviendra dessus en détail).
- **Operate / Monitor** : production + monitoring + maintenance (le feedback revient au début).

Le **quand** utiliser ce cycle ? **Toujours**, pour la moindre fonctionnalité, dès le premier jour du projet — pas seulement « à la fin » ou « en cas de souci ».

##### En résumé

- Le **SDLC** = le cycle de vie complet d'un logiciel (8 phases).
- Le **DevOps loop** = la façon moderne de parcourir ces étapes en boucle rapide et automatisée.
- Un DevOps **ne code pas que du code** : il comprend toutes les étapes pour faire circuler une application de l'idée à la production le plus vite et le plus sûrement possible.

---

#### 3. Exemples concrets

##### Exemple 1 — Le parcours d'une application (schéma de référence)

```text
👥 UN BESOIN        « Les clients veulent se connecter à leur espace. »
      ↓
📋 ANALYSE          On note : qui ? (client), quoi ? (login/mot de passe),
                    pourquoi ? (voir ses commandes). On rédige une User Story.
      ↓
📐 CONCEPTION       On choisit : une app web, base de données, API d'authentification.
      ↓
💻 DÉVELOPPEMENT    On écrit le code de la page de connexion.
      ↓
🧪 TESTS            On vérifie : le bon mot de passe fonctionne, le mauvais est refusé.
      ↓
🔗 INTÉGRATION      On assemble le code de connexion avec le reste de l'app.
      ↓
🚀 DÉPLOIEMENT      On installe l'app sur le serveur de production.
      ↓
🏠 PRODUCTION       Les vrais clients se connectent.
      ↓
🩺 MONITORING       On surveille : « tout le monde se connecte sans erreur ? »
```

##### Exemple 2 — Visualiser les acteurs du cycle

Chaque étape est portée par des acteurs (dans une équipe « DevOps » moderne, les rôles se mélangent fortement) :

```text
Analyse  →  Product Manager / Product Owner
Conception → Architecte / Développeur senior
Développement → Développeur
Tests → QA (Assurance Qualité) / Développeur
Intégration & Déploiement → DevOps / SRE
Monitoring → DevOps / Ops / SRE
```

> 🧠 **Jargon** : **SRE** = Site Reliability Engineer (« ingénieur fiabilité des sites »), un rôle qui applique le génie logiciel aux opérations. **QA** = Quality Assurance (assurance qualité).

##### Exemple 3 — Question que tu dois savoir poser à chaque étape

Un bon réflexe de DevOps est de se demander, pour chaque étape : **« comment je le fais tourner en boucle ? »** Par exemple :

- Analyse : comment puis-je avoir **rapidement** le retour d'un réel utilisateur ? *(→ petites User Stories)*
- Code : comment puis-je **fusionner mon code souvent** sans casser celle des autres ? *(→ Git, bloc 4)*
- Tests : comment automatiser les vérifications ? *(→ Leçon 3)*
- Déploiement : comment **automatiser** la mise en production ? *(→ CI/CD, bloc 11)*
- Monitoring : comment **remonter automatiquement** les problèmes aux développeurs ? *(→ bloc 12)*

---

#### 4. Bonnes pratiques modernes (2025-2026)

Ce qui se fait aujourd'hui dans l'industrie pour « parcourir » le SDLC efficacement :

1. **Travailler en petites boucles (itérations courtes)** plutôt qu'en « tout ou rien ». On déploie **souvent et petit** plutôt que **rarement et gros**.
2. **Intégrer les tests le plus tôt possible** (on dit « shift-left ») : on teste dès le développement, pas à la fin.
3. **Automatiser tout ce qui est répétitif** : build, tests, déploiement (la philosophie du futur bloc CI/CD).
4. **Ecouter le retour des utilisateurs et du monitoring** pour alimenter le début du cycle (le feedback ferme la boucle). C'est le cœur de l'approche **DevOps / SRE**.
5. **Documenter** chaque étape (README, runbook) pour que n'importe qui puisse prendre le relais.

> 🧠 **Jargon** : **Shift-left** = déplacer les vérifications « vers la gauche » du cycle (vers le début, là où c'est moins cher de corriger). **Runbook** = document décrivant comment faire tourner et dépanner une application.

---

#### 5. Pièges à éviter

| ❌ Anti-pattern | ⚠️ Pourquoi c'est dangereux | ✅ Version correcte |
|----------------|------------------------------|----------------------|
| **Sauter l'analyse du besoin** et foncer dans le code | On construit une fonctionnalité que personne ne veut. Gaspillage total. | Rédiger une User Story claire et valider le besoin avec un utilisateur/Product Owner avant de coder. |
| **Sauter les tests** : « on verra en production » | Un bug coûte 100× plus cher en production que lors du développement. | Tester en continu, et automatiser les vérifications dès le développement (shift-left). |
| **Tout déployer d'un coup à la fin du projet** (grosse mise en production) | Si ça casse, on ne sait pas quoi qui a cassé. Retour en arrière impossible. | Déployer en petites versions fréquentes et sûres. |
| **Négliger le monitoring** : « ça marche chez moi » | Une app peut planter en production sans que personne ne le voie. | Mettre en place la surveillance dès le premier déploiement (bloc 12). |
| **Considérer le SDLC comme une formalité administrative** | On le vit comme de la paperasse : c'est justement ce qui ralentit. | Le voir comme le **garde-fou** qui rend le travail prévisible et sûr. |

---

#### 6. Exercice pratique

> ⚠️ L'exercice détaillé et autocontrôlé se trouve dans **`02-exercice.md`**. La correction commentée est dans **`03-correction.md`**. Lis bien **cette leçon avant de passer à l'exercice.**

**Énoncé court** : prends une fonctionnalité simple de ton choix (ex. : « un panier d'achat », « une alerte email », « la recherche d'un produit »). Rédige pour chaque étape du SDLC (analyse → maintenance) **une ou deux phrases** expliquant ce qui se passe pour cette fonctionnalité précise, comme dans l'exemple 3.1 ci-dessus. Termine par **un schéma de ta fonctionnalité** du « besoin » au « monitoring ».

Tu peux le faire sur papier, dans un fichier `.md`, ou dans un simple document texte. **L'important n'est pas l'outil mais le raisonnement.**

---

#### 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. Voici l'essentiel du raisonnement attendu.

En prenant l'exemple du **panier d'achat** :

- **Analyse** : « En tant que client, je veux ajouter des produits à un panier afin de commander en une fois. » → on précise les besoins (ajouter, modifier, supprimer, total).
- **Conception** : on choisit une décomposition : une classe `Panier`, un endpoint API pour ajouter un produit, une table stockant les lignes du panier.
- **Développement** : on code l'ajout d'un produit et le calcul du total.
- **Tests** : on vérifie que `Paniervide + produit = 1 ligne`, que le total est correct, que `0 produit` est refusé si besoin.
- **Intégration** : on branche le panier à la page produit et à la base de données.
- **Déploiement** : on met la nouvelle version en ligne.
- **Production** : les vrais clients utilisent le panier.
- **Monitoring** : on surveille les erreurs d'ajout au panier et le temps de réponse.

> ✅ **Critère de réussite** : si tu es capable de remplir ces cases pour ta fonctionnalité **sans hésiter**, tu as validé la leçon.

---

#### 8. Checklist de validation

Coche chaque case que tu réussis :

- [ ] Je sais définir le **SDLC** et expliquer le rôle de chacune de ses **8 phases**.
- [ ] Je sais **ordonner** les étapes du cycle de vie.
- [ ] Je sais **distinguer** SDLC (le cycle) et DevOps (la façon de l'exécuter).
- [ ] Je sais **donner une analogie** (maison, restaurant…) pour chaque étape.
- [ ] Je peux **expliquer pourquoi** on ne doit sauter l'analyse ni les tests.
- [ ] Je peux **décrire le parcours** d'une application du besoin à la maintenance avec mes propres mots.
- [ ] J'ai réalisé **l'exercice** sur une fonctionnalité de mon choix et vérifié ma correction.

---

🧭 **Pont vers la suite** — Tu connais maintenant les **8 phases** du cycle de vie d'une application. Mais dans la vraie vie, un projet ne démarre pas par « coder » : il commence par **organiser** ce qu'on va faire. C'est exactement l'objet de la Leçon 2 : comment on **planifie** et **priorise** le travail (le backlog) avant de coder.

---

*Prochaine étape :* Leçon 2 — **Planification et backlog** dans `02-Planification-et-Backlog/`.