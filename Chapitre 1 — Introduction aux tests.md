

## Chapitre 1 — Introduction aux tests (explication puis code, pour chaque test)

### Objectifs

* Comprendre ce qu’est un test (structure, rôle, qualités)
* Comprendre plusieurs types de tests sur un même mini-sujet
* Pour chaque type : explication du test, puis code, puis explication du code
<img width="1280" height="800" alt="image" src="https://github.com/user-attachments/assets/fb3b0830-e03d-4e0a-af3c-886c50e34a03" />
<img width="1200" height="800" alt="image" src="https://github.com/user-attachments/assets/96ce5c9c-df59-4bd0-8f9e-1be417d5f268" />
---

# 1) Base commune du chapitre (mini-sujet “authentification”)

On va tester une règle simple liée à l’authentification :

* Un secret est **valide** s’il n’est pas `null` et s’il a une longueur minimale.

Cette règle est volontairement simple pour que l’on se concentre sur **les tests**.

---

## Code de production (commun à tous les tests)

### `src/main/java/service/PasswordPolicyService.java`

```java
package service;

public class PasswordPolicyService {

    // Attribut : longueur minimale attendue pour un secret
    private final int minLength;

    public PasswordPolicyService(int minLength) {
        // Le constructeur fixe la règle une fois pour toutes
        this.minLength = minLength;
    }

    public boolean isValid(String password) {
        // Si le secret est absent, on refuse
        if (password == null) {
            return false;
        }

        // Règle : la longueur doit être suffisante
        return password.length() >= minLength;
    }
}
```

Explications du code

* `minLength` est un attribut qui paramètre la règle.
* `isValid` est déterministe : mêmes entrées ⇒ même sortie.
* C’est idéal pour apprendre à tester : pas de base de données, pas de réseau, pas d’horloge.

---

# 2) Test unitaire

## 2.1 Explication du test (ce que c’est, ce qu’on vérifie)

Un **test unitaire** vérifie une petite unité de code (souvent une méthode) **en isolation**.

Ici, on vérifie uniquement la méthode `isValid` :

* secret trop court ⇒ `false`
* secret assez long ⇒ `true`
* secret `null` ⇒ `false`

Qualités attendues

* rapide
* lisible
* indépendant
* facile à diagnostiquer

---

## 2.2 Code du test unitaire

### `src/test/java/service/PasswordPolicyServiceTest.java`

```java
package service;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

public class PasswordPolicyServiceTest {

    @Test
    void shouldRejectTooShortPassword() {
        // Arrange : on crée l'objet à tester avec la règle "minLength = 8"
        PasswordPolicyService policy = new PasswordPolicyService(8);

        // Act : on exécute la méthode testée
        boolean result = policy.isValid("abc");

        // Assert : on vérifie le résultat attendu
        assertFalse(result);
    }

    @Test
    void shouldAcceptLongEnoughPassword() {
        PasswordPolicyService policy = new PasswordPolicyService(8);

        boolean result = policy.isValid("abcdefgh");

        assertTrue(result);
    }

    @Test
    void shouldRejectNullPassword() {
        PasswordPolicyService policy = new PasswordPolicyService(8);

        boolean result = policy.isValid(null);

        assertFalse(result);
    }
}
```

Explications du code (liées au test)

* `@Test` : indique que la méthode est un test JUnit.
* AAA est respecté :

  * Arrange : création de l’objet et données
  * Act : appel de la méthode
  * Assert : vérification avec `assertTrue/assertFalse`
* Chaque test vérifie une seule idée principale, ce qui rend les erreurs faciles à comprendre.

---

# 3) Test d’acceptance

## 3.1 Explication du test

Un **test d’acceptance** vérifie les **critères d’acceptation** d’une user story, sans chercher à tester tout le détail interne.

User story (simplifiée)

* “En tant qu’utilisateur, mon secret doit respecter une longueur minimale pour être accepté.”

Critères

* trop court ⇒ refus
* assez long ⇒ accepté

Ce test ressemble à un mini-scénario qui valide la fonctionnalité “acceptée”.

---

## 3.2 Code du test d’acceptance

### `src/test/java/service/PasswordPolicyAcceptanceTest.java`

```java
package service;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

public class PasswordPolicyAcceptanceTest {

    @Test
    void acceptancePasswordPolicy() {
        // Arrange : la règle métier attendue (minLength = 8)
        PasswordPolicyService policy = new PasswordPolicyService(8);

        // Assert : critères d'acceptation (on va à l'essentiel)
        assertFalse(policy.isValid("abc"));       // trop court => refus
        assertTrue(policy.isValid("abcdefgh"));   // assez long => accepté
    }
}
```

Explications du code (liées au test)

* On reste sur le strict minimum des critères, pas de multiplication de cas.
* L’intention est de valider une user story, pas de couvrir toutes les branches possibles.

---

# 4) Smoke test

## 4.1 Explication du test

Un **smoke test** vérifie très rapidement que le “vital” marche.

Idée

* “Est-ce que la règle fonctionne sur un cas nominal ?”

C’est un test court, stable, exécuté souvent (CI).

---

## 4.2 Code du smoke test

### `src/test/java/service/SmokeTest.java`

```java
package service;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

public class SmokeTest {

    @Test
    void smokePasswordPolicyWorks() {
        // Arrange : setup minimal
        PasswordPolicyService policy = new PasswordPolicyService(8);

        // Assert : vérification minimale sur un cas nominal
        assertTrue(policy.isValid("abcdefgh"));
    }
}
```

Explications du code (liées au test)

* Le smoke test doit rester minimal : un seul chemin critique, un seul résultat attendu.
* Il doit éviter tout ce qui le rendrait instable (temps, réseau, état partagé).

---

# 5) Résumé de fin de chapitre

* Un test = conditions initiales + action + résultat attendu
* Unitaire : petite unité, isolation, très rapide
* Acceptance : critères d’acceptation d’une user story
* Smoke : vital, minimal, très stable
