## Syllabus — Tests logiciels : unitaires, intégration, non-régression et couverture

### Chapitre 1 — Introduction aux tests

* Pourquoi tester (qualité, coûts, maintenance)
* Vocabulaire : cas de test, scénario, oracle, bug vs défaut
* Pyramide de tests (unitaires, intégration, end-to-end)
* Stratégie de test dans un projet Java (quand écrire quoi)

### Chapitre 2 — Tests unitaires

* Définition et objectifs (tester une méthode en isolation)
* Propriétés d’un bon test unitaire (rapide, déterministe, lisible)
* Données de test : jeux de données simples et cas limites
* Organisation : nommage des tests, structure AAA (Arrange/Act/Assert)
* Assertions : vérifier résultats et états
* Tests d’exceptions (comportements attendus en erreur)

### Chapitre 3 — JUnit 5 (bases)

* Annotations : `@Test`, `@BeforeEach`, `@AfterEach`, `@BeforeAll`, `@AfterAll`
* Assertions essentielles : `assertEquals`, `assertTrue/False`, `assertThrows`, `assertAll`
* Tests paramétrés : `@ParameterizedTest`, `@ValueSource`, `@CsvSource`
* Bonnes pratiques de lisibilité : messages d’assertion, séparation claire des étapes

### Chapitre 4 — Doubles de test et isolation (Mock, Stub, Fake)

* Pourquoi isoler (dépendances externes : DB, API, réseau)
* Types de doubles : stub, mock, fake, spy
* Principes de base avec Mockito (concepts, pas de magie)
* Vérifications : appels attendus, nombre d’appels, arguments
* Éviter les tests fragiles (sur-mocking, tests couplés à l’implémentation)

### Chapitre 5 — Tests d’intégration

* Définition : plusieurs composants réels ensemble
* Différence unitaires vs intégration (scope, durée, fiabilité)
* Tests d’intégration en Java (Spring Boot si projet Spring, sinon approche “module”)
* Gestion des dépendances : base de données, fichiers, configuration
* Environnements : local, CI, isolation des données, nettoyage après test

### Chapitre 6 — Tests de non-régression

* Définition : éviter qu’une fonctionnalité cassée revienne
* Régression fonctionnelle vs technique
* Construction d’un “socle” de tests de non-régression
* Sélection des tests critiques (authentification, droits, sécurité)
* Politique de validation : aucun merge sans suite verte

### Chapitre 7 — Couverture de tests

* Définition et objectifs (mesurer, pas “prouver”)
* Types : line, branch, instruction
* Limites (100% ≠ absence de bugs)
* Seuils pragmatiques (par module, par criticité)
* Couverture et dette de test : priorisation et plan d’amélioration

### Chapitre 8 — Qualité des tests et maintenance

* Tests déterministes : éviter horloge, hasard, réseau
* Anti-patterns : tests qui testent tout, tests sans assertions, tests trop couplés
* Refactor de tests : duplication, fixtures, données lisibles
* Tests et refactor : comment sécuriser une refonte

### Chapitre 9 — Intégration CI/CD (vision globale)

* Exécution automatique : à chaque push / pull request
* Séparer les suites : unitaires rapides, intégration plus lente
* Rapports : résultats, couverture, tendances
* Politique de blocage : seuil couverture, tests obligatoires

### Chapitre 10 — Projet de synthèse (authentification)

* Plan de test d’un module Auth
* Suite unitaires : validation input, règles métier, exceptions
* Suite intégration : repository réel, service, contrôle d’accès
* Suite non-régression : scénarios critiques (login OK/KO, rôle admin, etc.)
* Mesure de couverture et objectifs par composant
