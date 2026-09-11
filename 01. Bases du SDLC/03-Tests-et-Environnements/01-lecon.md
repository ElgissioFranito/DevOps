# Leçon 3 — Tests et environnements

> **Bloc 1 · Bases du SDLC** — Leçon 3 sur 4
> 🧭 **Pont depuis la Leçon 2** : on a prévu *quoi* faire (le backlog) et *dans quel ordre* (les sprints). Cette leçon ajoute les **garde-fous** : **les tests** (pour vérifier que ça marche), **les environnements** (où l'on exécute l'application), et le **build** qui produit le livrable déployable.

---

## 1. Objectifs d'apprentissage

À la fin de cette leçon, tu seras capable de :

1. **Expliquer** la différence entre test unitaire, test d'intégration et test end-to-end (E2E).
2. **Décrire** la pyramide des tests et pourquoi on met « beaucoup en bas, peu en haut ».
3. **Distinguer** les environnements *développement*, *staging* et *production*, et leur rôle à chacun.
4. **Définir** ce qu'est un *build* et un *artifact* (avec des exemples Maven et npm).
5. **Expliquer** pourquoi on teste dans staging avant de déployer en production.

---

## 2. Explication simple

### Les tests : le « pourquoi »

Reprends la Leçon 1. Entre le développement et le déploiement, il y a une étape indispensable : **vérifier que ça marche**. On a vu qu'un bug coûte ~100× plus cher en production. Les tests sont cette **ceinture de sécurité** qui attrape les erreurs **avant** qu'elles n'atteignent les vrais utilisateurs.

> 💡 **Analogie** : tester, c'est comme vérifier une voiture avant un long trajet. On ne teste pas « tout le trajet » d'un coup : on vérifie chaque mécanisme séparément (les freins), puis on vérifie que freins + direction vont bien ensemble, puis on fait un test complet sur piste avant le grand voyage.

### Les 3 niveaux de tests

| Niveau | Qu'est-ce qu'on vérifie ? | Échelle | Analogie voiture |
|--------|---------------------------|---------|------------------|
| **Test unitaire** | **Une petite unité de code** isolée (une fonction, une classe) marche correctement. | Très petite, très rapide, très nombreux | Vérifier les freins seuls sur un banc d'essai |
| **Test d'intégration** | Que **plusieurs unités fonctionnent ensemble** (code + base + API). | Moyenne, moins rapide | Vérifier que le freinage et la direction agissent ensemble |
| **Test end-to-end (E2E)** | Le **parcours utilisateur complet**, du clic jusqu'au résultat final, sur l'app la plus proche de la réalité. | Grande, lente, peu nombreux, fragile | Rouler tout le trajet d'essai sur piste réelle |

### La pyramide des tests

Parce que les tests E2E sont **lents et fragiles** (et donc coûteux), on n'en fait pas des milliers. La règle du métier est la **pyramide** :

```text
        ▲   /  E2E  \          peu (gros scénarios, lents)
       /    /---------\
      /    / Intégration \     moins
     /    /---------------\    beaucoup de tests
    /    /  Unitaires       \   TRÈS nombreux (rapides)
   /    /---------------------\
```

- **Tout en bas** : beaucoup de petits tests unitaires rapides.
- **Au milieu** : des tests d'intégration.
- **En haut** : peu de tests E2E.

Si tes tests E2E sont les plus nombreux, c'est un signal d'alerte (pyramide inversée).

### Les environnements : dev, staging, production

En plus de *ton* ordinateur (où tu codes), on déploie l'application dans des environnements partagés :

- **Développement (dev)** : là où on code et teste vite, sans précaution particulière, données factices. On peut casser et recommencer.
- **Staging** (préproduction) : une copie **la plus proche possible de la production** (mêmes versions, mêmes réglages), mais avec des **fausses données**. C'est l'endroit où on fait les tests d'intégration et E2E avant de livrer.
- **Production (prod)** : l'environnement réel utilisé par les vrais utilisateurs, avec de vraies données. On y déploie **uniquement ce qui a été validé** sur staging.

```text
Développeur (PC) → Dev → Staging → Production
      code, tests rapides    tests finals      vrais utilisateurs
```

> 🧠 **Jargon** : **Staging** = « répétition générale » de la production, en privé, avec de fausses données. Son but est de **réduire au maximum le risque** de casser la production.

### Build et artifact

- **Build** : la transformation du code source en un **livrable exécutable/déployable**. On compile, on assemble, on optimise.
- **Artifact** : le **résultat** de ce build (un fichier `.jar`, `.war`, un dossier de build, un conteneur…).

```text
Code source            Build                    Artifact
java/main/...  ──►  Maven build        ──►  application.jar
src/...        ──►  npm run build      ──►  dist/ (fichiers web)
```

Un même code source, construit de façon reproductible, donne un **artifact identique** — c'est cette reproductibilité qui permet de passer de celui-ci en production sans surprise.

### Petit lexique des outils cités dans ce bloc

> ℹ️ **Tu n'as pas besoin d'installer ni de maîtriser ces outils maintenant.** Voici simplement de quoi on parle, pour que ces noms ne te surprennent pas quand tu les rencontreras :

| Outil / terme | C'est quoi ? | Tu l'apprendras vraiment… |
|----------------|--------------|----------------------------|
| **Java** | Un **langage de programmation** (avec JavaScript, l'un des deux exemples du parcours). | Bloc 03 |
| **Maven** | L'outil qui **compile** le code Java et produit l'**.jar** (l'artifact). | Bloc 03 |
| **Node.js / npm** | Node.js exécute du **JavaScript** ; npm en gère les outils et le build. | Bloc 03 |
| **Spring Boot / NestJS** | Des **frameworks** (cadres prêts à l'emploi) pour créer des apps web plus vite. | Bloc 03/04 |
| **`.jar` / `dist/`** | Deux **formes d'artifact** : un fichier exécutable Java / un dossier de fichiers web. | Bloc 1 (L3-L4) |
| **curl** | Commande pour **interroger une adresse web** depuis le terminal (ex. vérifier qu'une app répond). | Bloc 05 |
| **healthcheck** | Un « **contrôle de santé** » (ex. l'adresse `/health`) qui dit si l'app démarre bien. | Bloc 1 (L3-L4) |

---

## 📖 Vocabulaire / Abréviations

| Terme | Définition (une ligne) |
|---|---|
| **Test unitaire** | vérifie UNE fonction isolée (rapide, très nombreux) |
| **Test d'intégration** | vérifie que plusieurs briques fonctionnent ensemble |
| **Test end-to-end (E2E)** | vérifie un parcours utilisateur complet, de bout en bout |
| **Pyramide des tests** | beaucoup d'unitaires, moins d'intégration, peu d'E2E |
| **Environnement** | un « monde » d'exécution : dev, staging, production |
| **Staging** | copie proche de la production pour tester avant de livrer |
| **Build** | transformation du code en livrable exécutable |
| **Artifact** | le livrable produit par le build (`.jar`, image Docker…) |

---

## 3. Exemples concrets

> Les commandes **Java/Maven** et **Node** ci-dessous sont données à but pédagogique : tu n'as pas besoin de les exécuter pour comprendre. Elles sont **testables** si tu as l'environnement installé (voir blocs 02/03).

### Exemple 1 — Un petit test unitaire en Java (JUnit, style Spring/Maven)

```java
// PanierTest.java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class PanierTest {
    @Test
    void ajouterUnProduit_creeUneLigne() {
        Panier panier = new Panier();
        panier.ajouter("Café", 2, 5.0);
        assertEquals(1, panier.tailles(), "Une ligne doit exister");
        assertEquals(10.0, panier.total(), 0.001, "2×5 = 10");
    }
}
```

Lancer :
```bash
mvn test            # Maven : compile + exécute tous les tests unitaires
```

### Exemple 2 — Un test unitaire en Node (Jest, variante NestJS)

```javascript
// panier.test.js
const { Panier } = require("./panier");

test("ajouter un café crée une ligne et calcule le total", () => {
  const panier = new Panier();
  panier.ajouter("Café", 2, 5.0);
  expect(panier.total()).toBe(10.0);
});
```

Lancer :
```bash
npm test            # exécute Jest sur les fichiers *.test.js
```

### Exemple 3 — Build & artifact

```bash
# Java / Maven : construit un .jar déployable dans target/
mvn clean package
# → produit target/application.jar (l'ARTIFACT)

# Node / npm : construit un dossier de fichiers web dans dist/
npm run build
# → produit le dossier dist/ (l'ARTIFACT)
```

### Exemple 4 — Vue d'ensemble d'un mini pipeline Dev → Staging → Prod

```text
docker build -t mon-app:1.0 .     # construit l'image (artifact)
# → déploie en staging, on teste
curl -k https://staging.exemple.com/health   # vérifie que la santé est OK
# ✓ validé sur staging
# → déploie la MÊME artifact en production
```

> 🧠 **Note** : le déploiement automatisé en continu est le sujet du bloc **CI/CD** (bloc 11). Ici, on veut juste comprendre le **pourquoi** de chacun de ces environnements.

---

## 4. Bonnes pratiques modernes (2025-2026)

1. **Respecter la pyramide des tests** : majorité de tests unitaires, peu d'E2E.
2. **Tester le plus tôt possible (shift-left)** : écrire les tests **en même temps** que le code, voire d'abord (TDD).
3. **Automatiser les tests dans le pipeline CI** : chaque commit déclenche la montée des tests (bloc 11).
4. **Garder staging fidèle à la production** (mêmes versions, mêmes réglages) pour que le dernier test soit fiable.
5. **Ne jamais supprimer les tests** : ils sont la mémoire de ce qui doit continuer à marcher.
6. **Construire des artifacts reproductibles** (mêmes builds → mêmes résultats) — idéalement versionnés.

> 🧠 **Jargon** : **TDD** = Test-Driven Development (développement piloté par les tests) : on écrit le test qui échoue, puis le code qui le fait passer. **CI** = Continuous Integration (intégration continue) : vérifications automatiques à chaque changement de code.

---

## 5. Pièges à éviter

| ❌ Anti-pattern | ⚠️ Pourquoi c'est dangereux | ✅ Version correcte |
|----------------|------------------------------|----------------------|
| Ne faire que des tests E2E | Lent, fragile, très coûteux ; on teste trop peu, trop tard. | Faire beaucoup de tests unitaires rapides + des E2E ciblés seulement. |
| Tester en production directement | On fait subir les bugs aux vrais utilisateurs. | Tester sur staging fidèle, puis déployer seulement ce qui est validé. |
| Staging différent de la production (versions différentes) | Le test ne reflète pas la réalité → surprise en prod. | Garder staging et prod **alignés** (mêmes versions, mêmes configs). |
| Ne rien tester « on verra » | Coût 100× plus élevé en production. | Automatiser les tests dès le développement. |
| Build non reproductible (« ça marche chez moi ») | Artifacts différents selon les machines → bug imprévisible. | Build automatisé et reproductible depuis le code versionné. |

---

## 6. Exercice pratique

> ⚠️ L'exercice détaillé est dans **`02-exercice.md`** et la correction dans **`03-correction.md`**.

**Énoncé court** : à partir de ta fonctionnalité de la Leçon 1 (ex. : le panier d'achat), tu vas :

1. **Lister** une success-story de tests : **3 tests unitaires**, **1 test d'intégration**, **1 test E2E** → décris en une phrase (en français) ce que chacun vérifie.
2. Associer chaque niveau de test à **l'environnement** où il se passe naturellement (dev / staging / prod) et justifier.
3. Décrire le **build** de ta fonctionnalité : de quel code source on part, quelle commande (ex. `mvn clean package` ou `npm run build`), et **quel artifact** est produit.
4. **Expliquer** en 2-3 phrases pourquoi on passe par staging avant la production.

> Tu n'as pas besoin d'exécuter de code pour cet exercice : l'objectif est de **raisonner** sur les concepts.

---

## 7. Correction détaillée de l'exercice

> La correction complète pas-à-pas est dans **`03-correction.md`**. Essentiel du raisonnement montré en **Leçon 3** :

- **Test unitaire** : vérifie une unité de code isolée (ex. le calcul du total du panier).
- **Test d'intégration** : vérifie que plusieurs briques marchent ensemble (ex. code panier + base).
- **Test E2E** : vérifie un parcours utilisateur complet (ex. j'ajoute au panier → je vois le total → ça fonctionne à l'écran).
- Environnements : dev (tests rapides/unitaires), staging (intégration + E2E), prod (rien à tester !).
- Build/artifact : `mvn clean package` → `application.jar` ; `npm run build` → `dist/`.
- Le **pourquoi staging** : c'est la « répétition générale » qui réduit fortement le risque de casser la production.

---

## 8. Checklist de validation

Coche chaque case que tu réussis :

- [ ] Je sais **différencier** test unitaire / intégration / E2E et donner un exemple de chacun.
- [ ] Je sais **dessiner** la pyramide des tests et expliquer pourquoi on met peu d'E2E.
- [ ] Je sais **expliquer** les rôles de dev, staging et production.
- [ ] Je sais **définir** build et artifact, et citer un exemple de commande Maven et npm.
- [ ] Je sais **justifier** pourquoi on teste en staging avant la production.
- [ ] J'ai complété **l'exercice** (tests, environnements, build, staging) et vérifié avec `03-correction.md`.

---

🧭 **Pont vers la suite** — Tu sais maintenant *vérifier* (tests) et *où faire tourner* (environnements). Il reste une grande question : **où vit le code, et comment part-il de là pour arriver jusqu'en production ?** La réponse fait intervenir **Git**, l'outil qui stocke et versionne le code. *Note : tu apprendras Git en détail au Bloc 04 — ici, on le voit juste comme le « point de départ » du parcours.* C'est l'objet de la Leçon 4.

---

*Prochaine étape :* Leçon 4 — **Du Git à la production** dans `04-Du-Git-a-la-Production/`.