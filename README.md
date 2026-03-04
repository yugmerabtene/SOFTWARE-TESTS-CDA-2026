## Syllabus — Tests logiciels (Java)

### Chapitre 1 — Introduction aux tests

* Pourquoi tester (qualité, confiance, coûts)
* Vocabulaire essentiel : cas de test, scénario, oracle, données de test, bug vs défaut
* Tests déterministes et tests instables (flaky)
* Organisation d’un projet Java : séparation production et tests

### Chapitre 2 — Panorama des niveaux de test

* Pyramide de tests et logique de progression
* Tests unitaires, intégration, fonctionnels, end-to-end
* Où placer chaque type de test dans un projet

### Chapitre 3 — JUnit 5 : bases indispensables

* Premier test : structure et règles d’écriture
* Annotations : `@Test`, `@BeforeEach`, `@AfterEach`, `@BeforeAll`, `@AfterAll`
* Assertions : `assertEquals`, `assertTrue/False`, `assertThrows`, `assertAll`
* Structure AAA (Arrange / Act / Assert)
* Tests paramétrés : `@ParameterizedTest`, `@ValueSource`, `@CsvSource`

### Chapitre 4 — Tests unitaires : méthode, cas limites, exceptions

* Tester une méthode en isolation
* Cas nominaux et cas limites
* Tests d’exceptions (erreurs attendues)
* Lisibilité et nommage des tests

### Chapitre 5 — Doubles de test : stub, fake, mock, spy

* Pourquoi isoler (DB, API, horloge, réseau)
* Différences entre doubles
* Quand utiliser un fake plutôt qu’un mock
* Risques : tests fragiles, sur-mocking

### Chapitre 6 — Mockito : isoler proprement une dépendance

* Créer un mock, définir un comportement, vérifier un appel
* `when/thenReturn`, `verify`
* Bonnes pratiques : tester le comportement, éviter de tester l’implémentation

### Chapitre 7 — Tests d’intégration : assembler les composants réels

* Objectifs et périmètre (service + repository)
* Jeux de données, préparation et nettoyage
* Tests stables et reproductibles
* Différence intégration en mémoire et intégration avec une base

### Chapitre 8 — Tests fonctionnels : scénarios métier

* Écrire un scénario lisible (donné / quand / alors)
* Scénarios Auth : login OK, login KO, droits, rôle
* Assertions sur le résultat visible (pas sur l’implémentation)

### Chapitre 9 — Tests d’acceptance : critères d’acceptation d’une user story

* Rôle des critères d’acceptation
* Transformer une user story en tests
* Couvrir le strict nécessaire pour accepter une fonctionnalité
* Différence acceptance vs fonctionnel

### Chapitre 10 — Smoke tests : vérifier le vital rapidement

* Définition : “ça démarre et le chemin critique minimal répond”
* Sélection des chemins vitaux (authentification, accès protégé minimal)
* Exécution fréquente en CI, stabilité maximale

### Chapitre 11 — Tests end-to-end (E2E) : parcours complet

* Définition et intérêt
* Points d’entrée : API, UI, CLI
* Stratégie : peu nombreux, ciblés, robustes
* Causes de fragilité et réduction des flakys

### Chapitre 12 — Non-régression : construire un socle durable

* Définir la non-régression (ce que l’on protège)
* Sélection des scénarios critiques
* Politique : aucun merge si non-régression rouge
* Organisation des suites (unit, integration, acceptance, smoke, E2E)

### Chapitre 13 — Tests de sécurité : tester les règles de sécurité

* Contrôle d’accès : rôles et autorisations
* Validation d’entrées : formats, valeurs inattendues, null
* Gestion d’erreurs : éviter les fuites d’informations
* Anti-brute force (principe), limites, verrouillage (logique testable)
* Tests de dépendances (principe) et règles de base

### Chapitre 14 — Couverture de tests : mesurer et piloter

* Couverture lignes, branches, instructions
* Interprétation et limites (100% ne garantit rien)
* Seuils pragmatiques par criticité
* Augmenter la couverture sans casser la lisibilité

### Chapitre 15 — TDD : Red / Green / Refactor

* Cycle et discipline
* Écrire le test avant le code (sur règles simples)
* Refactor sécurisé par les tests
* Quand le TDD est adapté et quand il ne l’est pas

### Chapitre 16 — CI/CD et qualité : exécution automatique

* Exécution à chaque push et pull request
* Ordonnancement des suites (rapides puis lentes)
* Rapports : résultats, couverture, tendances
* Règles de qualité : seuils, blocage, revues

### Chapitre 17 — Projet de synthèse : plan de test complet Auth

* Suite unitaires : règles et cas limites
* Suite intégration : assemblage service + repository
* Suite fonctionnelle : scénarios métier
* Suite acceptance : critères user story
* Suite smoke : chemin vital
* Suite E2E : parcours essentiel
* Socle non-régression final et couverture cible
